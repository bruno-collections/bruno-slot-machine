# Screenshots

Drop screenshots in this folder, then uncomment the matching line in the
top-level [README](../README.md). Every slot is already marked there:

```bash
grep -n "screenshot:" README.md
```

Each one looks like this, so filling a slot is deleting `<!-- screenshot:`
and the closing `-->`:

```markdown
<!-- screenshot: ![The booth screen, mid-spin](images/01-booth-screen.png) -->
```

## The slots

| File | What to capture |
|---|---|
| `01-booth-screen.png` | The hero shot — the app mid-spin or on a win, full window |
| `02-open-collection.png` | Bruno's **Collection → Open Collection** dialog |
| `03-app-in-sidebar.png` | The sidebar with **Bruno Slot Machine** and the `Machine` folder |
| `04-show-api.png` | The **SHOW API** overlay, with the stops mapped to reels |
| `05-environment-selector.png` | The environment picker showing Booth / Booth-Tiered / Rehearsal |
| `06-rng-error.png` | The error card offering **SPIN WITH LOCAL RNG** |
| `07-handout-receipt.png` | `Log Prize Handout`'s visualizer — the Handout Receipt card |

Nothing breaks if a slot stays empty: the placeholders are HTML comments, so
they render as nothing at all until you fill them in.

This folder is listed under `extensions.bruno.ignore` in `opencollection.yml`,
which is what keeps it out of Bruno's sidebar — the collection root doubles as
the repo root, so any directory Bruno isn't told to ignore shows up as a folder
in the app. Add new asset directories there too.

## Conventions

- **PNG**, about **1600px** wide. GitHub scales them down; anything wider is
  just bytes.
- Capture the booth screen at **16:9** so it matches what a projector shows.
- Use the **Booth** environment unless the shot is specifically about another
  one — the prize names and banner should match what the README describes.
- Crop to the part you're talking about. A full 4K desktop screenshot of one
  dialog reads as nothing at all on a phone.

## Using them in Bruno's docs tabs

The collection, folder and request **Docs** tabs are markdown too, so an
image can go there as well. Relative paths like `images/01-booth-screen.png`
resolve against the repo, not against Bruno's renderer, so they may well come
up blank in the app — worth a quick test before relying on it. Once this repo
is public, the reliable form is the absolute raw URL:

```markdown
![Booth screen](https://raw.githubusercontent.com/<org>/<repo>/main/images/01-booth-screen.png)
```
