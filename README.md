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
3. Refresh rate: anything. Once or twice a day is plenty — both screens only
   change at midnight, and a slow refresh is easier on the battery.
4. Open the **Markup / Edit Markup** editor, pick the **Full** layout, and paste
   in the matching `.liquid` file.
5. Use the live preview to confirm it renders, then **Save**.
6. Add both plugins to your **Playlist**.

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
  tz_offset_seconds: -14400,
  facts: (split("\n") | map(select(length > 0)))
}' data/facts.txt > data/hawaii.json
```

Keep facts under about 80 characters so they stay large and readable.

## Timezone

`tz_offset_seconds` in `hawaii.json` is set to **-14400 (US Eastern, daylight
time)**. Change it if that is wrong.

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
- **Class names** — `view view--full` and `title_bar` come from TRMNL's design
  system at usetrmnl.com/framework. Everything doing real layout work is an
  inline style, so the screen still renders correctly even if a class name has
  changed on their end.
