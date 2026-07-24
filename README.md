# Titans Sub Tracker — README

A courtside web app for tracking substitutions and playing time for the Titans basketball team. Runs entirely in the browser — no login, no server, no internet connection needed once loaded (except to fetch fonts on first load).

---

## Getting it onto your phone

1. Upload `index.html` and `icon.png` to a public GitHub repo (same filenames).
2. In the repo: **Settings → Pages → Deploy from a branch → main / (root) → Save**.
3. Visit `https://<your-username>.github.io/<repo-name>` in **Safari** (iPhone) or **Chrome** (Android).
4. Tap **Share → Add to Home Screen** (iPhone) or **⋮ → Add to Home screen** (Android).

To push an update later, re-upload the changed file(s) with the same filename and commit. If your phone still shows the old version, delete the home screen icon and re-add it — phones cache these shortcuts aggressively.

---

## Weekly workflow

1. Open the app — it always starts on the **"This week"** screen.
2. Tick which of the 7 players are playing today (unticked players grey out and are excluded entirely).
3. Fix any name spelling if needed — names are remembered for next time.
4. Tap **Start game**. The starting lineup (who's on court vs bench) is shuffled randomly each time.
5. Tap **Start** to begin the clock. Play the game — pause during genuine stoppages if you want playing-time tracking to reflect actual game time rather than dead time.
6. The app pauses itself automatically at half time (20:00) and again at full time (40:00).
7. Use **↺** any time to reset and start a fresh game with the same roster (also re-shuffles the starting lineup). Use **✎** to go back and change who's playing.

---

## How the rotation logic works

**Court size is fixed at 5.** How many players are ticked in each week determines the bench size:

| Players ticked | On court | Bench | Rotation |
|---|---|---|---|
| 5 | 5 | 0 | No subs — everyone plays the whole game |
| 6 | 5 | 1 | 1 rotating bench spot |
| 7 | 5 | 2 | 2 rotating bench spots |

**Sub interval.** The game (40 minutes total) is divided evenly by the number of players ticked in, so that — in principle — everyone gets an equal number of rotations through the bench:

```
sub interval = game length ÷ number of players playing
```

For 7 players that's 40 ÷ 7 ≈ 5:43. For 6 players it's 40 ÷ 6 ≈ 6:40. This interval is when the buzzer/alert fires — not necessarily when a swap actually happens (see "fair time" below).

**Fair time target.** Every player's target playing time for the whole game is:

```
fair target = (5 ÷ number of players playing) × game length
```

E.g. for 7 players: 5/7 × 40 min ≈ 28:34. A bench player is marked **"Fair time"** (grey outline) once they're within a **±2 minute tolerance** of that target — meaning they don't need to be subbed back on yet, even if it's technically their "turn." This is what prevents pointless late-game swaps once someone's already had a roughly fair amount of court time.

**Live indicators on court/bench cards:**
- **Green "Next up"** (bench) — this player is still meaningfully short of their fair target and should come on next.
- **Grey "Fair time"** (bench) — this player is within tolerance of their target; no urgency to sub them on.
- **Dashed red outline, "Off soon"** (court) — a live, continuously-updating preview of who's projected to come off at the next alert, based on current standings. Useful for giving players a heads-up before the buzzer.
- **Solid red outline, "Due off"** (court) — the buzzer has fired; this player should come off now.

**When the alert fires**, the app only asks for the number of swaps actually needed (i.e. only bench players who are genuinely short of their fair target, not the full bench count) — so a shift might call for 1 swap instead of 2, especially near the end of the game once someone's target is already met.

**Alert behaviour:**
- A loud buzzer sound (3 beeps, full volume + limiter).
- Phone vibration (**Android only** — iOS Safari doesn't support the Vibration API; there's no workaround for this from a web page).
- A pulsing red banner reading "Time to substitute" (tap to dismiss).
- Red outlines on the court players due off.

## Making the swap

Two ways:
- **Manual:** tap a bench player, then tap a court player, to swap those two specifically.
- **⇄ Sub now button:** appears active (with the count of subs needed) once an alert fires. One tap performs the correct swap(s) automatically — bringing on whoever's still short of their target and taking off whoever's played the most. Disabled/greyed out at all other times, and permanently disabled on 5-player weeks (no bench to manage).

The alert (banner, red outlines, "Sub now" button) clears automatically once the required number of swaps has been made, however you choose to make them.

---

## The clock

- Counts **actual running time**, not real-world time — it only advances while you've pressed Start/Resume, so pausing during dead-ball time doesn't count against playing-time tracking.
- Uses a wall-clock timestamp under the hood (not just a tick counter), so if your phone's screen locks or the browser gets suspended, the clock catches up to the correct time the moment you return rather than losing or freezing time.
- The app requests a **screen wake lock** while running, to try to stop the phone auto-locking during a game. This can't override you manually pressing the power button — if you do that, the screen (and any sound/vibration) is suspended by iOS until you unlock again, though the clock will still catch up correctly once you do.

---

## Known limitations

- **No iOS vibration.** Apple has never implemented the Vibration API in Safari, including for home-screen web apps. Sound and visual alerts work fine; vibration only works on Android.
- **No "critical alert" override.** A web page cannot bypass a phone's silent switch or Do Not Disturb — that capability is restricted to specific native apps Apple has separately approved. Keep the ringer on for the buzzer to be audible.
- **Data isn't saved mid-game.** If you close the tab/app during a game, playing-time progress for that game is lost (roster names and this-week's ticked players ARE remembered, via the app's persistent storage — just not in-progress game state).
- **Home screen icon caching.** iOS in particular can hold onto an old cached version after you push an update. If changes don't seem to appear, delete the home screen shortcut and re-add it from a fresh Safari tab.

---

## Adjustable settings (for future edits)

These live near the top of the `<script>` section in `index.html`:

| Setting | Current value | What it controls |
|---|---|---|
| `HALF_LEN` | 20 minutes | Length of each half |
| `FAIRNESS_TOLERANCE_SEC` | 120 seconds (±2 min) | How close to the fair-time target counts as "fair" |

`GAME_LEN` (total game length) is derived automatically from `HALF_LEN`, so changing halves updates the sub interval and fair-time targets everywhere without needing separate edits.
