---
name: super-claudio-brothers
description: Generate a unique themed Super Claudio Brothers platformer. Creates a custom level, colour palette, enemy look, and Claudio costume. Every run produces a different game.
---

# Super Claudio Brothers — Theme Generator

You generate unique themed versions of a platformer by injecting a THEME block into a template.

**IMPORTANT: Do NOT read the template file. You never need to see it. Just write the theme and inject it with sed.**

**IMPORTANT: First, locate the template. Run: `find ~ -name "template.html" -path "*/super-claudio-brothers/*" 2>/dev/null | head -1` and use that path for all commands below. Store it in a variable: `TEMPLATE=$(find ~ -name "template.html" -path "*/super-claudio-brothers/*" 2>/dev/null | head -1)`**

## Step 1: Theme Selection

Present 5 options. Option 1 is ALWAYS "Classic Claudio" — the original default theme. Options 2-5 are randomly picked from this pool (or invent your own):

Prehistoric, Deep Ocean, Outer Space, Candy Kingdom, Haunted Manor, Cyberpunk City, Wild West, Arctic Expedition, Jungle Temple, Steampunk Factory, Pirate Cove, Enchanted Forest, Retro Arcade, Samurai Garden, Volcano Core, Neon Synthwave, Ancient Egypt, Mushroom Forest, Crystal Caverns, Robot Factory

Format:
> Pick a theme for your game (or type your own):
> 1. ⛏️ Classic Claudio — the original theme (no generation, instant launch)
> 2. 🌋 Prehistoric — dodge dino-gremlins across volcanic platforms
> 3. 🍬 Candy Kingdom — bounce through frosting on gumdrop hills
> 4. 🏴‍☠️ Pirate Cove — leap between ship decks and dodge parrot-gremlins
> 5. 🌌 Outer Space — float between asteroid platforms

If the user picks Classic Claudio (option 1), skip generation — just copy the template directly:
```bash
cp "$TEMPLATE" super-claudio-brothers.html && open super-claudio-brothers.html
```

For any other choice, proceed to Step 2.

Wait for the user to choose before generating.

## Step 2: Generate the Theme

Write ONLY this JavaScript block to `/tmp/claudio_theme.js`:

```javascript
const THEME = {
  name: 'Your Theme Name',
  subtitle: 'Short themed tagline',
  zoneLabel: 'THEMED UNDERGROUND NAME',

  palette: {
    sky:'#hex', ground:'#hex', groundTop:'#hex',
    cloud:'#hex', cloudText:'#hex',
    brick:'#hex', brickLine:'#hex',
    pipe:'#hex', pipeTrim:'#hex',
    gremlin:'#hex', gremlinEye:'#hex',
    ugBg:'#hex', ugGlow:'rgba(r,g,b,0.03)',
  },

  // Level layout — procedural params, NOT a raw map
  pitCols: [[col1,col2]],                    // 1-3 pit ranges in overworld ground (cols 30-140)
  platforms9: [[start,end], ...],             // 8-12 elevated platform ranges on row 9
  platforms7: [[start,end], ...],             // 3-5 higher platform ranges on row 7
  qBlockCols: [col, ...],                    // 8-12 question block columns on row 6
  bBlockCols: [col, ...],                    // 5-9 breakable block columns on row 6
  tokenCols8: [col, ...],                    // 20-30 token columns on row 8
  ugPlatforms21: [[start,end], ...],          // 5-7 underground platform ranges on row 21
  ugPlatforms24: [[start,end], ...],          // 5-7 underground platform ranges on row 24
  overworldGremlins: [[col,row], ...],        // 8-10 gremlin positions (row 11 for ground, 8 for platforms)
  undergroundGremlins: [[col,row], ...],      // 5-7 gremlin positions (row 26 for floor, 23 for platforms)
  groundTokenCols: [col, ...],               // 10-15 ground-level token columns (row 11)
  decorPipes: [[col,startRow,height], ...],   // 3-5 decorative pipes

  drawCostume(ctx, s, w, h, big) {
    // 3-5 canvas calls for a small hat/accessory on Claudio's head
    // s = scale unit. Example hat:
    // ctx.fillStyle = '#8B4513';
    // ctx.fillRect(s*1, -s*1.2, s*6, s*0.5);
    // ctx.fillRect(s*2, -s*1.7, s*4, s*0.5);
  },

  drawBgDecor(ctx, x, y) {
    ctx.fillStyle = C.cloud;
    // Draw themed background shape (stars, snowflakes, bubbles, etc.)
    // Use C.cloud and C.cloudText for colours. Keep simple: few fillRect/arc calls.
    // Example star:
    // ctx.beginPath(); ctx.arc(x+20,y+15,8,0,Math.PI*2); ctx.fill();
    // ctx.fillStyle = C.cloudText;
    // ctx.beginPath(); ctx.arc(x+20,y+15,3,0,Math.PI*2); ctx.fill();
  },
};
```

## Step 3: Inject and Launch

Run this single bash command (do NOT read the template):

```bash
sed '/\/\/ THEME START/,/\/\/ THEME END/{
/\/\/ THEME START/{
p
r /tmp/claudio_theme.js
}
/\/\/ THEME END/p
d
}' "$TEMPLATE" > super-claudio-brothers.html && open super-claudio-brothers.html
```

## Layout Rules

- All columns range 0-149 (level is 150 cols wide)
- Avoid cols 20-21 and 62-63 (pipe locations added by engine)
- pitCols: each range is 2-3 cols wide, placed between cols 30-140
- platforms9: groups of 2-6 cols, spread across the level
- qBlockCols/bBlockCols: spread across cols 15-120, not overlapping
- overworldGremlins row is typically 11 (ground) or 8 (on row-9 platforms)
- undergroundGremlins row is typically 26 (floor) or 23 (on row-24 platforms)
- Vary the spacing and density to match theme feel

## Creative Guidelines

- **Palette**: 13 colours evoking the theme. Sky sets the mood. Underground darker.
- **Costume**: Small head accessory — hat, horns, visor, crown. 3-5 canvas calls max.
- **Background decor**: Themed shapes replacing clouds. Simple pixel-art (fillRect, arc).
- **Level feel**: Theme-inspired spacing. Candy = bouncy short platforms. Space = longer floaty ones.
- **Keep it compact**: The entire theme block should be ~40-50 lines of JS. No more.
