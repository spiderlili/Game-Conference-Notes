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
- hash (.uasset) + hash (Editor Configuration: ini. Files, Asset version, Code version) = DDC Key
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
- 


https://reattendance.com/event-lobby/5884/schedule
