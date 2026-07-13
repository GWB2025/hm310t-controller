# HM310T Browser Controller

A fully-featured, single-file browser application for controlling the **HANMATEK HM310T** programmable DC power supply (0–30 V, 0–10 A, 300 W) over USB/Modbus RTU. No installation, no build step, no server — just open the HTML file in Chrome or Edge and connect.

[![Launch App](https://img.shields.io/badge/Launch%20App-Open%20in%20Browser-4ecdc4?style=for-the-badge&logo=google-chrome&logoColor=white)](https://gwb2025.github.io/HANMATEK-Programmable-DC-Power-Supply-HM310T)


![Screenshot](screenshot.png)

## Features

| Feature | Description |
|---|---|
| **USB Connection** | Web Serial API (Chrome/Edge) talks directly to the FTDI USB-to-serial bridge on the HM310T rear panel |
| **Modbus RTU** | Full CRC-16, frame parsing, and register mapping in JavaScript — zero dependencies |
| **Live Readouts** | Real-time voltage, current, power, resistance, energy (Wh), and capacity (Ah) |
| **Set & Apply** | Enter target voltage and current limit, click Apply, or use keyboard shortcuts |
| **Output Toggle** | One-click ON/OFF with visual feedback and safety state |
| **Quick Presets** | 8 hard-coded presets (3.3 V, 5 V, 9 V, 12 V, 15 V, 24 V, etc.) plus custom saved presets |
| **Sequence Programmer** | Build multi-step voltage/current ramps with dwell times, loops, and inline editing |
| **Ramp Generator** | Auto-generate linear ramps from Start V to End V with configurable step size and dwell |
| **Battery Charger (CC/CV)** | Automated charging profiles for Li-ion, LiFePO₄, NiMH, NiCd, Lead Acid, or custom chemistry |
| **I-V Curve Tracer** | Sweep voltage and plot current to characterize diodes, LEDs, solar cells, transistors |
| **Data Logger** | Time-stamped logging of V, I, P, output state, sequence step, and charge phase |
| **Trend Chart** | Real-time canvas chart with auto-scaling Y-axes |
| **Protection Settings** | Read/write OVP, OCP, and OPP thresholds directly from the GUI |
| **Alarm Thresholds** | Min/max bounds for V and I; flashing red alert with optional auto-kill |
| **Simulation Mode** | Test sequences, chargers, and loggers without hardware — realistic slew and noise |
| **Keyboard Shortcuts** | Space (toggle output), ↑↓ (nudge voltage), Ctrl+Enter (apply), Ctrl+S (logger), Ctrl+P (sequence), Ctrl+K (kill) |
| **Persistent Settings** | All state saved to `localStorage` — restores on reopen |
| **Import / Export** | Sequences as JSON or CSV; logs as CSV or JSON |

## Quick Start

### 1. Download

Download `hm310t_controller.html` from the [releases page](../../releases) or clone this repository.

```bash
git clone https://github.com/GWB2025/HANMATEK-Programmable-DC-Power-Supply-HM310T
cd hm310t-controller
```

### 2. Open in Browser

Double-click `hm310t_controller.html` or serve it locally:

```bash
# Any static server works
python -m http.server 8080
# Then open http://localhost:8080/hm310t_controller.html
```

> **Browser requirement:** Chrome or Edge (Web Serial API). Firefox and Safari are not supported.

### 3. Connect Hardware

1. Plug the HM310T into your PC via the **USB port on the rear panel**.
2. In Windows Device Manager, confirm the FTDI COM port appears (e.g., COM3).
3. Click **Connect USB** → select the port from the browser picker.
4. The supply communicates at **9600 baud, 8 data bits, 1 stop bit, no parity** (default).

### 4. Test in Simulation Mode

No hardware yet? Click **Simulation: OFF** in the header to enable the virtual supply. You can test sequences, the charger, the logger, and the I-V tracer with realistic behavior before wiring anything up.

## Usage Guide

### Basic Control

1. Enter a **Voltage** (0–30 V) and **Current Limit** (0–10 A).
2. Click **Apply** or press `Ctrl+Enter`.
3. Toggle **Output ON/OFF** with the switch or press `Space`.
4. Nudge voltage with `↑` / `↓` (0.1 V steps; `Shift` for 0.01 V).

### Sequence Programming

Build automated test profiles:

1. Use the **Ramp Generator** to auto-populate steps, or click **+ Add Step** to build manually.
2. Edit voltage, current, and dwell time inline in the table.
3. Click **Play** (`Ctrl+P`) to run. The active step highlights and auto-scrolls.
4. Export the sequence as JSON or CSV for reuse.

**Example CSV format for import:**
```csv
voltage,current,dwell
3.3,0.5,2.0
5.0,1.0,5.0
9.0,2.0,3.0
12.0,3.0,10.0
```

### Battery Charging

1. Select **Chemistry** (Li-ion, LiFePO₄, NiMH, etc.) or choose **Custom**.
2. Set **Cells** count, **Charge Current**, and **Termination Current**.
3. Set a **Timeout** in minutes as a safety backstop.
4. Click **Start Charge**.

The controller runs a two-phase state machine:
- **CC phase** — holds charge current while voltage rises to the target.
- **CV phase** — holds target voltage while current tapers.
- **Done** — terminates when current falls below the threshold or timeout expires.

Capacity in mAh and elapsed time are tracked live. The logger records the charge phase.

### I-V Curve Tracer

Characterize a component:

1. Connect the device under test to the supply terminals.
2. Set sweep range (**Start V** → **End V**), **Step**, **Current Limit**, and **Dwell**.
3. Click **Run Sweep**.
4. The app plots the I-V curve and computes:
   - **Vf @ 1 mA** — forward turn-on voltage
   - **Knee V** — voltage at maximum dI/dV
   - **R dyn** — dynamic resistance at half maximum current

### Data Logging

1. Set **Log Interval** (e.g., 1000 ms) and **Max Points**.
2. Click **Start** (`Ctrl+S`) to begin logging.
3. Every sample is timestamped and shown in the live table.
4. Click **Export CSV** or **Export JSON** to download the full dataset.

Logged columns: `timestamp, voltage_V, current_A, power_W, output_state, sequence_step, charge_state`

### Protection Settings

Set hardware protection thresholds before powering sensitive circuits:

1. Enter **OVP** (over-voltage), **OCP** (over-current), and **OPP** (over-power) values.
2. Click **Apply** to write to the supply.
3. Click **Read** to verify current thresholds.

> **Safety tip:** Before powering a 3.3 V MCU, set OVP to 4 V to prevent accidental over-voltage.

### Alarm Thresholds

Set software bounds for live monitoring:

1. Enable **Alarm Thresholds** and set V min/max and A min/max.
2. If readings drift outside bounds, the panel flashes red and an **ALARM** pill appears.
3. Enable **Kill output on alarm** to automatically disable the output.

### Simulation Mode

Toggle **Simulation: ON** in the header to run a virtual HM310T:

- Realistic voltage slew (2 V/s), current limiting, and load behavior
- Configurable **Sim Load Resistance** in Settings (default 10 Ω)
- Small random noise added for realism
- All features (sequences, charger, logger, I-V tracer) work identically

## Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Space` | Toggle output ON / OFF |
| `↑` / `↓` | Nudge voltage ±0.1 V |
| `Shift + ↑` / `Shift + ↓` | Nudge voltage ±0.01 V |
| `Ctrl + Enter` | Apply voltage / current setpoints |
| `Ctrl + S` | Start / stop data logger |
| `Ctrl + P` | Play / pause sequence |
| `Ctrl + K` | Kill output immediately |

> Shortcuts are active only when not typing in an input field.

## Modbus Register Map

The app uses the reverse-engineered register map from the `pyHM310T` project:

| Register | Address | Type | Description |
|---|---|---|---|
| Voltage Setpoint | `0x0000` | Holding (2×16-bit) | 32-bit value, 2 decimal places |
| Current Setpoint | `0x0002` | Holding (2×16-bit) | 32-bit value, 3 decimal places |
| Output Enable | `0x0006` | Holding | `0x0001` = ON, `0x0000` = OFF |
| OVP Threshold | `0x0008` | Holding (2×16-bit) | Over-voltage protection |
| OCP Threshold | `0x000A` | Holding (2×16-bit) | Over-current protection |
| OPP Threshold | `0x000C` | Holding (2×16-bit) | Over-power protection |
| Voltage Display | `0x0010` | Input (2×16-bit) | Live measured voltage |
| Current Display | `0x0012` | Input (2×16-bit) | Live measured current |
| Power Display | `0x0014` | Input (2×16-bit) | Calculated power |
| Protection Status | `0x0020` | Input | Bitfield: OVP(1), OCP(2), OPP(4), OTP(8), SCP(16), FAN(32) |

**Serial parameters:** 9600 baud, 8 data bits, 1 stop bit, no parity. Slave address = 1.

## File Structure

```
HANMATEK-Programmable-DC-Power-Supply-HM310T/
├── hm310t_controller.html    # The complete app (single file, ~50 KB)
├── README.md                 # This file
├── LICENSE                   # MIT License
├── .gitignore                # Git ignore rules
├── examples/
│   ├── liion_charge.csv      # Example charge log
│   ├── led_iv_curve.json     # Example I-V sweep data
│   └── ramp_sequence.csv     # Example sequence for import
└── screenshot.png            # App screenshot (add your own)
```

## GitHub Pages Deployment

This repository includes a GitHub Actions workflow that automatically deploys the app to **GitHub Pages** on every push to the `main` branch. This means anyone can use the controller directly from a browser URL — no download required.

### Enable GitHub Pages

1. Push this repository to GitHub.
2. Go to **Settings → Pages** in your repository.
3. Under **Build and deployment**, select **Source: GitHub Actions**.
4. The workflow `.github/workflows/static.yml` will run automatically on the next push.
5. Your app will be live at:
   ```
   https://github.com/GWB2025/HANMATEK-Programmable-DC-Power-Supply-HM310T
   ```

### Using the live URL

- Open the URL in **Chrome or Edge**.
- Click **Connect USB** and select your HM310T port — the app works exactly like the local file.
- Bookmark the page for quick access.

> **Note:** Because the app uses the Web Serial API, it still requires a secure context (HTTPS) and a supported browser. GitHub Pages provides HTTPS by default, so this works out of the box.


## Browser Compatibility

| Browser | Support | Notes |
|---|---|---|
| Chrome | ✅ Full | Web Serial API native |
| Edge | ✅ Full | Web Serial API native |
| Firefox | ❌ No | Web Serial API not implemented |
| Safari | ❌ No | Web Serial API not implemented |
| Opera | ⚠️ Partial | May work if Chromium-based |

## Troubleshooting

### "Web Serial API not supported"
Use Chrome or Edge. The app cannot function in Firefox or Safari.

### Port not appearing in picker
1. Ensure the USB cable is connected to the **rear panel** USB port (not the front).
2. Install the FTDI VCP driver if Windows does not auto-detect: [FTDI Drivers](https://ftdichip.com/drivers/)
3. Check Device Manager → Ports (COM & LPT) for the FTDI device.
4. Try a different USB cable (some cables are charge-only, no data lines).

### Modbus timeout errors
1. Verify the supply is powered on and the USB cable is seated firmly.
2. Check that no other software (e.g., manufacturer's PC software) has the COM port open.
3. Increase the **Inter-step Delay** in Settings if running long sequences.
4. Try disconnecting and reconnecting the USB port.

### Simulation mode works but hardware does not
This confirms the app logic is correct. The issue is likely:
- Wrong COM port selected
- FTDI driver not installed
- USB cable without data lines
- Supply not powered on

## Safety Warning

This is a **300 W programmable power supply**. Always:

1. Start with **Output OFF**.
2. Verify voltage and current setpoints on the readout cards.
3. Set appropriate **OVP / OCP / OPP** thresholds before enabling output.
4. Use the **Kill Output** button (`Ctrl+K`) for emergency shutdown.
5. Never leave a charging battery unattended.

The authors are not responsible for damage to equipment, batteries, or property resulting from use of this software.

## Contributing

Contributions are welcome. Areas of interest:

- Additional battery chemistries and charge profiles
- Export formats (MATLAB, Python pandas, etc.)
- Dark/light theme toggle
- Mobile-responsive layout improvements
- Additional I-V curve analysis (series resistance, ideality factor)

1. Fork the repository
2. Create a feature branch (`git checkout -b feature-name`)
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

## License

MIT License — see [LICENSE](LICENSE) for details.

## Acknowledgments

- Reverse-engineered Modbus register map based on [joeyda3rd/pyHM310T](https://github.com/joeyda3rd/hanmatek-power-supply)
- Web Serial API implementation by the W3C Web Platform Incubator Community Group
