# Bruno Slot Machine

A conference booth game where the reels are an HTTP response.

Three integers come back from [random.org](https://www.random.org) — real
randomness, harvested from atmospheric noise — and those three integers *are*
the three reel positions. Two 🐶 out of three wins a Bruno t-shirt.

The whole thing is one [Bruno](https://www.usebruno.com) collection: a
**Collection App** for the booth screen, and the requests it drives. No API
key, no signup, no backend, no build step.

```text
SPIN ── bru.ctx.runRequest() ──> Machine/Spin Reels ──> random.org
                                        │
                                  "0\n2\n1"  ──> three reel stops
```

## Quick start

```bash
git clone <this repo> bruno-slot-machine
```

Open the folder in Bruno as a collection, pick the **Booth** environment, and
open the **Bruno Slot Machine** app. Press SPIN — or space, which is what a
wireless presenter clicker sends, so the clicker becomes the lever.

Pre-flight from the terminal before the doors open:

```bash
bru run "Machine" --env Booth
```

## Booth-day checklist

1. `bru run "Machine" --env Booth` — the RNG quota test fails loudly if this
   IP is out of true random bits for the day.
2. Open the app, switch to the **Rehearsal** environment, spin a few times.
   Every spin wins there, and the banner says REHEARSAL, so a practice spin
   can't be mistaken for a real one.
3. Switch back to **Booth**. Press **RESET COUNTER** to zero the tally.
4. Full-screen the app on the booth screen.

## Tuning the machine

**The environment is the control panel.** The app reads it on startup and
again whenever it changes, so switching environments — or editing one while
the app is open — retunes the game live. Everything falls back to a default,
and anything out of range is clamped rather than obeyed, so a typo can't blank
the booth screen.

| Variable | Default | What it does |
|---|---|---|
| `winRate` | `0.5` | Share of spins that win, 0–1. **The odds dial** |
| `reelCount` | `3` | How many reels, 2–6. Also how many integers `Spin Reels` asks for |
| `maxStop` | `63` | Last stop on each reel, so 63 is a 64-stop strip |
| `winMin` | `2` | How many reels must match. Ignored when `prizeTiers` is set |
| `prizeName` | `BRUNO T-SHIRT` | What a winner gets |
| `prizeTiers` | unset | Several prizes: `{"3": "BRUNO HOODIE", "2": "BRUNO T-SHIRT"}` |
| `winSymbol` / `missSymbol` | 🐶 / ⚡ | The two symbols. Emoji, or word marks like `200` and `404` |
| `eventLabel` | *(banner)* | The attract banner |
| `winHint` / `missHint` | *(see Booth)* | The line under a win and under a loss |
| `missText` / `nearMissText` | *(see Booth)* | The verdict on a loss, and when one reel short |
| `winsLabel` | `SHIRTS WON` | Label under the running count of prizes handed out |

`winRate` is a **target**, not a weight: ask for half of all spins and the app
searches the reel strip for the stop count that pays closest to it, then
reports what it *actually* pays in the SHOW API overlay. The machine can never
advertise a rate it doesn't honour, and the rate is nowhere on the
player-facing screen.

Rates quantise with strip length. On the default 64-stop reel the achievable
rates step about a point at a time, so `winRate: 0.1` really pays 10.7%. Raise
`maxStop` to 255 for finer control.

```bash
# try a tuning without editing anything
bru run "Machine/Spin Reels.yml" --env Booth --env-var reelCount=5
```

### Environments as presets

| Environment | The machine it builds |
|---|---|
| **Booth** | The live game: 3 reels, 50/50, one t-shirt |
| **Booth-Tiered** | Same odds, two prizes — three 🐶 wins a hoodie, two wins a shirt |
| **Rehearsal** | `winRate: 1`, banner says REHEARSAL, win line says there's nothing to hand over |

## How the odds work

Real slot machines don't roll a symbol, they roll a **stop** on a physical
reel, and how many stops each symbol occupies is what sets the odds. This
machine works the same way: a strip of `maxStop + 1` stops split between two
symbols, and random.org picks the stop for each reel.

Two symbols split evenly makes every reel a fair coin, and two-or-more heads
out of three lands on 4 of the 8 possible outcomes — which is why the default
is a true 50/50 rather than a tuned approximation.

## What's in here

| Path | What it is |
|---|---|
| `Bruno Slot Machine.yml` | The Collection App — the booth screen, as one HTML file |
| `Machine/Spin Reels.yml` | The spin: one true random integer per reel |
| `Machine/Check RNG Quota.yml` | Pre-flight: true random bits left today |
| `environments/` | Booth, Booth-Tiered, Rehearsal |

## If the wifi dies

The spin fails loudly rather than quietly faking it. The error offers **SPIN
WITH LOCAL RNG**, which falls back to `crypto.getRandomValues` — and the badge
in the footer changes to **LOCAL CRYPTO RNG** for as long as that's what's
drawing the reels, so nobody is told random.org picked their shirt when it
didn't.

random.org meters free use per IP (1,000,000 bits a day, refilled at midnight
UTC). A spin costs a few dozen bits, so a booth won't run it out — but a
conference shares one public IP across a lot of laptops, which is what
`Check RNG Quota` is for.

## Live drawing, by design

There are no claim codes and no ticket numbers. The winner is standing in
front of you, so the screen says they won and someone hands them a shirt. A
losing spin points them at the swag table anyway, so nobody walks off with
nothing.
