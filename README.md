# mockbball — Practice Code Window

A Sportscode-style coding window for basketball practice film. Dark mode by default (toggle in the top bar). Code possessions and stats with hotkeys while the video plays; every possession becomes a clip you can filter and play back.

## Open it

**https://mpkcb21.github.io/mockbball/**

It's hosted on GitHub Pages straight from `main`, so any push updates the live site within a minute or two. Nothing to install.

For working on it locally, serve the folder (YouTube embeds don't work on pages opened straight from disk):

```sh
python3 -m http.server 8000   # then open http://localhost:8000
```

## Practices tab

Every practice lives here with its date, name and video link.

- **New practice:** give it a name and date, paste the YouTube link (live or recorded), and press *Create & start coding*. Or use *…with a video file* for film on your computer.
- **The video link can be changed later.** If you coded from a local file and later uploaded the film to YouTube, paste the link into that practice's row. Timestamps carry over as long as it's the same video.
- **Tick several practices** to see their combined stat sheet underneath. Click any blue number to watch every matching possession across all of them, grouped by practice. The player switches videos on its own between practices.
- *Code* opens a practice for coding, and *Review* opens it on its own.

Notes on video sources:
- **YouTube:** for live streams, turn on DVR in the stream settings so you can scrub back. The stream must be Public or Unlisted.
- **Local files** never leave your computer. Reviewing a file-based practice needs that file loaded in this visit; the app tells you which file to load.

## How clipping works

Press **P** (or GOLD / BLUE POSSESSION) when a possession starts. That marks the start of a new clip and ends the previous one. **Z** ends a clip without starting a new one (dead ball, water break). Every stat is stamped with the video time and lands in whichever possession it falls inside.

## Hotkeys

The full key is in the top-right of the app. Typical flow: **number → letter** (e.g. `7` then `T` = turnover on Player 7).

| Key | Action | Key | Action |
|---|---|---|---|
| 1–9, 0 | Select Player 1–9, Player 10 | P | New possession (switch team) |
| G / H | Gold / Blue possession | Z | End / dead ball |
| A | Assist | Y | Assist opportunity |
| T | Turnover | O | Offensive rebound |
| F | Foul drawn | W | Offensive foul |
| R | Transition created | D | Defensive rebound |
| L | Steal | B | Block |
| E | Deflection | Q | Charge taken |
| V | Defensive foul | C | Activity |
| K | Ball screen defense | I | Missed boxout |
| U | Missed tag-up | M | Defensive mistake |
| S | Shot chart | + / X | Made / missed FT |
| Space | Play / pause | ← → (⇧) | Back / forward 3s (10s) |
| [ ] | Prev / next clip | ⌘/Ctrl Z | Undo |

In the shot chart: click the spot, then `0–9` shooter, `+`/`−` make/miss, `Enter` to log, `Esc` to close.

## Reviewing after practice

Open **Review** (top bar, or *Review* on a practice in the Practices tab). The coding buttons are hidden and the stat sheet sits next to the video. Reviewing several practices shows their combined numbers.

- Click any blue number in the stat sheet to play every possession behind it. For example, clicking Player 7's **TO** cell plays each possession where Player 7 committed a turnover, back to back.
- Inside each possession row, the stat that matched is outlined. Click any tag to jump straight to that moment (a few seconds before it).
- **Shot chart:** below the box score, every shot from the practices in view. Click a player's name to chart just them (click again for everyone). Filter by result, zone, 3 type or grade, or click a row in the zone table (Rim, Paint, Non-paint 2, 3PT, each 3 type) to show only those shots. Click any dot to watch that shot, even if it's from a different practice than the one loaded.
- To play every possession a player recorded a stat in, pick them in the Clips panel's **Player** filter.
- Hotkeys in Review mode only control video (Space, arrows, `[ ]`), so nothing gets coded by accident.

## Cloud sync (Supabase)

Each save writes three tables:

| Table | One row per | Key columns |
|---|---|---|
| `sessions` | practice | full session JSON, used to reopen it in the app |
| `possessions` | possession | `n`, `team`, `start_s`, `end_s`, `points`, `video_url` |
| `events` | stat / shot / FT | `t` (seconds), `stat`, `player`, `possession_n`, `video_url` |

`video_url` is a YouTube link at that timestamp (empty for local files), so the tables are useful outside the app too, e.g.:

```sql
select player, stat, t, video_url from events where player = 'Player 7' and stat = 'TO';
```

Setup:

1. Create a free project at supabase.com.
2. In **SQL Editor**, run the setup SQL from the app's **☁ Cloud** dialog (it's safe to re-run if you set up the older single-table version).
3. Open the app at https://mpkcb21.github.io/mockbball/, click **☁ Cloud**, paste the project URL and the anon/publishable key (Project Settings → API), and press *Connect*.
4. Every coach who connects with the same key sees the same list in the **Practices** tab. Practices already saved in your browser upload automatically on connect.

Anyone with the project key can read and write, so share it only with staff. If two people code the same session at once, the last save wins.

Without cloud set up, sessions still auto-save in the browser per video, and can be exported as `.json`.

## Exports

- **Filtered clips .csv**: the clip list currently shown, including a YouTube timestamp link per clip.
- **ffmpeg cut script**: a `.sh` file that cuts the filtered clips into separate mp4s from a local copy of the video.
