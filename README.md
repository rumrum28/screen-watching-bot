Screen Watcher Bot — Step-by-Step Build Prompt

22 Oct 2026 · @John Eric

How to use this prompt

Build the bot one step per session: paste the Master Context, then the prompt for the step you are on. Each step ends with working code you can run, so nothing is lost if your tokens run out mid-project.

1. Start a new chat and paste the Master Context block (next section).
2. Paste the prompt for your current step, exactly as written.
3. Save every file the AI gives you into your project folder and test it.
4. Tick the step off below. If a step breaks or your tokens run out, use the Resume prompt and paste your current code.

Progress tracker

[ ] Step 1 — Project setup and config profiles
[ ] Step 2 — Screen region selector
[ ] Step 3 — Capture loop and hotkeys (start, pause, quit)
[ ] Step 4 — Health bar color detection
[ ] Step 5 — Image and pixel matching
[ ] Step 6 — Action system (key press, left/right click)
[ ] Step 7 — Moving object detection and auto-click
[ ] Step 8 — Profile manager for other games
[ ] Step 9 — Debug preview, logging and tuning

Master Context (paste first, every session)

This block tells the AI the whole plan, so it writes code that fits the steps before and after.

You are helping me build a personal Python tool called "Screen Watcher".
I run it with `python main.py`, no executable needed. I write and test on macOS but play on Windows,
so every file must work on BOTH (detect the OS with platform.system()).
On macOS: explain the Screen Recording + Accessibility permissions and handle Retina 2x scaling.
On Windows: handle display scaling (DPI awareness).

GOAL
The tool watches a region of my screen while I play a game and reacts to what it sees:
1. Health bar: it reads my life bar color/fill. Green = full, orange = about half,
   red = 20% or lower. When it is red (or below a % I set), it presses my heal key.
2. Image/pixel match: if an image I upload (or capture from the screen) or a specific
   pixel color appears, it triggers an action.
3. Moving objects: if something moves inside a region I select, it clicks on it.
4. Actions are configurable: key press, left click, or right click, with cooldowns.
5. It must be game-agnostic: every game gets its own profile (JSON) with its own
   regions, colors, images, keys and rules.

TECH STACK
- mss (fast screen capture), numpy, opencv-python (color + template + motion detection)
- pydirectinput for keys/clicks in games (fallback: pyautogui), pynput or keyboard for hotkeys
- tkinter for the region selector and simple dialogs
- JSON files in a /profiles folder for per-game settings

PROJECT STRUCTURE (target)
screen_watcher/
  main.py            # entry point, CLI: python main.py --profile <name>
  config.py          # load/save/validate profiles
  selector.py        # drag-to-select screen region + color picker
  capture.py         # mss capture loop with FPS limit
  detectors/
    health_bar.py    # HSV color ranges + fill percentage
    template.py      # image / pixel matching
    motion.py        # moving object detection
  actions.py         # key press, left/right click, cooldowns
  rules.py           # trigger -> action engine
  debug_view.py      # optional preview window with detections drawn
  profiles/          # one JSON per game
  templates/         # saved images to match
  requirements.txt

RULES FOR YOU
- We build this in steps. Only do the step I give you. Do not jump ahead.
- Give complete, runnable files (not fragments) for every file you create or change.
- Keep code simple and commented; I am a hobbyist.
- Always keep a panic/quit hotkey and a pause hotkey working.
- At the end of each step, tell me: how to test it, and what the next step is.

Step prompts

Paste one of these right after the Master Context. Each step builds on the files from the step before.

Step 1 — Project setup and config profiles

STEP 1: Set up the project.

- Create the folder structure from the Master Context (empty placeholder files are fine for later steps).
- Write requirements.txt and the exact pip install command.
- Write config.py: load, save and validate a profile JSON. If a profile is missing, create a default one.
- Design the profile JSON format. It must hold: profile name, monitor index, FPS limit,
  hotkeys (start/pause/quit), regions (name + x, y, width, height), health bar settings
  (region, HSV ranges for green/orange/red, heal threshold %), templates (image path + threshold),
  motion settings, and a list of rules (trigger -> action + cooldown).
- Write a sample profiles/example_game.json with comments explaining each field in your reply.
- Write a minimal main.py that loads a profile via --profile and prints it.

Step 2 — Screen region selector

STEP 2: Build selector.py.

- A full-screen semi-transparent tkinter overlay where I click-and-drag a rectangle to select a region.
- Support multiple monitors (use mss to get monitor positions) and Windows display scaling (DPI awareness).
- A color picker mode: I click one pixel and it shows the RGB and HSV values and saves them.
- A "capture template" mode: I drag a rectangle and it saves that area as a PNG in /templates.
- Save selected regions into the active profile with a name I type (e.g. "health_bar").
- Add CLI commands: python main.py --profile X --select-region NAME / --pick-color / --capture-template NAME

Step 3 — Capture loop and hotkeys

STEP 3: Build capture.py and the main loop.

- Use mss to grab only the regions the profile needs (not the whole screen) for speed.
- Limit the loop to the profile's FPS (default 20) to save CPU.
- Global hotkeys that work while the game is focused: F8 = start/resume, F9 = pause, F10 = quit
  (configurable in the profile).
