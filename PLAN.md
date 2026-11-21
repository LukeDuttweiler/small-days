# Small Days: Game Plan

An original old-school life-sim RPG. You live day to day: manage your needs, grow your skills, earn money and chase a goal. It's built in phases, and **each phase ends playable**.

- **File:** `index.html` (all code inline: vanilla JS + CSS, no dependencies). Open it with a double-click.
- **Look:** retro terminal. Green-on-black monospace, ASCII bars, a few emoji.

---

## Phase overview

| Phase | Focus | Shippable result |
|---|---|---|
| 1 | Core loop: needs, money, time, sleep | Survive the days and earn money |
| 2 | Skills & career ladder | Stats grow and you climb a career |
| 3 | Random life events with choices | Every day plays out differently |
| 4 | Shop, relationships, win condition, polish | A full game with an ending |

---

## Phase 1: Core Loop (detailed)

### Goal
The smallest fun loop: **choose an action → time passes → needs change → money changes → repeat until bedtime → new day.** The tension comes from balancing work (money) against keeping your needs up.

### Screen layout
```
┌──────────────────────────────────────────┐
│  SMALL DAYS            Day 3 · 14:00  😐 │
├──────────────────────────────────────────┤
│  Energy  [██████████░░░░░░░░░░]  52      │
│  Hunger  [██████░░░░░░░░░░░░░░]  31  ⚠   │
│  Fun     [████████████████░░░░]  80      │
│  Money   $142                            │
├──────────────────────────────────────────┤
│  > 13:00 You worked a shift. +$40        │
│  > 11:00 You ate a sandwich. Yum.        │
│  > 08:00 Day 3 begins. Birds chirp.      │
│  (newest at top, last ~15 lines)         │
├──────────────────────────────────────────┤
│ [1] Work 4h  [2] Eat 1h  [3] Play 2h     │
│ [4] Nap 2h   [5] Sleep                   │
└──────────────────────────────────────────┘
```
- Max width ~640px, centered; buttons wrap on narrow screens.
- A bar turns **yellow below 30** and **red below 15**, with a ⚠ marker.
- Mood emoji in the header comes from the average of the needs: 😄 ≥70, 🙂 ≥50, 😐 ≥30, 😟 <30.

### State
```js
const state = {
  day: 1,
  hour: 8,            // 0–24; the day starts at 8:00
  money: 50,
  needs: { energy: 80, hunger: 70, fun: 60 },  // clamped 0–100
  log: [],            // strings, newest first
  over: false         // reserved for later phases
};
```

### Needs decay (per in-game hour passed)
| Need | Decay / hour |
|---|---|
| Energy | −4 |
| Hunger | −5 |
| Fun | −3 |

Decay is applied **after** an action's effects, one hour at a time, so long actions cost more. Sleep is the exception (see below).

### Actions (Phase 1)
| Action | Hours | Cost | Effects (before decay) | Available when |
|---|---|---|---|---|
| 💼 Work | 4 | — | +$40, Fun −10 | hour 8–17 (must finish by 21) |
| 🍔 Eat | 1 | $10 | Hunger +40 | money ≥ $10 |
| 🎮 Play | 2 | — | Fun +35 | always |
| 😴 Nap | 2 | — | Energy +20 | always |
| 🛏️ Sleep | until 8:00 next day | — | Energy → 100, Hunger −15 | hour ≥ 20, **or** Energy < 25 |

- Unavailable actions are shown as disabled buttons with a short reason on hover ("Office is closed", "Not enough money", "Not tired yet").
- An action can't run past midnight, except Sleep.

### Time rules
- Each action advances `hour` by its duration.
- If `hour` reaches **24** without sleeping → **forced sleep**: log "You collapse into bed, exhausted." Next day starts with Energy only 60, and Fun −10.
- Sleep: `day++`, `hour = 8`, plus the effects above. Log "☀️ Day N begins."

### Pass-out rule (failure state for Phase 1)
After each action, if **any need hits 0**:
- Energy 0 → "You pass out from exhaustion." You wake at 8:00 the next day and lose $20 (medical bill).
- Hunger 0 → "You faint from hunger." Same outcome; the bill sets Hunger to 30.
- Fun 0 → "You have a breakdown and call in sick." Lose the rest of the day (sleep), Fun set to 40.
- Money can go negative (debt shown in red). No game over in Phase 1; game over and a goal come in Phase 4.

