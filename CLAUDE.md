# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**无限向下 (Infinitely Down)** - A WeChat Mini Game based on the classic "是男人就下100层" (If You're a Man, Drop 100 Floors). Vertical falling casual game with competitive social features and hybrid monetization.

- **Engine**: Cocos Creator 3.8.5
- **Language**: TypeScript (ES2015)
- **Platform**: WeChat Mini Game (primary), WeChat PC (secondary)
- **Status**: Early MVP phase - core game code not yet implemented

## Development Setup

This is a **Cocos Creator project**, not a standard npm project. The `package.json` contains only project metadata for the engine.

**To develop:**
1. Open project in Cocos Creator 3.8.5
2. Create scenes, prefabs, and scripts via the editor
3. Game scripts go in `assets/` directory as TypeScript files extending `cc.Component`
4. Preview using Cocos Creator's built-in preview mode
5. Build target: WeChat Mini Game (requires WeChat DevTools for testing)

**Build constraint**: First package must be ≤4MB (WeChat Mini Game requirement)

## Project Structure

```
assets/           # Game code, scenes, prefabs, sprites, audio (currently empty)
library/          # Cocos auto-generated asset database - DO NOT edit manually
temp/             # Auto-generated files including TypeScript declarations
settings/v2/      # Cocos project settings (build targets, device configs)
profiles/         # Editor and build profiles
```

## Key Technical Patterns

**Cocos Creator patterns to follow:**
- Component-based architecture using `cc.Component`
- Scene management via Cocos scene system
- Use object pooling for platforms/hazards (performance critical for mini games)
- Access WeChat APIs via `wx.*` global

**State management:**
- Session state: in-memory
- Player progress: `localStorage` + cloud backup
- Leaderboards: server-side (cloud functions)

**Anti-cheat requirement:** Critical values (scores, currency) must be validated server-side

## Game Architecture (from design docs)

**Core gameplay:**
- Player falls through platforms, avoiding hazards
- Controls: left/right movement, tap to jump, long-press for power jump
- Death conditions: pushed off top of screen, fall into void, or hazard damage

**Platform types (MVP):** Normal, Moving, Breakable
**Platform types (later):** Spring, Conveyor, Ice, Spikes, Hidden

**Game modes:**
- Endless mode (core)
- 100-layer challenge (classic tribute)
- Daily trials, Friend relay races, Real-time arena (post-MVP)

**Monetization points:**
- Death screen: rewarded video for revive
- Result screen: rewarded video for double coins
- Shop: IAP skins, battle pass, character gacha

## Development Phases (from ROADMAP.md)

**MVP (0-3 months):** Core falling mechanics, 3 platform types, basic ads, friend leaderboard
**Growth (3-9 months):** More platforms/hazards, relay races, guild system, battle pass, IAP
**Ecosystem (9-18 months):** UGC level editor, video channel integration, IP collaborations