- Print status changes to the console. The loop does nothing yet except capture and report FPS.

Step 4 — Health bar color detection

STEP 4: Build detectors/health_bar.py.

- Convert the health bar region to HSV and count pixels inside the green, orange and red ranges.
- Return the dominant state ("green", "orange", "red", "unknown") AND an estimated fill %
  (colored pixels along the bar's length vs total length; bars fill left to right by default,
  allow right-to-left or vertical in the profile).
- Add a calibration command: python main.py --profile X --calibrate-health
  It asks me to have full health, captures the bar, and suggests HSV ranges automatically.
- Use smoothing (e.g. state must hold for 3 frames) so one flicker does not trigger a heal.
- For now just print state and % every frame in the loop.

Step 5 — Image and pixel matching

STEP 5: Build detectors/template.py.

- Template matching with cv2.matchTemplate on a region, using the threshold from the profile.
- Return found / not found, confidence, and the match center in SCREEN coordinates.
- Support an uploaded image file path or an image captured in Step 2.
- Add a simple pixel-color trigger: check one (x, y) point against a target color with a tolerance.
- Optional: grayscale matching and a few scales (0.8x-1.2x) for games that resize UI.
- Print detections in the loop.

Step 6 — Action system and rules

STEP 6: Build actions.py and rules.py.

- Actions: press_key(key, hold_ms), left_click(x, y), right_click(x, y), click_at_match.
- On Windows use pydirectinput (works in most DirectX games); on macOS, or if it fails, use pyautogui/pynput. Pick automatically by OS.
- Each rule has its own cooldown (e.g. heal at most every 2 seconds) and an enabled flag.
- Rule engine: each frame, evaluate every rule's trigger (health state/%, template found,
  pixel color, motion found) and fire its action if the cooldown has passed.
- Example rules in the profile: "if health < 20% press '1'", "if template 'loot_icon' found left-click it".
- Add a --dry-run flag that logs actions instead of performing them, so I can test safely.

Step 7 — Moving object detection and auto-click

STEP 7: Build detectors/motion.py.

- Detect moving objects in a selected region using frame differencing or cv2.createBackgroundSubtractorMOG2.
- Clean the mask (blur, threshold, dilate), find contours, filter by min/max area from the profile.
- Optional color filter (only objects within an HSV range) to ignore effects and UI.
- Ignore zones: let the profile exclude areas (like my own character or the health bar).
- Return the target to click: largest or closest-to-center object, in screen coordinates.
- Add a motion trigger to the rule engine with its own cooldown and left/right click choice.
- Warn me that my own camera movement can look like motion, and offer a setting to
  pause motion clicks when the whole region changes a lot at once.

Step 8 — Profile manager for other games

STEP 8: Make it easy to set up a new game.

- python main.py --new-profile NAME: a guided console wizard that walks me through
  selecting regions, calibrating the health bar, capturing templates, and choosing keys/clicks
  for each rule, reusing the Step 2 and Step 4 tools.
- python main.py --list-profiles and --edit-rule commands.
- Validate the profile on load and print clear errors (missing template file, region off-screen).

Step 9 — Debug preview, logging and tuning

STEP 9: Build debug_view.py and finish up.

- python main.py --profile X --debug opens a small OpenCV window showing the watched regions
  with boxes drawn around matches/moving objects and the current health state and %.
- Log every fired action with a timestamp to logs/session.log.
- Show live FPS and per-detector time so I can see what is slow.
- Give me a tuning checklist: what to change if heals fire too early/late,
  if templates are missed, or if motion clicks the wrong thing.
- Write a short README.md with all commands.

Resume prompt (when tokens run out or a step breaks)

Paste the Master Context first, then this. Fill in the brackets and paste the files that the current step touches, not the whole project.

RESUME: I am continuing the Screen Watcher project.

- Completed steps: [e.g. 1, 2, 3]
- Current step: [e.g. Step 4 — Health bar color detection]
- Where I stopped: [e.g. "the reply cut off in health_bar.py" or "it runs but always says unknown"]
- Error message (if any): [paste the full traceback]

Here are my current files:

--- config.py ---
[paste]
--- main.py ---
[paste]
--- [other file for this step] ---
[paste]

Please continue the current step from where it stopped. Give complete files only for
the files you change, and keep everything compatible with the earlier steps.

Tip: if a reply gets cut off mid-file, type continue from the last complete line of <filename> instead of starting over.

Notes and cautions

• Online games and anti-cheat: many online games ban automated input, even simple heal bots. Use this in single-player or offline games, or where the game's rules allow macros.
• Input not working in a game: run the terminal as Administrator if the game runs as admin, and prefer pydirectinput over pyautogui. Borderless windowed mode makes capture and selection most reliable.
• Display scaling: if regions land in the wrong place, check Windows scaling (125%, 150%) — Step 2 handles this, but it is the most common bug.
• Always keep the panic key: test new rules with --dry-run first, so a bad motion rule cannot click wildly around your desktop.