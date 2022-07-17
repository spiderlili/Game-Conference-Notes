Speakers: Sam Royall (Electric Square), Neil Hutchison (AlphaBlit), Matt Rubin (AlphaBlit)

# Overview
- Co-developing 'Detonation Racing' for Apple Arcade
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
- cont: 1500

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

https://reattendance.com/event-lobby/5884/session-stage
