---
name: buddy-reroll
description: "This skill should be used when the user asks to 'reroll buddy', 'change buddy', '刷宠物', '换宠物', 'reset buddy', 'reroll /buddy', '想要一只新的宠物', or mentions Claude Code buddy/companion customization. Helps users search for a specific buddy combination and apply it to their config."
---

# Claude Code Buddy Reroll

Help users reroll their Claude Code `/buddy` companion to get a specific species, rarity, eye style, hat, and stats.

## How it works

The buddy is determined by `hash(userID + SALT)` where SALT is `friend-2026-401`. The hash function depends on the Claude Code installation method:

| Install method | Runtime | Hash function | Run command |
|---|---|---|---|
| `npm install -g @anthropic-ai/claude-code` | Node.js | **FNV-1a** | `node buddy-reroll.js` |
| Official native installer | Bun | **Bun.hash** | `bun buddy-reroll.js` |

Using the wrong runtime produces completely wrong results.

## Step 1: Determine install method

Check `~/.claude.json` for the `installMethod` field:
- `"global"` → npm → use `node`
- `"native"` → official → use `bun`

## Step 2: Check current buddy

The userID to check depends on config:
- Default: the `userID` field in `~/.claude.json`
- If `oauthAccount.accountUuid` exists, that field takes priority

Run:
```bash
node scripts/buddy-reroll.js --check <userID_or_uuid>
# or
bun scripts/buddy-reroll.js --check <userID_or_uuid>
```

## Step 3: Search for target buddy

Ask the user what species, rarity, eyes, hat they want. Then run:

```bash
node scripts/buddy-reroll.js --species cat --rarity legendary --eye '✦' --hat crown
```

Full option list:
- `--species`: duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk
- `--rarity`: common, uncommon, rare, epic, legendary
- `--eye`: · ✦ × ◉ @ °
- `--hat`: none, crown, tophat, propeller, halo, wizard, beanie, tinyduck
- `--shiny`: require shiny variant (1% chance, takes longer)
- `--min-stats [value]`: require all stats >= value (default 90)
- `--count <n>`: number of results (default 3)

## Step 4: Apply to config

The only file to modify is **`~/.claude.json`**. It has this structure:

```json
{
  "userID": "abc123...",              // npm users: 32-byte hex, buddy seed
  "oauthAccount": {
    "accountUuid": "uuid-here"       // OAuth users: UUID format, takes priority over userID
  },
  "companion": {
    "name": "Gravy",
    "personality": "A common cat of few words.",
    "hatchedAt": 1775015300069
  }
}
```

**Which field to change for the buddy seed:**
- If `oauthAccount.accountUuid` exists → change this field
- Otherwise → change `userID`

**For the companion (name + personality), two options:**

Option A — Let Claude Code regenerate (delete the block):
```json
// Remove the entire "companion" block from ~/.claude.json
// After restart, Claude Code will call an LLM to generate a new name
// and personality based on the buddy's rarity, species, stats, and
// random inspiration words from a 156-word bank.
```

Option B — Set custom name and personality:
```json
"companion": {
  "name": "YourCustomName",
  "personality": "A legendary cat with a crown that judges your code quality.",
  "hatchedAt": 1775015300069
}
```
The `hatchedAt` field is a timestamp in milliseconds. Keep it as-is if updating an existing companion.

**Full workflow:**
1. Read `~/.claude.json`
2. Determine which field holds the buddy seed (`oauthAccount.accountUuid` or `userID`)
3. Replace that field's value with the uid from the search script
4. Ask user: regenerate name/personality, or set custom ones?
5. Update or delete the `companion` block accordingly
6. Tell user to restart Claude Code and run `/buddy`

## Species reference

18 species with unique ASCII art:
- duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk

6 eye styles: · (default) ✦ (sparkly) × (dizzy) ◉ (wide-eyed) @ (digital) ° (surprised)

8 hats: none, crown, tophat, propeller, halo, wizard, beanie, tinyduck (common rarity always gets none)

5 rarities with weights: common 60%, uncommon 25%, rare 10%, epic 4%, legendary 1%

## Important notes

- The buddy-reroll.js script is at `scripts/buddy-reroll.js` relative to this skill
- Stats (DEBUGGING, PATIENCE, CHAOS, WISDOM, SNARK) affect the LLM-generated name and personality
- Shiny has a 1% chance and currently has no visual difference, but flags the companion as extra special in the name generation prompt
- Changing userID/accountUuid does not affect API keys, conversation history, or local config — it only affects buddy seed and telemetry
