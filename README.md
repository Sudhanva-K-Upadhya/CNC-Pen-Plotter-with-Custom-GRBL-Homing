# ✒️ CNC Pen Plotter — GRBL Firmware (Custom Homing)

A DIY CNC pen plotter built on an **Arduino Uno + CNC Shield**, running a fork of the [grbl/grbl](https://github.com/grbl/grbl) firmware with a **custom Y-first homing sequence**.

---

## 📸 Gallery

> _Replace the placeholders below with your own images._

| View | Photo |
|------|-------|
| Full machine (front) | ![Full machine front view](images/machine-front.jpg) |
| Full machine (side) | ![Full machine side view](images/machine-side.jpg) |
| Arduino Uno + CNC Shield | ![Arduino and CNC Shield](images/arduino-cnc-shield.jpg) |
| Pen carriage / Z-axis | ![Pen carriage](images/pen-carriage.jpg) |
| First plot result | ![First plot](images/first-plot.jpg) |

---

## 🧰 Hardware

| Component | Details |
|-----------|---------|
| Microcontroller | Arduino Uno (ATmega328P) |
| Motion controller | CNC Shield v3 |
| Stepper drivers | A4988 / DRV8825 (×3) |
| Steppers | NEMA 17 (X and Y axes) |
| Pen lift | Servo motor or small NEMA 17 (Z axis) |
| Frame | _(describe your frame — e.g. 3D printed, laser cut, aluminium extrusion)_ |
| Power supply | 12 V DC, ≥ 2 A |
| Limit switches | 2× microswitches (X-min, Y-min) |

---

## 💾 Firmware

**Base:** [grbl/grbl](https://github.com/grbl/grbl) — the original open-source CNC firmware for Arduino.

**Modified file:** `grbl/config.h`

### 🔧 Custom Homing Sequence

The default GRBL homing order is X → Y. For this plotter the **Y axis is homed first** to avoid mechanical conflicts during the homing stroke.

The following lines were changed in `grbl/config.h`:

```c
// --- CUSTOM HOMING SEQUENCE ---
// Home Y axis first, then X axis.
// Default GRBL homes X first; this is reversed for pen-plotter geometry.

#define HOMING_CYCLE_0 (1<<Y_AXIS)   // Step 0 — Home Y first
#define HOMING_CYCLE_1 (1<<X_AXIS)   // Step 1 — Then home X

// Z axis (pen lift) is NOT included in the homing cycle.
// #define HOMING_CYCLE_2 (1<<Z_AXIS) // Uncomment if you add a Z limit switch
```

> **Why Y first?**  
> On this plotter the X carriage rides on the Y gantry. Homing Y first parks the gantry safely before the X carriage moves to its home position, preventing the pen from dragging across the work surface.

---

## ⚙️ GRBL Configuration (`$$` settings)

Connect via a serial monitor (115200 baud) and apply these recommended starting values:

```
$0  = 10      ; Step pulse time (µs)
$1  = 25      ; Step idle delay (ms)
$2  = 0       ; Step port invert mask
$3  = 0       ; Direction port invert mask — adjust if axes move backwards
$4  = 0       ; Step enable invert (0 = active LOW for CNC shield)
$5  = 0       ; Limit pins invert (set to 1 if using NC switches)
$6  = 0       ; Probe pin invert
$10 = 1       ; Status report mask
$11 = 0.010   ; Junction deviation (mm)
$12 = 0.002   ; Arc tolerance (mm)
$13 = 0       ; Report in inches (0 = mm)
$20 = 0       ; Soft limits (enable after tuning $130/$131)
$21 = 1       ; Hard limits — enable if limit switches are wired
$22 = 1       ; Homing cycle — ENABLE (required for custom homing)
$23 = 3       ; Homing direction invert (bits: X=bit0, Y=bit1) — home toward min switches
$24 = 50.000  ; Homing feed rate (mm/min)
$25 = 500.000 ; Homing seek rate (mm/min)
$26 = 250     ; Homing debounce delay (ms)
$27 = 1.000   ; Homing pull-off distance (mm)
$100 = 80.000 ; X steps/mm — calibrate for your machine
$101 = 80.000 ; Y steps/mm — calibrate for your machine
$102 = 800.000; Z steps/mm — calibrate for pen-lift servo/stepper
$110 = 3000   ; X max rate (mm/min)
$111 = 3000   ; Y max rate (mm/min)
$112 = 500    ; Z max rate (mm/min)
$120 = 200    ; X acceleration (mm/sec²)
$121 = 200    ; Y acceleration (mm/sec²)
$122 = 50     ; Z acceleration (mm/sec²)
$130 = 200    ; X max travel (mm) — set to your actual bed size
$131 = 200    ; Y max travel (mm) — set to your actual bed size
$132 = 10     ; Z max travel (mm)
```

---

## 🔌 Wiring Overview

```
Arduino Uno
  └── CNC Shield v3
        ├── X driver  → X stepper (left-right axis)
        ├── Y driver  → Y stepper (front-back / gantry axis)
        ├── Z driver  → Z stepper or servo (pen lift)
        ├── X-min pin → limit switch (X home)
        └── Y-min pin → limit switch (Y home)

Power:
  12 V PSU → CNC shield VIN/GND terminals
  USB      → Arduino (for serial/flashing only — do NOT power steppers via USB)
```

> 📷 _Add your wiring diagram photo here:_  
> ![Wiring diagram](images/wiring-diagram.jpg)

---

## 🚀 Getting Started

### 1 — Flash the firmware

```bash
# Clone this repo
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
cd YOUR_REPO

# Open in Arduino IDE
# Board   : Arduino Uno
# Port    : COMx / /dev/ttyUSBx
# Sketch  : grbl/examples/grblUpload/grblUpload.ino
# Upload  ✔
```

### 2 — Connect & configure

Open Arduino IDE Serial Monitor (or any G-code sender) at **115 200 baud**.

```
$$          ; view current settings
$22=1       ; enable homing
$21=1       ; enable hard limits
$H          ; run homing cycle (Y homes first, then X)
```

### 3 — Run a plot

Use any G-code sender that supports GRBL:

- **Universal Gcode Sender (UGS)** — recommended
- Candle
- bCNC

Generate G-code from SVG with **Inkscape + Inkscape-GRBL** or **svg2gcode**.

---

## 📁 Repository Structure

```
.
├── grbl/                  # Modified GRBL source
│   ├── config.h           # ← Custom homing sequence lives here
│   └── ...
├── images/                # Your photos go here
│   ├── machine-front.jpg
│   ├── machine-side.jpg
│   ├── arduino-cnc-shield.jpg
│   ├── pen-carriage.jpg
│   ├── wiring-diagram.jpg
│   └── first-plot.jpg
├── gcode/                 # Example G-code files
│   └── test-square.nc
└── README.md
```

---

## 🐛 Troubleshooting

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| Axes move in wrong direction during homing | Direction invert bits wrong | Flip `$3` bits or physically swap stepper wires |
| Machine doesn't stop at limit switch | Limit switches not wired / `$21=0` | Check wiring; set `$21=1` |
| Homing alarm immediately on `$H` | Limit switch already triggered at startup | Set `$5=1` if using NC switches |
| Steps/mm wrong (circles are oval) | `$100`/`$101` not calibrated | Measure a known distance, calculate: `steps_mm = steps_per_rev × microsteps / mm_per_rev` |
| Pen drags on retract | Z not configured | Tune `$102`, `$112`, `$122` |

---

## 📄 License

The GRBL firmware is licensed under the **GNU GPLv3** — see [grbl/grbl LICENSE](https://github.com/grbl/grbl/blob/master/LICENSE).  
All modifications in this repository are released under the same GPLv3 license.

---

## 🙏 Credits

- [grbl/grbl](https://github.com/grbl/grbl) — Sungeun K. Jeon and the GRBL contributors
- Arduino & CNC Shield community

---

_Built with ✒️ and patience._
