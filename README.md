# Super Claudio Brothers

A Claude-themed Mario-style platformer — built entirely in a single HTML file with vanilla JS and Canvas 2D.

![Title Screen](screenshots/title-screen.png)

## Play

Open `index.html` in any browser. No dependencies, no build step, no server required.

### Controls

| Key | Action |
|-----|--------|
| Arrow keys / WASD | Move |
| Up / Z / Space | Jump |
| Shift | Run |
| Down (on pipe) | Enter/exit underground |
| P | Pause |
| L | Leaderboard |

## Features

- Side-scrolling platformer with overworld and underground zones
- Gremlins, tokens, question blocks, breakable blocks, pipes, and a flagpole
- Power-ups (System Prompt makes you big, star mode gives invincibility)
- Checkpoint system and lives
- Global leaderboard via Supabase
- Retro pixel-art style with procedural level generation
- Sound effects and background music (Web Audio API, no external files)

## Claude Code Skill

There's a Claude Code skill that generates unique themed versions of the game. Each run creates a different colour palette, level layout, enemy style, and Claudio costume.

Install the skill by copying `skills/super-claudio-brothers/` to `~/.claude/skills/`, then run `/super-claudio-brothers` in Claude Code.

### How it works

The game engine lives in `template.html` with a replaceable `THEME` block. The skill generates only the theme (~40 lines of JS) and injects it via `sed` — no need to regenerate the entire game. This makes each themed version generate in seconds, not minutes.

## Tech

- Single HTML file, zero dependencies
- Vanilla JavaScript + Canvas 2D
- Web Audio API for sound (no audio files)
- Supabase for global leaderboard
- Procedural level builder from theme parameters

## Project Structure

```
index.html          # Original playable game
template.html       # Engine template with replaceable THEME block
schema.sql          # Supabase leaderboard schema
skills/             # Claude Code skill for themed generation
screenshots/        # Screenshots for README
CLAUDE.md           # Project context for Claude Code
```
