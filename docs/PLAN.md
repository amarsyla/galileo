# Galileo: plan

## 1. The idea
Most students learn about gravity and motion from a textbook diagram. Many schools, including most in Kosovo where this project started, have no lab equipment to do the real experiment.

Galileo turns any phone into that lab. A student films a pendulum in slow motion, uploads the clip, and watches the physics appear on top of their own video: a glowing trail, speed arrows, a stopwatch counting swings, and a graph that draws itself as the video plays. After a few runs with different string lengths, Galileo helps them find the pattern and measure gravity with their own hands.

**Who it's for:** students around 12 to 16 and their teachers. No physics background needed to use it. Every number comes with a plain-language explanation.

**What makes it real science, not a cartoon:** the visuals come from actual measurements. Tracking is frame by frame at 240 fps, distances are calibrated with a printed marker, timing comes from the video's own timestamps, and every result carries an error bar.

## 2. The student experience
1. **Set up.** Tie a small bright object to a string, hang it from a table edge, and place the printed Galileo marker next to it. The app shows a one-screen setup card.
2. **Guess first.** Before filming, the app asks a prediction question: "If the string is longer, will it swing faster or slower?" The student taps an answer.
3. **Film.** Use the phone's built-in slow-motion camera (240 fps), then upload the clip.
4. **Tap the object.** One tap on the swinging object, and Galileo locks on.
5. **Watch the physics.** The video plays back with animated overlays (section 3).
6. **Repeat with other lengths.** Each run adds a dot to a chart. After 3 to 5 runs, the pattern appears.
7. **Discover the rule.** Galileo reveals the result: "Your pendulum says gravity is 9.77 ± 0.04 m/s². The textbook says 9.81. You measured gravity!" The prediction from step 2 is checked against the data.
8. **Next challenge.** Galileo suggests what to try next ("Try a 1.2 m string to make your answer even more precise").

## 3. Visuals over the video
All overlays are drawn live in the browser on top of the video, synced frame by frame, so students can pause, scrub, and slow down.

| Overlay | What it shows | Plain-language caption |
|---|---|---|
| Glowing trail | The path of the object, colored by speed | "Fastest at the bottom, slowest at the ends" |
| Speed arrow | An arrow that grows and shrinks with speed | "Watch the arrow stretch as it swings down" |
| Swing counter + stopwatch | Ticks once per full swing, shows the time per swing | "One swing takes 1.58 seconds" |
| Real ruler | Distances in centimeters, calibrated from the marker | "The string is 62 cm long" |
| Energy bars | Height energy vs motion energy, trading back and forth | "Energy keeps switching between height and speed" |
| Live graph | Position over time, drawing itself as the video plays | "This wave is your pendulum's heartbeat" |
| Ghost frames | Freeze-frame showing several moments at once (strobe effect) | "Each ghost is 1/10 of a second apart" |

**The big reveal (multi-run):** each run flies in as a dot on a "swing time vs string length" chart, the best-fit curve snaps through the dots, and the gravity number counts up to its final value. A bonus animation shows the same pendulum on the Moon swinging about 2.5 times slower.

## 4. How it works

```
Phone slow-mo video (240 fps)
   |  upload from the web app
   v
Backend pipeline
   1. Check: real frame timestamps and frame rate (reject clips that lost slow motion)
   2. Calibrate: find the printed marker, convert pixels to centimeters
   3. Track: SAM 3.1 follows the tapped object in every frame -> position over time
   4. Quality check: simple automatic checks + Muse Spark watches a preview and flags bad runs
      ("the object left the frame", "the string was touched")
   5. Analyze: Muse Spark Open proposes candidate models (simple swing, slowing swing, air drag);
      a fitting tool tests each one against the data and the best fit wins
   6. Across runs: swing time vs string length -> gravity with an error bar,
      plus a suggested next experiment
   7. Explain: Muse Spark writes short, student-level captions from the measured numbers
   v
Web app: tap to track, animated overlays, reveal chart, simple lab notebook
```

**Rules the system follows**
- **The AI never makes up numbers.** Muse Spark reasons and explains, but every number comes from measurement and fitting code, and captions are generated from those numbers.
- **Timing comes from the video file,** not from a model.
- **Muse Image is only for illustrations** (setup card, notebook cover), never for data charts.

**Meta models used**
| Model | Role |
|---|---|
| SAM 3.1 | Tracks the object in every frame from one tap |
| Muse Spark | Watches runs to catch bad ones, writes student-level explanations and the lab notebook |
| Muse Spark Open | Agent that proposes and compares physical models using a fitting tool |
| Muse Image | Illustrated setup card and notebook art |

**Stack:** Python (FastAPI, OpenCV, SciPy, ffmpeg) backend on Modal. Mobile-first web app in React + TypeScript, overlays on Canvas synced with `requestVideoFrameCallback`. Baseten hosts Muse Spark Open.

## 5. Accuracy check
A test set of real pendulum clips with tape-measured string lengths. A command-line tool runs the whole pipeline on all of them and reports how close the measured gravity is to 9.81 m/s². This runs after every major change, and the results table goes in the README.
