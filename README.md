# Desktop Tamagotchi

> A lightweight virtual pet that lives in your system tray — built with Python and Tkinter, no GUI framework required.

![Python](https://img.shields.io/badge/Python-3.7%2B-blue?logo=python&logoColor=white)
![Platform](https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-lightgrey)
![Status](https://img.shields.io/badge/Status-Stable-brightgreen)

---

## Overview

Desktop Tamagotchi is a single-file Python application that runs a virtual pet simulation in a compact always-on-top window. The pet has three tracked stats — hunger, happiness, and energy — that decay over time and require the user's attention. When the window is minimised, the pet lives in the system tray and can trigger OS-level notifications when a stat crosses a critical threshold.

---

## Features

- **Live stat simulation** — hunger, happiness, and energy decay on a 30-second tick cycle
- **Four-frame ASCII sprite animation** — sprite cycles every 600 ms; expression changes with status
- **Seven distinct states** — `happy`, `normal`, `hungry`, `sad`, `tired`, `sleep`, `dead`
- **Sleep mode** — toggling sleep pauses decay and recovers energy; hunger still drains slowly
- **System tray integration** — minimises to tray via `pystray`; tray icon colour reflects pet status
- **Tray context menu** — feed or play without opening the main window
- **OS notifications** — critical alerts via `plyer` with a rate-limiting guard (90 s minimum interval)
- **Fallback popup** — if `plyer` is unavailable, a small auto-dismissing Tk `Toplevel` is used instead
- **Graceful degradation** — tray and notification libraries are optional; the app runs without them

---

## Project Structure

```
desktop-tamagotchi/
│
├── tamagotchi.py        # Full application — model, view, controller in one file
├── requirements.txt     # Optional dependencies
└── README.md
```

### `tamagotchi.py` internal layout

| Section | Description |
|---|---|
| `Pet` class | Data model — stats, age, status property, tick/feed/play/sleep methods |
| `FRAMES` | Four-frame ASCII sprite dictionary keyed by status string |
| `COLORS` / `BG_COLORS` | Accent and background colour maps per status |
| `STATUS_TEXT` | Human-readable label map per status |
| `TamagotchiApp._build_ui()` | Constructs the Tkinter widget tree |
| `TamagotchiApp._update_ui()` | Redraws all dynamic elements from current pet state |
| `TamagotchiApp._animation_loop()` | 600 ms Tk `after` loop for sprite animation |
| `TamagotchiApp._game_loop()` | 30 s Tk `after` loop that calls `pet.tick()` |
| `TamagotchiApp._check_notifications()` | Evaluates thresholds and dispatches alerts |
| `TamagotchiApp._notify()` | Notification dispatcher — plyer, then popup fallback |

---

## Requirements

**Python 3.7 or newer.**

### Optional dependencies

```
pystray    # System tray icon and context menu
Pillow     # Required by pystray for icon rendering
plyer      # OS-level desktop notifications
```

Install all at once:

```bash
pip install pystray pillow plyer
```

The application runs without any of these packages — tray and notification features are disabled automatically if imports fail.

---

## Installation

```bash
# Clone the repository
git clone https://github.com/your-username/desktop-tamagotchi.git
cd desktop-tamagotchi

# Install optional dependencies
pip install -r requirements.txt

# Run
python tamagotchi.py
```

---

## Usage

After launch, the window appears in the top-left corner of the screen and stays on top of other windows.

### Actions

| Button | Effect |
|---|---|
| Feed | Restores +30 hunger, +5 happiness |
| Play | Restores +25 happiness, costs -15 energy, -10 hunger |
| Sleep / Wake up | Toggles sleep mode — energy recovers, hunger still drains |
| Hide to Tray | Withdraws the window; pet continues running |

### Tray context menu

| Item | Effect |
|---|---|
| Open | Restores the main window |
| Feed | Feeds the pet without opening the window |
| Play | Plays with the pet without opening the window |
| Quit | Stops the application |

---

## How It Works

### Simulation model

`Pet.tick()` is called every 30 seconds. Each call applies the following deltas:

| State | Hunger | Happiness | Energy |
|---|---|---|---|
| Awake | -4 | -2 | -3 |
| Sleeping | -3 | — | +10 |

If both `hunger` and `energy` reach 0 simultaneously while awake, `pet.alive` is set to `False` and the pet enters the `dead` state.

### Status priority

The `status` property evaluates conditions in order of severity:

```
dead > sleep > hungry (< 20) > sad (< 20) > tired (< 20) > happy (all > 70) > normal
```

### Notification rate limiting

`Pet.should_notify(kind, interval=90)` stores the timestamp of the last notification per kind in `last_notified`. A notification is only dispatched if more than 90 seconds have passed since the previous one of the same kind, preventing alert spam.

### Architecture

The application uses a single `after`-based event loop pattern rather than threads for UI updates, which is the correct approach for Tkinter. The system tray runs in a dedicated daemon thread (required by `pystray`). All tray-triggered callbacks marshal back to the main thread via `root.after(0, ...)`.

---

## Configuration

Key values are defined as instance attributes on `Pet.__init__` and can be adjusted directly:

```python
self.name   = "Buddy"   # Pet display name
self.hunger = 80        # Starting hunger  (0–100)
self.happy  = 80        # Starting happiness (0–100)
self.energy = 80        # Starting energy   (0–100)
```

Tick intervals are set in `TamagotchiApp`:

```python
self.root.after(600,    self._animation_loop)   # Sprite frame rate (ms)
self.root.after(30_000, self._game_loop)        # Stat decay rate   (ms)
```

Notification throttle interval is controlled by the `interval` parameter in `should_notify()` (default: 90 seconds).

---

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "feat: describe your change"`
4. Push and open a Pull Request

---

## Acknowledgements

Built with [Tkinter](https://docs.python.org/3/library/tkinter.html), [pystray](https://github.com/moses-palmer/pystray), [Pillow](https://python-pillow.org/), and [plyer](https://github.com/kivy/plyer).