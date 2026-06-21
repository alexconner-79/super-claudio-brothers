# Super Claudio Brothers — Claude Code Handoff

## What this is

A Claude-themed Mario-style platformer built as a single HTML file. It's a playable metaphor for how Claude actually works: tokens as currency, hallucinations as enemies, context limits as the timer, system prompts as power-ups.

The game is complete and playable. The main outstanding task is replacing the in-memory/artifact leaderboard with a real Supabase-backed global leaderboard.

---

## Project structure

```
super-claudio-brothers/
├── CLAUDE.md          ← you are here
├── README.md          ← player-facing docs
├── public/
│   └── index.html     ← the entire game (self-contained, no dependencies)
└── supabase/
    └── schema.sql     ← run this in Supabase to set up the scores table
```

No build step. No bundler. The game is vanilla JS + Canvas 2D in a single HTML file.

---

## The Supabase leaderboard task

### What to do

1. Create a free Supabase project at supabase.com
2. Run `supabase/schema.sql` in the Supabase SQL editor
3. Copy your project URL and anon key
4. In `public/index.html`, find the `// ── Storage` section at the top of the script
5. Replace the three storage functions (`getLeaderboard`, `saveLeaderboard`, `submitScore`) with Supabase fetch calls (see below)

### Current storage functions to replace

```js
// Currently using artifact window.storage — replace these three:
async function getLeaderboard() { ... }
async function saveLeaderboard(lb) { ... }
async function submitScore(initials, score) { ... }
```

### Supabase replacements

```js
const SUPABASE_URL  = 'https://YOUR_PROJECT.supabase.co';
const SUPABASE_ANON = 'YOUR_ANON_KEY';

async function getLeaderboard() {
  const res = await fetch(
    `${SUPABASE_URL}/rest/v1/scores?order=score.desc&limit=10`,
    { headers: { apikey: SUPABASE_ANON, Authorization: `Bearer ${SUPABASE_ANON}` } }
  );
  return res.ok ? await res.json() : [];
}

async function submitScore(initials, score) {
  // Insert the new score
  await fetch(`${SUPABASE_URL}/rest/v1/scores`, {
    method: 'POST',
    headers: {
      apikey: SUPABASE_ANON,
      Authorization: `Bearer ${SUPABASE_ANON}`,
      'Content-Type': 'application/json',
      Prefer: 'return=minimal',
    },
    body: JSON.stringify({ initials: initials.toUpperCase().slice(0,3), score }),
  });

  // Fetch updated top 10
  const top10 = await getLeaderboard();

  // Calculate rank
  const allRes = await fetch(
    `${SUPABASE_URL}/rest/v1/scores?order=score.desc`,
    { headers: { apikey: SUPABASE_ANON, Authorization: `Bearer ${SUPABASE_ANON}` } }
  );
  const all = allRes.ok ? await allRes.json() : [];
  const rank = all.findIndex(e => e.initials === initials.toUpperCase().slice(0,3) && e.score === score) + 1;

  return { rank: rank || all.length, top10 };
}
```

Note: `saveLeaderboard` is no longer needed once Supabase is wired in — Supabase handles persistence. You can remove it.

---

## Game architecture (key things to know)

### State machine
`state` variable controls what's on screen:
`'title'` → `'playing'` → `'complete'` → `'nameentry'` → `'leaderboard_result'` → `'title'`
Also: `'paused'`, `'gameover'`, `'leaderboard'` (view from title)

### Zones
- `underground = false` → overworld (rows 0-15 of tile map)
- `underground = true` → The Token Mines (rows 17-29)
- `camY` smoothly scrolls between zones during pipe transitions
- `pipeTransition` object controls the scroll animation

### Pipe entry/exit
- Entrance: col 20-21, overworld. Stand next to it, press Down.
- Exit: col 62-63, underground. Stand next to it, press Down. Emerges at col 65 overworld.
- Detection uses x-overlap, not tile check (see `// Check for pipe entry` in update loop)

### Physics constants (tuned, don't change without reason)
```
topSpeed:     3.4 walk / 5.2 run
accelGround:  0.42 walk / 0.55 run
accelAir:     0.28
frictionGnd:  0.82
frictionAir:  0.96
gravity up:   0.45  (floaty arc peak)
gravity down: 0.62  (snappy fall)
jumpForce:    -12.2 / -13.5 (powered)
```
Variable jump height: releasing jump early cuts `vy *= 0.78`.

### The Claude avatar (drawClaudio function)
Pixel-faithful recreation of the Claude favicon:
- Terracotta square body (`#c1714a`)
- Two dark eye squares with glint
- Ear nubs either side
- Two stubby legs that animate with walk speed
- Powered (big) mode: orange glow outline, taller hitbox (h:42 vs h:30)

### Level map
`RAW` array, 30 rows × 150 cols. Characters:
- `G` = ground tile
- `B` = brick (destroyable when powered)
- `Q` = question block (releases token or system prompt power-up)
- `P` = pipe tile (placed in RAW then overwritten by buildLevel for entrance/exit pipes)
- `C` = token spawn (parsed in buildLevel, not rendered as a tile)
- ` ` = air

Gremlins and extra tokens are spawned manually in `buildLevel`, not from RAW.

### Leaderboard data shape
Each entry: `{ initials: string (3 chars), score: number, date: string (YYYY-MM-DD) }`
Top 10 shown. Player rank shown even if outside top 10.

---

## Theme reference (if adding content)

| Mario element   | Claudio equivalent          |
|-----------------|-----------------------------|
| Mario           | Claudio (Claude avatar)     |
| Mushroom        | System Prompt card          |
| Coins           | Tokens (gold hexagons)      |
| Goombas         | Hallucination Gremlins      |
| Pipes           | Prompt Pipes (`>_`)         |
| Underground     | The Token Mines             |
| Timer           | Context window              |
| Game over       | "Context Limit Reached"     |
| Level complete  | "Response Complete!"        |
| Lives           | Attempts Remaining          |

---

## Controls

| Key         | Action       |
|-------------|--------------|
| ← →         | Move         |
| ↑ / Z       | Jump         |
| Shift       | Run          |
| ↓           | Enter pipe   |
| P           | Pause        |
| Enter       | Start/confirm|
| L           | Leaderboard  |

---

## Ideas for next steps (beyond Supabase)

- Score deduplication: only keep a player's best score per initials
- Anti-cheat: validate score server-side with a Supabase Edge Function (check score is plausible given time elapsed)
- World 1-2: extend the underground section, add a second exit pipe further right
- Mobile controls: on-screen d-pad + jump button
- Sound toggle: mute button in HUD
- Sprite sheet: replace canvas-drawn Claudio with a proper spritesheet PNG for smoother animation
