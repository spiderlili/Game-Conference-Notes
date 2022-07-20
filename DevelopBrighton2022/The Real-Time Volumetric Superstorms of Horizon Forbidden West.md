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

#### Vertical Profile Gradient in practice

[cont: 0927](https://reattendance.com/event-lobby/5884/session-stage)

# How to support large tornadic storm systems

# Internal lighting

# lightning effects and further visual enhancements 

# Performing within a reasonable budget on PS5 & PS4 hardware

# How to dramatically increase the speed of fast moving temporally constructed volumetric imagery

# Related Talks & Resources
- [SIGGRAPH 2022 - Nubis, Evolved: Real-Time Volumetric Clouds for Skies, Environments, and VFX](https://t.co/a3YHUMPlsw)
- [Nubis: Realtime Volumetric Cloudscapes In A Nutshell](https://www.guerrilla-games.com/read/nubis-realtime-volumetric-cloudscapes-in-a-nutshell)
- [Nubis: authoring real-time volumetric cloudscapes with the decima engine](https://www.guerrilla-games.com/read/nubis-authoring-real-time-volumetric-cloudscapes-with-the-decima-engine)

