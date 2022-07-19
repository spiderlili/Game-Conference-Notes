Speakers: Sam Royall (Electric Square), Neil Hutchison (AlphaBlit), Matt Rubin (AlphaBlit)

# Overview
- Co-developing ['Detonation Racing'](https://apps.apple.com/us/app/detonation-racing/id1477484840) for Apple Arcade
- Using the DOTS architecture in Unity to develop a scalable system that played to the strengths of a wide range of device capabilities
- Vehicle handling implementation using the DOTS physics engine 
- How to leverage variable time-steps to support non-interactive sequences during gameplay
- Overview of an internally developed offline visibility culling system for reducing vertex bandwidth
- Overview of the performance reporting system that was utilised to ensure the release of a high-quality and high-performance title across the board of target devices on the Apple platform

# ECS 
- Entity worlds are load in place
- Archetype: a unique set of component types
- Chunks: entities of the same archetype are stored in chunks
- Chunks are 16 kilobytes in size
- Changing an Entity's Archetype is slow (structural change)
- Editor integration is poor (.e.g. can't click on an objet in scene view)
- Entity Debugger takes some acclimatisation

# Job System
- Race condition detection built-in
- Correct use of memory access semanics leads to better parallelisation
- Scales nicely with higher core counts
- Watch out for thread contention on lower core counts as can starve job manager (.e.g. FMOD camping out on a core)
- Hoop jumping to get debug rendering
- No console logging from Burst Compiled jobs
- No asserts from Burst Compiled jobs

# DOTS Physics
- Stateless
- Deterministic
- Elegant
- Fast
- No sleeping: all dynamic bodies simulate every step of the physics world
- Struggles with gnarly collision meshes
  - Weld verts & keep meshes watertight where possible 
- Can filter collisions to clean up response when things do get janky
- Had to reverse engineer DynamicsWorld code to get vehicle handling to fit cleanly

# When NOT to use DOTS in your production game?
- Don't know what the game is
- Don't have a realistic budget & schedule
- Don't have CPU performance challenges

# DOTS Physics Bullet Time
- Cinematic slow-motion effect during crash sequence
  - Ease-in / ease0out target time scale 
- [StepPhysicsWorld.cs](https://docs.unity3d.com/Packages/com.unity.physics@0.0/api/Unity.Physics.Systems.StepPhysicsWorld.html)
  - Scale delta-time passed to physics step 
- Time.timeScale needed for set-piece animation playback & VFX

## Scalable Performance
- Update / Render / GPU Work
- Multi-core desktop CPU vs dual-core mobile CPU
- Great big balancing act
- [iOS Benchmarks](https://browser.geekbench.com/ios-benchmarks)

## Rendering Quality Levels
- Dial down visuals on low end
  - Single shadow pass 
- Increase graphics fidelity on high-end
  - Additional lights
  - Post-processing
  - HDR 
- Advanced
  - [SRP Batcher: Speed Up Your Rendering](https://blog.unity.com/technology/srp-batcher-speed-up-your-rendering) 
    - Tick-box in URP Settings: Experimental API, DOTS Hybrid Renderer only
    - Persistent cmd-buffer: next-gen rendering engine
    - Instancing support
    - [BatchRenderGroup.AddBatch](https://docs.unity3d.com/ScriptReference/Rendering.BatchRendererGroup.AddBatch.html)
    - [BatchRendererGroup](https://docs.unity3d.com/ScriptReference/Rendering.BatchRendererGroup.html)
- Graphics Jobs are project-side setting: need to conditionally disable Graphics Jobs
  - Inject cmd-line arg to UnityPlayer
  - -force-gfx-mt
 
## Performance tuning
- [Optimizing graphics rendering in Unity games](https://learn.unity.com/tutorial/fixing-performance-problems-2019-3-1#5e85bbb0edbc2a08897d4839)
- Min-spec: iPad Mini 4th gen (Metal support)
- CPU Profiler looked tight but within reason
- Job heavy on dual-core mobile CPUs
  - Thread contention 
- Better rendering perf w/o Graphics Jobs
  - Trace stabilized after disabling 
- Vertex-bound on GPU capture
  - Processing > 3 million verts in certain track positions 

## Benchmarking Testbed
- [Unity's SRP Batcher Benchmark](https://github.com/Unity-Technologies/SRPBatcherBenchmark)
- Rapid iteration
- Bottom up
- Generate grid of primitives
  - GameObject.CreatePrimitive() in edit-mode
  - Specify maximum number verts-processed 
- Keep adding primitives until bottlenecked
  - Back off a bit from threshold 
- Don't want to tax the CPU / GPU on mobile
  - Power drain: avoid thermal throttling 
- Profile along the way to understand costs
  - Testbed captures are fast 
- Empirical observation
- On-screen debug diagnostics

## DOTS Hybrid Renderer
- Integrate Hybrid Renderer into testbed
- Add Subscenes with different use-cases
- Converts MeshRenderers & MeshFilters to DOTS RenderMesh components
- Use BatchRenderGroup
  - Passes OnPerformanceCulling() callback
  - real-world example 
- Performs view-frustum culling & LOD selection on CPU
  - Sets results into BatchCullingContext.VisibleIndices 
- Wants to extend culling to support "remove hidden surfaces" to reduce vert processing

## Offline visibility Capture
- Generate "a priori" visibility map offline using track-spline
- Add unique ID to each MeshRenderer / RendererMesh
  - PreVisibilityIndex / VisibilityId 
- Discretize track spline(s) into sections
- Perform track walk by stepping camera along spline
- Sample number of visible pixels for each draw call
- Record results of visible RenderMesh instances into per-section visibility bitmask
- Logically "Or" bitmasks together for all camera samples in a given section
- Create Visibility_XXX.unity for each track
- References source level track geometry assets: same Subscenes referenced by in-game Scenes
- Used occlusion queries for previous projects: no obvious way to get this approach working in Unity
- Roll your own based on [picking hack from OpenGL days](http://www.opengl-tutorial.org/miscellaneous/clicking-on-objects/picking-with-an-opengl-hack/)
  - Encode id into unlit color value in fragment shader
  - Read pixels from frame buffer, build histogram of visible pixels for each id 
- Swap materials at runtime before capture
  - Replace in-game material with simple unlit shader: URPUnlitShader_LD.shader 

## Offline visibility Capture Tool
- Capture runs in play-mode in Custom Editor Window
- Performs multiple laps in different lanes: left / center / right
- Performs multiple laps on each track: 
  - forward / back / left / right facing angles
  - Records results to different bitmasks
- Writes results to VisibilityDatabase for runtime access 
  - VisibilityDatabase_XXX: ScriptableObject with U64 bitmask encoding
- Supports batch captures of multiple tracks

### Offline Visibility Capture Issues
- Capture resolution & aspect ratio of game window: 1080 at 16:9
- Texture2D Properties: FilterMode.Point, TextureFormat.RGB24, Linear space
- Previous captures can become out-of-date when source MeshRenderers are changed
  - globally disable visibility culling at runtime 
- Shadow pass
- Copute shader for histogram calcs

## Runtime Visibility Culling
- Discard draw calls that don't contribute to the final frame buffer
- Extend Hybrid Renderer culling system to include additional "visibility culling" data
  - Plumbed into HybridChunkCullingData ECS Component
- Perform constant time lookup() of visibility maps
  - Based on player camera position & facing angle
  - Forward / Backward, Left / Right 
- Writes up-to-date visibility info each frame
  - Copies data from visibility bitmask into culling info
  - Runs before Hybrid Renderer System
  - does not require structural changes to ECS
  - scene graph
- Hybrid Renderer skips view-frustum-culling for non-visible instances
- Disable when camera strays too far from track surface 

### Native lightweight perf timers
- PlayerLoop CPU Time
  - Finish UnityPlayerLoop() - start UnityPlayerLoop() 
- Render Time
  - PresentMTL() - StartFrameRenderingMTL() 
- Time-to-Present
  - PresentMTL() - start UnityPlayerLoop() 
- GPU Time
  - GPUEndTime - GPUStartTime
  - does not work with graphics jobs!

### Perf Timer Validation
- Echo device display in Quicktime player
- Capture video alongside XCode debug gauges

### Automated Perf Test Fraework
- Historical results across multiple devices
- Can track performance improvements / regressions overtime

# Automated Performance Monitoring
- Historical results across multiple devices
- Can track performance improvements / regressions over time

## Why automate performance monitoring?
- Testing Time
  - Maps x Devices x Game Modes x ... = Huge quantity of Test Cases
  - A few minutes per map adds up quickly
  - Multiplayer testing requires many team members 
- Cycle Time: faster feedback to the team.
- Problems are generally cheaper to fix earlier
- Confidence: for the team, leadership, clients, partners during development, release, updates.
- Hands-free: automation doesn't keep office hours, distributed teams need coverage
- Find out performance impacts & reduce expensive risks
- Can take the tools with you beyond a single project: familiar concepts & implementations, easier to pitch with proven code & results.
  - Ready to go, even before the game

## How to Automate performance monitoring?
- Build & Test Systems
- Coordinator: TeamCity, Jenkins, CircleCI, etc
- Agents: building & testing
- Devices: mobile, desktop, networked
- Lots of wires
- Good USB hubs
- [Unity Test Runner](https://docs.unity3d.com/2017.4/Documentation/Manual/testing-editortestsrunner.html)
- [Unreal Automation System](https://docs.unrealengine.com/4.27/en-US/TestingAndOptimization/Automation/)
- ios-deploy

# Performance Monitoring in Detonation Racing
- Context is important: data needs context to interpret
- Why requires where & when
- Big events in the game, like strikes / crashes are important

## What else can you automatically test?
- Customisation
- User settings
- Specific Device configurations
- Shader Warmup
- Load Times
- Front End
- New platforms / devices / metrics / reports
