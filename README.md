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

<!-- screenshot: ![The booth screen, mid-spin](images/01-booth-screen.png) -->

## What you need

| | |
|---|---|
| **Bruno v4.0.0 or newer** | The booth screen is a [Bruno App](https://docs.usebruno.com/apps/overview), and Apps landed in v4. On an older Bruno the requests still work, but the app tab won't be there |
| **`bru` v4.0.0 or newer** *(optional)* | Only for the terminal pre-flight. `npm install -g @usebruno/cli` |
| **Internet** | random.org draws the reels. No API key, no signup, no account |

Nothing to install, nothing to build, no backend to stand up.

## Quick start

```bash
git clone <this repo> bruno-slot-machine
```

1. In Bruno: **Collection → Open Collection**, and pick the folder you just
   cloned.
2. Select the **Booth** environment (top right).
3. Click **Bruno Slot Machine** in the sidebar — that's the app.
4. Press **SPIN**. Or press space: that's what a wireless presenter clicker
   sends, so the clicker becomes the lever.

<!-- screenshot: ![Bruno's Open Collection dialog](images/02-open-collection.png) -->
<!-- screenshot: ![The collection in Bruno's sidebar](images/03-app-in-sidebar.png) -->

Pre-flight from the terminal before the doors open:

```bash
bru run "Machine" --env Booth     # 3 requests, 12 tests
```

## Demo it in a minute

If you're showing Bruno rather than running a booth, this is the tour:

| Do this | Say this |
|---|---|
| Press **SPIN** | "The reels keep turning until the random numbers land — that's a live HTTP call." |
| Press **SHOW API** | "That's the request that just ran, and every integer mapped to its reel." |
| Open `Machine/Spin Reels` | "The whole game is a GET request in a YAML file. The app is a file in the same collection." |
| Switch to **Booth-Tiered** | "The machine just reconfigured itself — two prizes now, and no code changed." |
| Open `Machine/Log Prize Handout` after a win | "Every prize that goes out posts a JSON receipt, and the tests check the receipt against what we sent." |
| Run `bru run "Machine" --env Booth` | "Same collection, no app, straight from CI." |

<!-- screenshot: ![The SHOW API overlay](images/04-show-api.png) -->

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

<!-- screenshot: ![The environment selector](images/05-environment-selector.png) -->

| Variable | Default | What it does |
|---|---|---|
| `winRate` | `0.5` | Share of spins that win, 0–1. **The odds dial** |
| `reelCount` | `3` | How many reels, 2–6. Also how many integers `Spin Reels` asks for |
| `maxStop` | `63` | Last stop on each reel, so 63 is a 64-stop strip |
| `winMin` | `2` | How many reels must match. Ignored when `prizeTiers` is set |
| `prizeName` | `BRUNO T-SHIRT` | What a winner gets |
| `prizeTiers` | unset | Several prizes: `{"3": "BRUNO T-SHIRT", "2": "BRUNO STICKER PACK"}` |
| `winSymbol` / `missSymbol` | 🐶 / ⚡ | The two symbols. Emoji, or word marks like `200` and `404` |
| `eventLabel` | *(banner)* | The attract banner |
| `winHint` / `missHint` | *(see Booth)* | The line under a win and under a loss |
| `missText` / `nearMissText` | *(see Booth)* | The verdict on a loss, and when one reel short |
| `winsLabel` | `SHIRTS WON` | Label under the running count of prizes handed out |
| `logUrl` | `echo.usebruno.com` | Where `Log Prize Handout` posts. Empty means don't log |

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
| **Booth-Tiered** | Same odds, two prizes — three 🐶 wins a t-shirt, two wins a sticker pack |
| **Rehearsal** | `winRate: 1`, banner says REHEARSAL, win line says there's nothing to hand over, `logUrl` empty so practice wins aren't logged |

### Take it to your own event

Copy an environment, change the words, and it's a different machine — there
is no code to touch. A stricter, API-flavoured variant:

```yaml
# environments/My-Event.yml
name: My-Event
variables:
  - name: rngBaseUrl
    value: https://www.random.org
  - name: winRate
    value: { type: number, data: "0.2" }   # one spin in five
  - name: winSymbol
    value: "200"                            # word marks work as well as emoji
  - name: missSymbol
    value: "404"
  - name: prizeName
    value: STICKER PACK
  - name: eventLabel
    value: SPIN FOR A 200
  - name: winsLabel
    value: PACKS WON
  - name: logUrl
    value: https://echo.usebruno.com
```

Pick it in the environment selector and the booth screen retunes while it is
open — no restart, no reload.

## How the odds work

Real slot machines don't roll a symbol, they roll a **stop** on a physical
reel, and how many stops each symbol occupies is what sets the odds. This
machine works the same way: a strip of `maxStop + 1` stops split between two
symbols, and random.org picks the stop for each reel.

Two symbols split evenly makes every reel a fair coin, and two-or-more heads
out of three lands on 4 of the 8 possible outcomes — which is why the default
is a true 50/50 rather than a tuned approximation.

## What's in here

The repo root *is* the collection, so there are two kinds of thing here.

**Bruno's** — what shows up in the app sidebar:

| Path | What it is |
|---|---|
| `opencollection.yml` | The collection itself: name, docs, and the ignore list below |
| `Bruno Slot Machine.yml` | The Collection App — the booth screen, as one HTML file |
| `Machine/Spin Reels.yml` | The spin: one true random integer per reel |
| `Machine/Check RNG Quota.yml` | Pre-flight: true random bits left today |
| `Machine/Log Prize Handout.yml` | A JSON receipt for each prize handed over |
| `environments/` | Booth, Booth-Tiered, Rehearsal |

**The repo's** — kept out of Bruno's way:

| Path | What it is |
|---|---|
| `README.md` | This file |
| `LICENSE` | MIT |
| `images/` | Screenshots, and [where each one goes](images/README.md) |
| `.gitignore` | `bru run` artefacts, editor droppings |

Bruno would otherwise show every directory in the collection root as a
folder, so `images/` would sit in the sidebar next to `Machine`. The
collection tells it not to:

```yaml
# opencollection.yml
extensions:
  bruno:
    ignore:
      - node_modules
      - .git
      - .github
      - images
```

**Add any new repo-side directory to that list**, or it turns up in the app.

## If the wifi dies

The spin fails loudly rather than quietly faking it. The error offers **SPIN
WITH LOCAL RNG**, which falls back to `crypto.getRandomValues` — and the badge
in the footer changes to **LOCAL CRYPTO RNG** for as long as that's what's
drawing the reels, so nobody is told random.org picked their shirt when it
didn't.

<!-- screenshot: ![The error card offering local RNG](images/06-rng-error.png) -->

random.org meters free use per IP (1,000,000 bits a day, refilled at midnight
UTC). A spin costs a few dozen bits, so a booth won't run it out — but a
conference shares one public IP across a lot of laptops, which is what
`Check RNG Quota` is for.

## Live drawing, by design

There are no claim codes and no ticket numbers. The winner is standing in
front of you, so the screen says they won and someone hands them a shirt. A
losing spin points them at the swag table anyway, so nobody walks off with
nothing.

## Keeping a tally

Nothing is redeemed later, but it is still worth knowing what went out.
`Log Prize Handout` POSTs one JSON receipt per prize — the prize, the reels,
the stops, which RNG drew them and when:

```json
{ "prize": "BRUNO T-SHIRT", "reels": "🐶 🐶 ⚡", "stops": [54, 11, 5],
  "rng": "random.org", "spin": 37, "at": "2026-09-04T13:05:14.921Z" }
```

<!-- screenshot: ![The Handout Receipt visualizer](images/07-handout-receipt.png) -->

`logUrl` decides where that lands. It ships pointing at
[echo.usebruno.com](https://echo.usebruno.com), which mirrors the payload
straight back — honest about being a stand-in, and the collection still has
no backend of its own. Point it at your own webhook or sheet and the booth
starts keeping a real record: one environment variable, no code.

Only real handouts are logged. The app posts on a win and never on a loss,
and an empty `logUrl` — which is what **Rehearsal** has — turns logging off
entirely, so a practice win can't reach the tally. A failed log never
interrupts the booth: the winner already has their shirt, and the footer
shows the failed call without putting a dialog in front of a queue.

It is also the request to open when someone asks what else Bruno does. It is
the only POST here, the only JSON body, and the only one with a pre-request
script — which stands in a whole handout, so the request and its tests work
from the sidebar with the app closed.

## If something looks wrong

| What you see | What it is |
|---|---|
| No **Bruno Slot Machine** entry in the sidebar | Bruno is older than v4.0.0 — Apps don't exist there. The requests still work |
| App opens but says **BRUNO CONTEXT NOT FOUND** | The HTML is being previewed outside Bruno. It needs `bru.ctx`, so it only runs as a Bruno App |
| Spins fail with **COULD NOT REACH THE RNG** | No internet, or random.org is unreachable. Take the **SPIN WITH LOCAL RNG** offer — the footer badge changes to LOCAL CRYPTO RNG so nobody is misled |
| Spins fail with **HTTP 503** | This IP is out of true random bits for the day. Run `Check RNG Quota` to confirm; it refills at midnight UTC |
| `bru run "Machine" --env Rehearsal` says **1 Skipped** | Working as intended. Rehearsal has no `logUrl`, so `Log Prize Handout` skips itself — the run is still green |
| Emoji render as `ðŸ¶` in a visualizer | A visualizer that emits raw UTF-8. Both visualizers here escape non-ASCII to numeric references to avoid it |
| The tally is wrong after testing | Press **RESET COUNTER** on the app. The count lives in the booth machine's local storage, not in the collection |

## Making changes

The collection is plain YAML and reads as documentation, so most edits are a
text edit away. Everything that a booth would want to change — odds, prizes,
symbols, wording — is an environment variable, not code, so try that route
first.

Every request carries its own docs in Bruno's **Docs** tab, the `Machine`
folder has a docs tab of its own, and so does the collection. Those are the
same words as this README; if you change behaviour, change them together.

Before you hand it to someone else:

```bash
bru run "Machine" --env Booth        # 3 passed, 12 tests
bru run "Machine" --env Rehearsal    # 2 passed, 1 skipped
```

## License

[MIT](LICENSE). Take it to your own booth, your own conference, your own
prize table — that's what it's for.
