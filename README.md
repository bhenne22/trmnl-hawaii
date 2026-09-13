# TRMNL: Hawaii Countdown + Word of the Day

Two TRMNL private plugins for Izzy. Trip date: **August 5, 2027**.

- **Hawaii Countdown** — big "N sleeps" number plus a different Hawaii fact every day.
  330 facts, written at a 1st-grade reading level. The countdown is 328 days, so
  she will never see the same fact twice before the plane takes off.
- **Word of the Day** — a word, a kid-level definition, and an example sentence.

TRMNL rotates between them automatically via the playlist. There is no rotation
logic to build.

## Files

| File | What it is |
|---|---|
| `data/hawaii.json` | Trip date + all 330 facts. Built from `data/facts.txt`. |
| `data/facts.txt` | One fact per line. **Edit this**, then rebuild (below). |
| `data/words.json` | The word list. Edit directly. |
| `templates/countdown-full.liquid` | Paste into the countdown plugin's markup box. |
| `templates/word-full.liquid` | Paste into the word plugin's markup box. |

## Hosting the JSON

TRMNL polls a URL, so the JSON has to be reachable from the internet. Easiest path
is a **public GitHub repo** — no server, and you can edit the word list from your
phone in GitHub's web UI.

```bash
gh repo create trmnl-hawaii --public --source=. --push
```

The URLs TRMNL polls are then:

```
https://raw.githubusercontent.com/<you>/trmnl-hawaii/main/data/hawaii.json
https://raw.githubusercontent.com/<you>/trmnl-hawaii/main/data/words.json
```

Note: raw.githubusercontent.com caches for about 5 minutes. Irrelevant for a
screen that changes once a day, but it means an edit will not show up instantly.

## TRMNL setup (about 10 minutes, once)

Do this twice, once per plugin:

1. TRMNL dashboard → **Plugins** → **Private Plugin** → **New**.
2. Strategy: **Polling**. Paste the matching URL from above.
3. Refresh rate: 15 minutes. It has to be short enough to notice the daily
   stamp bump (see "Why the daily stamp" below) soon after midnight. A slow
   refresh here means the countdown sits on yesterday's number all morning.
4. Open the **Markup / Edit Markup** editor, pick the **Full** layout, and paste
   in the matching `.liquid` file.
5. Use the live preview to confirm it renders, then **Save**.
6. Add both plugins to your **Playlist**.

## Why the daily stamp

TRMNL only regenerates a plugin's screen when the polled payload changes. When
the bytes match the previous poll, the logs say:

```
Skipping: No change in data
```

Both payloads here are static JSON, so the screens froze and the countdown only
moved when something forced an unrelated re-render — which is why it once sat on
the wrong number until late morning. `"now"` in Liquid is evaluated at render
time, so a screen that never re-renders never advances.

`.github/workflows/daily-stamp.yml` fixes this by bumping a `generated_on` field
in both JSON files to the current America/Chicago date. That changes the payload
exactly once per local day and forces a re-render just after midnight. It runs
hourly and only commits when the date actually rolls over, so DST needs no
special handling.

If the countdown ever sticks again, check the workflow's run history first, then
the plugin logs for `Skipping: No change in data`.

**Do not remove `generated_on` from either JSON file** — the workflow's `sed`
looks for it, and without it the screens go stale again.


## Adding each week's words

Open `data/words.json` and add entries to the `words` array:

```json
{
  "date": "2026-09-28",
  "word": "curious",
  "definition": "Wanting to find out more about something.",
  "example": "I was curious about what was in the box."
}
```

- `date` is `YYYY-MM-DD` in your local time.
- If no entry matches today, the plugin rotates through the list instead of going
  blank, so a missed week degrades gracefully.
- Old entries can stay. They are simply never matched again.

## Editing the facts

Edit `data/facts.txt` (one fact per line, no blank lines), then rebuild:

```bash
jq -R -s --arg trip "2027-08-05" '{
  trip_date: $trip,
  trip_label: "Hawaii",
  tz_offset_seconds: -18000,
  facts: (split("\n") | map(select(length > 0)))
}' data/facts.txt > data/hawaii.json
```

Keep facts under about 80 characters so they stay large and readable.

## Timezone

`tz_offset_seconds` is set to **-18000 (US Central, daylight time)** in both
`hawaii.json` and `words.json`. Chicago switches to -21600 in winter; see below
for why that does not really matter.

This value is only a fallback. If your TRMNL account has a timezone set, the
templates use `trmnl.user.utc_offset` instead, which handles daylight saving
automatically. The offset only decides *when* the number flips over — get it
wrong and the count changes an hour early or late, not by a whole day.

## If something looks off in the preview

- **Number is blank or wrong** — your TRMNL may nest the payload under `data`.
  The templates already try both (`facts` then `data.facts`), but check the
  plugin's sample-data panel to see the actual shape.
- **Layout looks cramped** — the sizes are inline `font-size` values, so just
  change the numbers. They were picked for the 800x480 panel.
- **"Full view not available"** — this means the markup did not render. Two
  causes, in order of likelihood:

  1. **You wrapped the markup in a view div.** The TRMNL editor supplies
     `<div class="screen">` and `<div class="view view--full">` itself. Your
     markup must start at `<div class="layout">`, with `<div class="title_bar">`
     as its sibling. TRMNL's docs say the `view` classes are "specific to public
     plugin development, *not* to be used within TRMNL editor." Nesting a second
     one silently fails. The templates here are already correct.
  2. **The markup went into the wrong tab.** The editor has a separate box per
     layout (Full / Half Horizontal / Half Vertical / Quadrant). Paste into
     **Full**, and make sure it saved.

- **Vertical centering looks off** — the layout div uses `height: 100%`. If it
  does not fill the screen, change it to `height: 400px` (480px panel minus the
  title bar).

- **Class names** — `layout` and `title_bar` come from TRMNL's design system at
  trmnl.com/framework. Everything doing real layout work is an inline style
  (including `display: flex`), so the screen still renders correctly even if a
  class name changes on their end.
