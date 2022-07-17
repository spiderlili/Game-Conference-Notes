[Speaker: Mark Lintott, Engineering Director @Epic Games](https://twitter.com/rendertramp?lang=en)

# Overview
- It is widely accepted that the more efficiently a developer can iterate, the more likely the result of their efforts will be much higher in quality.
- The rate of growth for modern games and the more recent world events that have forced many of us to work in a more globally distributed and isolated manner have only made developer iteration harder than ever.
- Long sync times from source control, increasingly long data and code builds, valuable SSD space being eaten up and increasing larger product footprint are just some of the factors affecting our ability to work efficiently in this new world.
- Upcoming features for Unreal Engine 5 that have already impacted developer iteration for both The Matrix Awakens Demo and Fortnite, and for teams and partners at Epic Games.

## Developer Iteration Loop: PIE (Paly in Editor)
Goal: enable developers to iterate on content as quickly as possible
1. Sync Content (Perforce)
2. Load Editor (Unreal)
3. Make changes in Editor
4. Play in Editor, loop back to 3

## What are the bottlenecks to PIE?
Many developers spend a lot of time just syncing & building before Unreal Engine is ready to iterate:
- Sync content: slow
- Read content: fast
- Build Editor Data: slow

## Syncing & Building is getting worse
- Increased project sizes
- Increased team sizes
- Teams distributed across the world
- Many people working from home

## Why not just buy everyone a really fast PC?
One obvious way to improve iteration is to just keep throwing hardware at the problem. BUT:
- Doesn't address syncing
- Not really scalable
- Costly 

## Some UE5 Solutions
- Horde Storage: Don't build what's already been built
- Virtual Assets: Sync only what you need
- Horde Compute: Build what you must efficiently

## First line of defence: DDC (Derived Data Cache)
- Read Content => DDC Key
- hash (.uasset) + hash (Editor Configuration: ini. Files, Asset version, Code version) = DDC Key - "Virtual Assets"
- Before Build Editor Data: Data is NOT in the cache
- Build Editor Data => DDC => Ready to iterate in Unreal Engine

## Cold vs Hot DDC
- Cold Cache path can be much slower than the Hot Cache path
- Typically the first Editor Load can be very slow as you're warming the cache
- Ideally: don't want to build data if someone else has already done it - don't build what's already been built

## DDC Examples
- Local Project DDC: Don't build what you have already built for that project - out of the box. 
  - Worst case scenario: build everything for every project
    - Project 1 <=> Local DDC
    - Project 2 <=> Local DDC
    - Project 3 <=> Local DDC
- Local Shared DDC: Don't build what you have already built on this workstation
  - Better: bare minimum you should have, but still not great. At least you share between your projects! 
    - Project 1, Project 2, Project 3 <=> Local Shared DDC 
- Team Shared DDC: Don't build what someone else has already built in your team
  - Hopefully this is how most teams have set up their DDC  
    - Dev1 (Local Shared DDC), Dev2 (Local Shared DDC), Dev3 (Local Shared DDC) <=> On-Prem Share (Team Shared DDC) 

## Team Shared DDC
- Doesn't work well in the modern world
- Today's teams are widely distributed & many people work from home, this does not scale.
- Team Shared DDC efficiency affected by network conditions
  - Developer bandwidth
  - Distance of developer to network share
  - VPN Bandswidth / latency 

## Horde Storage: Cloud Based Global DDC
- Hosted in the cloud
- Split into multiple regions
- Replicated between regions
- Get your data close to your developers
- Flexible & configurable
- No VPN latency
- Ready to Iterate time > 2x faster

### Horde Storage Key Features
- Compatible with Amazon S3 & Microsoft Azure
- General CAS (Content Addressable Store)
- Active & passive replication between regional instances
- OIDC login & authentication
- Transient & permanent storage
- Garbage Collection

### Horde Storage Case Studies
- Fortnite: Sync Time reduced from 73 min to 29 min (2.5x faster). Virtualized Textures, Static Meshes & Sound Waves are 3.5x smaller.
  - Note: Static Mesh saving will increase with the uptake of Nanite 
- Matrix Awakens
- Lyra
- Epic internal projects - Amazon S3

## Anatomy of a .uasset
- Structured data: always needed
  - Class Data
  - Structure
  - Properties 
- Bulk data: NOT always needed
  - Representation of the raw source data, .e.g. pixel data
Solution: Virtual Asset, store Virtual Asset & Payload separately

## Virtual Assets - Faster Syncing: Sync just what you need
- If you don't need the bulk data: sync less data
- If you do need the bulk data: sync it on demand
- If your bulk data is already in the Cloud (.i.e. using Horde Storage): it's way faster to sync it from the Cloud than from Perforce

### Virtual Assets Features
- Sync faster
- Smaller disk footprint
- Deduplicates in the local shared DDC
- Sync from the Global Cloud DDC rather than Perforce

## Build what you must efficiently with Horde Compute
- UE5 builds much more data in parallel
- Multi-cores to the rescue: heavy parallelism is great, Unreal will take full advantage of all the cores you have
- What if we could have lower spec local machines & be more efficient by leveraging resources in the Cloud?
- Horde Compute is a general purpose reomte build system for UE5
- It can run jobs on local desktops, dedicated physical hardware or in the cloud
- In the future: you will no longer require costly local developer hardware to build efficiently

## DDB (Derived Data Build)
- A new architecture for UE5 that allows data to be built in process or out of process
- A DDB Command is a struct that contains the arguments, a build function key, a set of build input keys & a set of output keys.
- Build function & all inputs are read from DDC
- Outputs are stored in the DDC with the output keys
- Still supports the massively parallel local build model but opens the door to remote building too

## Future Improvements in the works
- Async Editor Load
- Multi-process Cook
- Hybrid Iterative Cook
- Cook on the Fly 2.0