### Code structure (single `<script>`)
```
state                 // as above
ACTIONS = [ {id, label, emoji, hours, cost, effect(s), available(s) -> true | "reason"} ]
log(msg)              // unshift "HH:00 msg", trim to 15
clamp(n)              // 0–100
advanceTime(hours)    // loop hours: decay needs
doAction(action)      // check availability → pay cost → effect → advanceTime → checkPassOut → checkMidnight → render
sleep(forced)         // new-day logic
checkPassOut()
render()              // header, bars, money, log, buttons
bar(value)            // returns 20-char ASCII bar
keyboard: keys 1–5 trigger the matching button
```
Phases 2–4 add entries to `ACTIONS` and new fields to `state`, so this structure doesn't need rewriting.

### Balance sanity check (one "good" day)
8:00 Work (→12:00) → Eat (→13:00) → Work (→17:00) → Eat (→18:00) → Play (→20:00) → Sleep.
Net: +$80 − $20 = **+$60/day**. Energy ends around 80 − 12×4 = 32 before sleep, so it's fine. Hunger stays above 0 if you eat twice. Fun drops from the work shifts, and the evening Play recovers it.
A greedy day (3 work shifts, no play) should push Fun or Hunger into the danger zone, which is the core tension. We'll tweak the numbers after the first playtest.

### Phase 1 done when
- [ ] Opening `index.html` shows the layout above, with no console errors
- [ ] All 5 actions work and respect their availability rules
- [ ] Needs decay, bars change color, mood emoji updates
- [ ] Sleep, forced midnight sleep and pass-outs all advance the day correctly
- [ ] Number keys 1–5 work
- [ ] Playing 3+ days feels like a balancing act

---

## Phase 2: Skills, Career & Flavor (detailed)

### Context
Phase 1 (`/Users/LukeDuttweiler/Desktop/VibeGame/index.html`) works, but it's "barely a game": the same 5 buttons, the same log lines, and nothing to aim for. Phase 2 adds **skills, a career and a lot of flavor**, so each day reads like a small story and every choice moves you toward something. When it's done, also copy this section into `/Users/LukeDuttweiler/Desktop/VibeGame/PLAN.md` (replacing the short Phase 2 stub) so the folder keeps the up-to-date reference.

Guiding idea: **every click should produce a line of text worth reading, and most clicks should nudge a number you care about.**

---

### 1. Character creation (new start screen)
Shown before Day 1, as a simple form in the same terminal style:
- **Name** (text input; default "Sam").
- **Pick 1 trait** (each changes the game a little and adds unique log lines):
  | Trait | Bonus | Drawback |
  |---|---|---|
  | 🏃 Athletic | Fitness XP ×1.5 | Hunger decays +1/h |
  | 📚 Bookworm | Smarts XP ×1.5 | Fun from Play −10 |
  | 🍳 Foodie | Cooking XP ×1.5, Cook gives +10 hunger | Eat costs $15 |
  | 💬 Charmer | Charm XP ×1.5 | Work Fun penalty −5 more |
  | 🦉 Night Owl | Can work until 23:00; no forced-sleep penalty | Morning actions before 10:00 cost +5 energy |
- The log opens with a personalized intro: *"Sam moves into a tiny apartment on Maple Street with $50 and a suspicious houseplant."*

### 2. Skills
- `state.skills = { fitness, cooking, charm, smarts, creativity }`, stored as XP. **Level = floor(XP / 100), capped at 10.** Shown as `★★★☆☆☆☆☆☆☆` with a small XP progress bar.
- A **level-up** logs a celebratory line plus a skill-specific quip from a list, for example Cooking 3: *"You can now make eggs that aren't also charcoal."*
- Skills feed back into the game:
  - **Fitness:** max Energy +5 per level; Work energy cost −1 per 2 levels.
  - **Cooking:** Cook gives +5 hunger per level, with a chance at a "✨ masterpiece meal" that also gives Fun.
  - **Charm:** a better chance of good Chat outcomes; needed for promotions on the Charm-heavy tracks.
  - **Smarts:** needed for promotions; Study XP ×(1 + 0.1·level).
  - **Creativity:** Paint/Write become profitable side gigs (sell work for money that grows with level).

