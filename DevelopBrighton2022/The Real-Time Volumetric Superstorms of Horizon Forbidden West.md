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
- Nubis Data Field (NDF) Generator uses a set of noise remap functions & composites to produce a variety of different cloudscapes.

### The Renderer
- Construct renders using raymarching: can be used whnever you need to construct an image that contains volumetric objects.
- Construct density & lighting samples at several points along the view vector & combine them.
- Done for every pixel to produce a full render
- Need to know where to begin & end march to raymarch volumetrics like clouds efficiently
- Since the atmosphere of the earth is a spherical shell: can define raymarch ray segment using intersections on the inside & outside of the shell.
  - Cirrus cloud layer onlybneed 1 intersection as it's a 2D layer
- Know where you don't need to construct any samples: skip raymarching these pixels altogether for optimisation
  - If objects like trees & mountains render in front of clouds: don't march from those pixels => reduce the total amount of work that needs to be done.
- For pixels that have visible sky in them: march over the view ray to gather the density & lighting data needed to construct pixel color & opacity
  - Start with large steps
  - Construct a "coarse sample" at each step: ubilt with only the cloud coverage & vertical profile gradient to keep the operation cheap
  - Check these coarse samples until one of them produces a non-zero result: then switch to taking smaller steps to construct "fine samples" which include 3D noise texture amples needed to erode the base cloud shape
  - Repeat until the total density sampled along the view ray sums full opacity => stop the march

#### How to calculate light intensity info for cloud color?
- For every fine sample, march towards the light along the light ray, taking fine samples to gather the density amount between the sample & light source for use in light intensity calculations.
- Very computationally expensive to model what happens when a photo enters a cloud: focus on the characteristics of cloud lighting that's the most important to your usecase, model less expensive approximation methods for them.
- Light energy = direct scattering + ambient scattering
  - Direct scattering: light energy associated with the strongest light source (sun)
  - Ambient scattering: light energy associated with the contribution from other bright sources (sky, neighoburing clouds)

#### Direct Scattering
- Define direct scattering at a given sample point in a cloud as a function of 3 probabilities:
  - Direct scattering = Transmittance * Scattering Phase * In-Scatter Probability
- Absorptive properties [(Beer's Law)](https://en.wikipedia.org/wiki/Beer–Lambert_law) is a reasonable approximation for traslucent clouds:
  - Sum up the density samples along the light rays, plug it into the equation as the optical depth  
- Augustus Beer (1852): quick way to masure how much light was being absorbed by opaque molecules in a solution
- J.H Lambert (1760): express reduction of light transmittance as a function of optical depth (d) in a given medium
  - T = e ** -d 
- The deeper a photon is in a translucent object: the lower the probability that you'll escape & reach someone's eyes 
- HenyeyGreenstein function to approximate the intensity of scattered light that the viewer sees

### Performance
- Performing within a reasonable budget on PS5 & PS4 hardware
- How to dramatically increase the speed of fast moving temporally constructed volumetric imagery

## Superstorms
- El Reno, OK (2013) visualization, National Center for Supercomputing Applications: reacreating a superstorm using a supercomputer
  - [The Man Who Caught the Storm, Brantley Hargrove](https://www.amazon.co.uk/Man-Who-Caught-Storm-Legendary/dp/1476796092)
- The Mesocyclone
  - A column, flat & spread out on bottom
  - Ragged shapes
  - Vortex shape / motion 
- The Anvil Cloud
  - Above the mesocyclone
  - Propagation into cirrus layer 

# lightning effects and further visual enhancements 
- Lightning System: General Position => Trigger Flash Animation
- Nubis Renderer: receivs flash positino from lightning system's general position, genereate stable mask

# Related Talks & Resources
- [SIGGRAPH 2022 - Nubis, Evolved: Real-Time Volumetric Clouds for Skies, Environments, and VFX](https://t.co/a3YHUMPlsw)
- [Nubis: Realtime Volumetric Cloudscapes In A Nutshell](https://www.guerrilla-games.com/read/nubis-realtime-volumetric-cloudscapes-in-a-nutshell)
- [Nubis: authoring real-time volumetric cloudscapes with the decima engine](https://www.guerrilla-games.com/read/nubis-authoring-real-time-volumetric-cloudscapes-with-the-decima-engine)

