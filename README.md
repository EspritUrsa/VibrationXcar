# 🎮 VibrationXcar

> Turns any XInput gamepad into an RC car using its vibration motors.

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![Windows](https://img.shields.io/badge/Windows-8.1%2B-0078D6)
![License](https://img.shields.io/badge/License-MIT-green)

---

## How it works

Place the gamepad on a smooth flat surface (a sheet of paper works great).
The vibration motors create friction force — the gamepad starts moving.
By controlling the power of each motor independently, you can drive it
forward, backward, and steer left or right.

---

## Requirements

- **Python** 3.8 or higher
- **Windows** 8.1 or higher
- **XInput gamepad** (any controller that works as an Xbox-compatible device)
- `keyboard` library (the only dependency)

---

## Installation

```bash
pip install keyboard
```

No other dependencies — XInput is accessed directly through the built-in
`ctypes` module and the system `xinput1_4.dll`.

---

## Usage

> ⚠️ **Run as Administrator** — the `keyboard` library captures input at a
> low level and requires administrator rights on Windows.

```bash
python vibrationxcar.py
```

---

## Controls

| Key | Action |
|-----|--------|
| `W` | Forward |
| `S` | Backward (walking pulse algorithm) |
| `A` | Left |
| `D` | Right |
| `W` + `A` | Forward-left (diagonal) |
| `W` + `D` | Forward-right (diagonal) |
| `E` | Turbo — both motors at maximum 65535 (toggle) |
| `1` – `7` | Switch gamepad profile |
| `[` | Calibrate: pulls left — reduce left motor |
| `]` | Calibrate: pulls right — increase left motor |
| `Esc` | Quit (motors are guaranteed to stop) |

---

## Profiles

XInput gamepads have two physically asymmetric motors. The heavy (left)
motor produces more torque than the light (right) motor. Profiles
compensate for this difference so the gamepad drives straight.

| # | Profile | When to use |
|---|---------|-------------|
| 1 | Left heavier — mild (1.4x) | Gamepad slightly pulls left on W |
| 2 | Left heavier — medium (1.7x) | Gamepad noticeably pulls left on W |
| 3 | Left heavier — strong (2.2x) | Gamepad strongly pulls left on W |
| 4 | Right heavier — mild (1.4x) | Gamepad slightly pulls right on W |
| 5 | Right heavier — medium (1.7x) | Gamepad noticeably pulls right on W |
| 6 | Right heavier — strong (2.2x) | Gamepad strongly pulls right on W |
| 7 | Symmetric | Both motors are equal |

### Finding your profile

1. Place the gamepad on a smooth flat surface
2. Hold `W` and switch through profiles `1` → `2` → `3`
3. If it still pulls right at all profiles, try `4` → `5` → `6`
4. Found a close match — fine-tune with `[` and `]`

---

## Technical details

### XInput via ctypes

Instead of third-party libraries, VibrationXcar calls `XInputSetState`
directly from the Windows system DLL. This ensures compatibility with
Windows 8.1 and keeps dependencies minimal.

```python
class XINPUT_VIBRATION(ctypes.Structure):
    _fields_ = [
        ("wLeftMotorSpeed",  WORD),   # 0–65535, heavy motor
        ("wRightMotorSpeed", WORD),   # 0–65535, light motor
    ]
```

### Backward algorithm

A simple on/off pulse does not produce enough force — the motor does not
spin up fully in 120 ms. VibrationXcar uses a **walking pulse algorithm**:

```
Step 1: Left 100% + Right 30%  → 200 ms  (main push)
Pause:                          →  55 ms
Step 2: Right 100% + Left 30%  → 200 ms  (sustain)
Pause:                          →  55 ms
```

Motors alternate dominance, creating an asymmetric force vector.
Duty cycle: **78%** (previously 60%). The motor stays at full speed
long enough to produce a strong consistent push.

### Turbo mode

Turbo (E key) sends `65535 / 65535` directly via `XInputSetState`,
bypassing all profile coefficients. The command is re-sent every 50 ms
to prevent the Windows driver from resetting the vibration state.

### Safety

The `finally` block in `main()` guarantees motors are turned off on
any exit: `Esc`, `Ctrl+C`, or terminal window close.

---

## Project structure

```
VibrationXcar/
├── vibrationxcar.py    # main script
├── requirements.txt    # dependencies
├── LICENSE             # MIT
└── README.md           # this file
```

---

## License

MIT — do whatever you want, attribution appreciated.