### 3. New actions (Phase 2 total: ~12, grouped)
Buttons are grouped under small headings: **🏠 Home**, **🏙️ Out & About**, **💼 Work**, **🛏️ Rest**. The number keys still map in display order, and we add **Q/W/E…** keys if there are more than 9.

| Group | Action | Hours | Cost | Effects | Skill XP |
|---|---|---|---|---|---|
| Home | 🍳 Cook | 1 | $4 | Hunger +30 (+5/lvl) | Cooking +25 |
| Home | 📖 Study | 2 | — | Fun −10, Energy −10 | Smarts +30 |
| Home | 🎨 Create (paint/write/music, random) | 2 | — | Fun +15; at Creativity 3+, sell for $5×lvl | Creativity +30 |
| Home | 🎮 Play | 2 | — | Fun +35 | — |
| Out | 🍔 Eat out | 1 | $10 | Hunger +40, Fun +5 | — |
| Out | 🏋️ Gym | 2 | $5 | Energy −20, Fun +5 | Fitness +35 |
| Out | ☕ Café: chat with a neighbor | 1 | $3 | Fun +20 (outcome depends on Charm) | Charm +30 |
| Out | 🌳 Walk in the park | 1 | — | Fun +10, Energy +5, random small find | Fitness +10 |
| Work | 💼 Work shift | 4 | — | Pay by career rank; Fun −10 | job-relevant skill +15 |
| Work | 🗂️ Work overtime | +2 after a shift | — | 1.5× hourly pay; Fun −15 | — |
| Rest | 😴 Nap | 2 | — | Energy +20 | — |
| Rest | 🛏️ Sleep | — | — | as in Phase 1 | — |

Opening hours add realism: Gym 6–22, Café 7–19, Park only while it's light out (7–20).

### 4. Career system
- On Day 1, after character creation, **pick a career track** (you can change it later at a "job board" action, at the cost of dropping back to rank 1):
  | Track | Key skills | Ranks (pay / shift) |
  |---|---|---|
  | 🏢 Office | Smarts, Charm | Intern $30 → Assistant $45 → Analyst $65 → Manager $90 → VP $130 → CEO $200 |
  | 🍽️ Culinary | Cooking, Charm | Dishwasher $28 → Line Cook $42 → Sous Chef $60 → Head Chef $90 → Restaurateur $135 → Celebrity Chef $200 |
  | 🎨 Creative | Creativity, Smarts | Barista-with-a-dream $25 → Freelancer $40 → Designer $60 → Art Director $90 → Studio Head $130 → Icon $200 |
  | 💪 Athletics | Fitness, Charm | Gym Cleaner $28 → Trainer $42 → Coach $62 → Pro Athlete $95 → Team Captain $140 → Legend $200 |
- **Promotion rule:** after each shift, you're promoted if both key skills are at least `rank` (rank 1 needs level 1 in each), **plus** you've worked `rank×2` shifts at your current rank **plus** your performance meter is at least 60.
- **Performance meter (0–100):** a shift worked with all needs ≥ 50 gives +8; with any need < 30 gives −5 and a grumpy boss line; skipping a whole workday gives −10. This makes taking care of yourself matter for your career.
- **Work flavor:** each shift logs a random line from a list specific to your track and rank, e.g. Office/Intern: *"You fixed the printer by glaring at it."* Chef/Head Chef: *"A food critic sent their compliments. Your sous chef cried."* About 60 lines in total (≈2–3 per rank per track).
- A promotion is a big moment: a boxed log line (`★ PROMOTED: Analyst ★`), new pay and a quip from your boss.

