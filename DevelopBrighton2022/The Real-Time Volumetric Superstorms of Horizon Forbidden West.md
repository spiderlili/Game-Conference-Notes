[Speaker: Andrew Schneider, Guerrilla Games](https://twitter.com/vonschneidz)

# Overview
- Real-time Volumetric Cloud systems in Games have seen increasing adoption by game developers in the past few years. 
- Many systems use a coverage/type map based modeling method combined with an optimized ray-march and shading solution similar to or expanded upon what was described in detail in the SIGGRAPH 2015 Publication, The Real-Time Volumetric Cloudscapes of Horizon Zero Dawn. 
- How Guerrilla expanded its Nubis cloud system without using expensive simulations or lighting calculations so that the system could scale between current & previous-gen Playstation consoles, including:
  - Tornadic superstorms
  - Internal lighting 
  - Lightning flashes
  - A solution to render faster moving clouds with temporal upscaling as well as visual enhancements

# Inspirations
- [Luke Howard, "On the Modification of Clouds" (1802)](https://blogs.bl.uk/science/2020/04/clouds-how-luke-howard-linked-weather-lore-and-natural-philosophy.html) classified clouds by their altitude, basic physical characteristics, and the conditions (amount of water vapor, temperature, weather, changes over altitude, wind, turbulence) that give rise to their development in Latin:
  - Cirrus: Heap / Pile 
  - Cumulus: Layer / Sheet 
  - Stratus: Hair / Fibre

# Nubis: Real-time Volumetric Cloud System
- Guerrilla's Nubis Cloud System is named after Luke Howard's "nubification": Water vapors in warm layers of air, rises and enters cold air which cuases it to condense on dust particles, rendering it invisible to us
- Low emission of vapor into cool air produces the wispy stratus clouds that flow over the landscape
- High emission corresponds to round billowing cumulus clouds that grow higher until they flatten out into a colder layer of air
- There are multiple layers of air in the atmosphere with differing characteristics that vapor can travel through
- SIGGRAPH 2015 Advances in Real-Time Rendering Course: 2 probabilities that determines what type & how much of a cloud to draw at a given point:
  - Cloud Coverage: describes the presence of clouds
  - Cloud Type: describes the height of clouds, is it blowy / wispy?

### How to describe the probability & change in density over height for a cloud?
Use a gradient:
- Cloud type is determined by the rate of emission & temperature
- Each cloud type is a different height range => can order a set of gradients according to Cloud type in order to represent the height probabilities for each type of cloud. 
- Type can then serve as the value that blends between cloud type functions (stratus, stratocumulus, cumulus). 
- Combine into 1 look up: Vertical Profile Gradient to accelerate / decelerate some of the transitions between types, add some areas with varied intensity to create more variety.

### Vertical Profile Gradient in practice
- Details on clouds can be categorised into:
  - Billows: forms when density increases in a given area, pushes water vapor upward & outward
  - Wisps: forms when density decreases, vapor dissipates & curls around the weight created by turbulent forces

### How to describe the curling & rolling shapes of clouds over 3D space?
Use 3D noise textures like:
- Perlin noise: wispy
- Inverted Worley noise (1 - Worley): billowy
- Perlin-Worley (2015): combines both Perlin & Worley noise depending on cloud type & height
  - Subtract the web-like shapes of Worley noise from the high-density regions of Perlin noise to produce round shapes 
  - Get the best of both worlds with 1 3D texture sample (expensive)

#### A More Performant Density Model
- base cloud density = low frequency, Perlin-Worley & Worley noises, erode the shape using high frequency Worley noises to add detail without taking anyway from the centre of the clouds. 
- The erosion is done using a remapping function (like Houdini's fit range):
  - base_cloud_density = remap(lf_noise, 1.0 - hf_noise, 1, 0, 1)
- Using 2 noise samples => only do high frequency noise operations where needed to improve performance
- As opposed to multiplying the noises together: remapping prevents a loss of too much density at the core of the cloud shape
- Using noise as a base probability for cloud density => can apply the coverage value [0, 1] as an erosion to the product of height gradient:
  - clouds_density = remap(base_cloud_density * vert_profile_gradient, 1 - cloud_coverage, 1, 0, 1) 
- This allows the clouds to expand / contract over a gradient of coverage values in space: useful when animating clouds coverage in transitions
- Simulate the movement of clouds across the regions of type & coverage probability by adding an offset in wind direction which is incremented in time & subtracted from the sampele position of 3D noises:
  - sample_pos += wind_direction * time_offset

### Solution for Cirrus Clouds
- Create a texture containing 3 tiling cloudscapes that cover cloud types from wispy Cirrostratus to the billowy Cirrocumulus.
- Blend between each depending on the cloud type being sampled.
- The inverse of coverage is used to erode the series of cloud shape.

## Nubis Data Field Generator
- To fill the sky with clouds: need a system to define a field of cloud coverage & type data over the XY domian of the world so it can be sampled in the nubis renderer.
- Nubis Data Field Generator is usig a set of noise remap functions & composites: can produce a variety of different cloudscapes.

#### The Renderer
- Construct renders using raymarching: can be used whnever you need to construct an image that contains volumetric objects.
- Construct density & lighting samples at several points along the view vector & combine them.
- Done for every pixel to produce a full render
- To ray march volumetrics like clouds efficiently: good idea on where to begin & end march

### The Lighting Model
#### Direct Scattering
Optical Depth(d): T = e ** -d
- Augustus Beer (1852)
- J.H Lambert (1760)

### Performance
- Performing within a reasonable budget on PS5 & PS4 hardware
- How to dramatically increase the speed of fast moving temporally constructed volumetric imagery
## Superstorms
- El Reno, OK (2013) visualization, National Center for Supercomputing Applications
- The Mesocyclone
  - A column, flat & spread out on bottom
  - Ragged shapes
  - Vortex shape / motion 
- The Anvil Cloud
  - Above the mesocyclone
  - Propagation into cirrus layer 
[cont: 1036](https://reattendance.com/event-lobby/5884/session-stage)

# lightning effects and further visual enhancements 
- Lightning System: General Position => Trigger Flash Animation
- Nubis Renderer: receivs flash positino from lightning system's general position, genereate stable mask

# Related Talks & Resources
- [SIGGRAPH 2022 - Nubis, Evolved: Real-Time Volumetric Clouds for Skies, Environments, and VFX](https://t.co/a3YHUMPlsw)
- [Nubis: Realtime Volumetric Cloudscapes In A Nutshell](https://www.guerrilla-games.com/read/nubis-realtime-volumetric-cloudscapes-in-a-nutshell)
- [Nubis: authoring real-time volumetric cloudscapes with the decima engine](https://www.guerrilla-games.com/read/nubis-authoring-real-time-volumetric-cloudscapes-with-the-decima-engine)

