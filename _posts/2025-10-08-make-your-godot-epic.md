---
title: "GodotEOS: Make Your Godot Epic"
date: 2025-10-08T10:00:00+02:00
authors:
  - Jan Mesarč
  - Jakub Hubáček
categories:
  - godot
  - tools
tags:
  - godot
  - gamedev
  - eos
  - epic
  - plugin
toc: true
---

What happens when you want to make a Godot more Epic? You get GodotEOS, a plugin that brings Epic Online Services (EOS) to Godot Engine. This post dives into why we built it, how we did it, and what’s next.

## What is GodotEOS?
GodotEOS is a Godot plugin that wraps Epic Online Services (EOS) SDK into a Godot‑friendly API. It’s open source, modular, and we hope that it’s designed to fit Godot’s style. It supports Godot 4.3+ and is available on GitHub: [GodotEOS GitHub](https://github.com/Flying-Rat/GodotEOS)

What can you do with it? Here are some highlights:
- Sign users in with Epic services (and manage their tokens)
- Get basic user info and friends
- Use achievements, stats, and leaderboards
- Add more online features as you go, subsystem by subsystem

How it fits into Godot?
- Made as a GDExtension, so it feels native in Godot 4
- Uses signals for async work, so you can await results
- Clear init flow with helpful errors and logs
- Modular design: turn subsystems on/off per project

What it is not
- It is not a full game server or a full networking stack
- It does not force you to use all of EOS. Use only what you need.


## Why we built it

After a not so much good start with `GodotItch` plugin, we wanted to make something good and more useful. We checked what Godot is missing. There is a great plugin for Steam, but for GOG or Epic? Nothing. We wanted to support more stores and platforms, and Epic Online Services looked like a great fit.
Here are some reasons why:

- EOS is free and has a rich set of features.
- EOS works everywhere: consoles, PC, mobile.
- EOS is store-agnostic: you can use it on Steam, Epic Store, GOG, Itch, or self-distributed games.
- EOS is not just multiplayer: it has user accounts, achievements, stats, leaderboards, lobbies, matchmaking, and more.
- EOS is battle-tested: it’s used by many big games and studios.

What we wanted to achieve:

- Godot is fast and friendly. Online services are not.
- We wanted a simple, Godot‑style API with signals and small steps.
- You pick only the parts you need.


## The dev journey

This is how we built it, step by step:

- Talking to EOS: We wrote a native extension that calls the EOS C SDK and maps it to Godot classes and signals.
- Startup is strict: EOS needs IDs and keys (Product, Sandbox, Deployment, client). We made a clear init flow with helpful errors.
- Everything is async: EOS uses callbacks. We expose signals and safe helpers so you can await results without pain.
- Lifetime and threads: We guard EOS handles, do the needed “tick”/pump, and avoid race issues.
- Clean errors and logs: When something fails, you see why and where.
- Subsystems from day one: You can turn features on and off to keep builds small and simple.

## Subsystems (modular by design)

![Subsystems](https://media3.giphy.com/media/v1.Y2lkPTc5MGI3NjExdW9zd2tiZTc4Y2VnbnR4aHgzdzNienJ4em50cmN5MWNkMG1kMml5cyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/df6N7bCsVBu3ifLcex/giphy.gif)

## What works today

Integration process:
| Subsystem | Status | Description |
|-----------|--------|-------------|
| [Platform](Platform) | ✅ Core | Foundation for all EOS operations |
| [Authentication](Authentication) | ✅ Ready | Login/logout and user management |
| [User Info](UserInfo) | ✅ Ready | User profile data caching |
| [Friends](Friends) | ✅ Ready | Social friend management |
| [Achievements](Achievements) | ✅ Ready | Achievement and stats system |
| [Leaderboards](Leaderboards) | ✅ Ready | Competitive ranking system |


## What we plan next

What next? Here’s what we have in mind:
| Subsystem | Status | Description |
|-----------|--------|-------------|
| Presence | 🔄 Planned | User online status |
| Reports | 🔄 Planned | Player reporting functionality |
| Metrics | 🔄 Planned | Telemetry and analytics |
| TitleStorage | 🔄 Planned | Cloud-based title storage |
| UI | 🔄 Planned | Epic Games overlay UI |
| AntiCheat | 📅 Future | Anti-cheat systems |
| Sessions | 📅 Future | Multiplayer session management |
| Lobby | 📅 Future | Lobby matchmaking |
| Invites | 📅 Future | Custom invitation system |
| Mods | 📅 Future | Modding support |

## Ideas for the future

## Summary