### 5. Flavor systems (the "lots of flavor" part)
1. **Varied action text:** each action has 4–6 random log lines, and some use your name, trait or skill level. Eat out cycles through restaurant names ("Taco Tuesday's", "Noodle Hut", "The Greasy Fork"). Play picks a random game or show.
2. **The neighbor:** a small cast of 4 named neighbors with personalities (Rosa the retired opera singer, Dev the anxious coder, Mr. Pickles the cat, Jun the conspiracy-theorist barista). Café chat picks one and logs a mini-exchange, with Charm deciding a good, awkward or great result. Each neighbor gets a 0–100 friendship value now; Phase 4 builds on it.
3. **Morning briefing:** at the start of each day, log the weather (☀️/🌧️/❄️/🌫️) and a silly town headline from about 25 options ("Local Goose Elected to City Council", "Maple St. Pothole Declared Historic Landmark"). Weather changes things slightly: rain gives −5 Fun from the Park and +5 Fun from Play.
4. **Inner monologue:** when a need drops below 30, log a one-time thought line ("Your stomach makes a noise like a dial-up modem."). It resets after the need recovers.
5. **Day summary card:** on Sleep, show a short recap (money earned or spent, skills gained, performance change) and a "Day grade" from F to A+ based on mood and productivity.
6. **Time-of-day color:** the header shows 🌅 morning, 🌞 afternoon, 🌆 evening or 🌙 night, and the panel tint shifts slightly (CSS variable change).

### 6. UI changes
- The stats panel splits into two columns: **Needs + Money** on the left, **Skills (stars)** on the right. On narrow screens they stack.
- A new line under the header: `Sam · 🦉 Night Owl · 🏢 Analyst (Perf 72)`.
- Action buttons are grouped (see §3), and buttons show the cost in their label (`☕ Café 1h $3`).
- The log keeps about 30 lines, with special styling for promotions and level-ups (bright, boxed) and for headlines (italic).

### 7. Code changes (all in `index.html`)
Extend the Phase 1 structure; don't rewrite it:
- `state`: add `name, trait, career:{track, rank, shiftsAtRank, perf}, skills:{…xp}, neighbors:{…}, weather, today:{moneyStart, xpGained}, thoughtsShown:{}`.
- `ACTIONS` entries gain `group`, `xp:{skill:amount}`, `lines:[…]` or `flavor(s)`, and `open:[from,to]`. Reuse `canDo()` and add the opening-hours and trait checks there.
- New helpers: `pick(arr)`, `gainXP(skill, amt)` (applies the trait multiplier and checks level-up), `level(skill)`, `checkPromotion()`, `morningBriefing()`, `daySummary()`, `checkThoughts()`.
- Data tables at the top of the script: `TRAITS`, `CAREERS`, `WORK_LINES`, `HEADLINES`, `NEIGHBORS`, `LEVELUP_QUIPS`, `THOUGHTS`. Keeping the flavor in data means it's easy to add more lines later.
- `newDay()` calls `daySummary()` (before the reset) and `morningBriefing()` (after it).
- Start screen: a `#setup` div that replaces `#game` until the player confirms.

### 8. Build order (each step stays playable)
1. Data tables plus skills/XP/level-up and the skills panel.
2. New actions with groups, opening hours and varied lines.
3. Character creation (name + trait) and career pick.
4. Career pay, performance, promotions and work flavor lines.
5. Morning briefing, weather, inner thoughts, day summary, time-of-day tint.
6. Copy this plan into `PLAN.md`.

### Verification
- `open /Users/LukeDuttweiler/Desktop/VibeGame/index.html` and check: the setup screen appears; the trait changes numbers (e.g. Foodie's Eat costs $15); skills level up with a quip; opening hours disable buttons correctly; working well-rested raises performance and leads to a promotion within about 5–7 days of focused play; the morning headline and weather appear each day; the day summary shows on sleep; there are no console errors and keyboard shortcuts work.
- Balance target: a focused player reaches rank 3 by about Day 7 and rank 6 by about Day 20 or later.

---

## Phase 3: Random Life Events
- About a 25% chance after each action. An event shows text plus 1–2 choice buttons.
- Examples: found $20; a stranger needs help (Charm check); pop quiz at work (Smarts check → raise or scolding); rainy day (Play gives less Fun); sprained ankle (no Gym for 2 days).

## Phase 4: Goals & Polish
- Shop: Comfy Bed (better sleep), Computer (Play/Study boost), Fridge (cheaper food).
- Neighbor friendship meter with friend and best-friend milestones.
- Win: reach CEO **or** save $5,000 to buy a house. Ending screen shows days taken and a stats summary.
- localStorage autosave and a "New Game" button.
