# HM310T Browser Controller

A single-file browser app for the **HANMATEK HM310T** programmable DC bench
supply, over USB using the Web Serial API. No install, no build step, no server —
open the HTML file in Chrome or Edge and connect. A simulation mode lets you try
every feature with no hardware attached.

[![Launch App](https://img.shields.io/badge/Launch%20App-Open%20in%20Browser-4ecdc4?style=for-the-badge&logo=google-chrome&logoColor=white)](https://GWB2025.github.io/hm310t-controller/hm310t_controller.html)

> This is a corrected rewrite. The Modbus register map, the protection decoding,
> and the charger were all reworked against a **physical HM310T**, because the
> reverse-engineered map that circulates online is wrong for this instrument in
> several places that matter.

> **Browser requirement:** Chrome or Edge. The Web Serial API is not implemented
> in Firefox or Safari, so the launch link will load the page there but cannot
> connect to hardware.

## What was wrong before, and what changed

The earlier version followed the widely-copied `pyHM310T` register map and bit
definitions. Measured against real hardware, those are wrong, and the app
behaved accordingly. The corrections, all confirmed on a bench unit:

| Area | The online map / old app | Measured reality (this app) |
|---|---|---|
| Voltage setpoint | 0x0000 | **0x0030** |
| Current setpoint | 0x0002 | **0x0031** |
| Output enable | 0x0006 | **0x0001** |
| Protection status | 0x0020 | **0x0002** |
| OVP / OCP / OPP | 0x0008 / 0x000A / 0x000C | **0x0020 / 0x0021 / 0x0022–23** |
| Protection bit 0x02 | "OCP tripped" | **healthy idle bit — not a fault** |
| Protection bit 0x04 | "OPP tripped" | **OVP tripped** |
| Ratings | assumed 30 V / 10 A / 300 W | **read from 0x0014–0x0017** (this unit: 32 V / 10.2 A / 326.4 W) |
| Block reads | 8 single reads, 50 ms apart (~400 ms/poll) | **block reads, capped at 4 registers** — the instrument refuses more |

Behavioural fixes that follow from the above:

- **The lag is gone.** Live values come back in one or two Modbus transactions
  instead of eight sequential reads separated by fixed delays.
- **Protection is decoded honestly.** The old app lit a permanent red "OCP" lamp
  on a healthy instrument (bit 0x02 is always set) and called an over-voltage
  trip "OPP". Lamps now reflect what the bits mean; the resting bit is shown as
  a raw-status display, not an alarm.
- **A trip is understood to latch.** On this instrument a protection trip clears
  only with a mains power cycle — nothing over the serial link clears it. The
  app says so, and it **refuses to send an output-on command that is certain to
  trip** (OVP at or below the setpoint, or a trip already latched), offering an
  explicit override rather than silently bricking the session.
- **The port can vanish and the app survives it.** A serial-layer failure
  (unplugged adapter, USB sleep) is caught, the connection is dropped cleanly,
  any ramp or charge is abandoned, and the UI stays responsive to reconnect.

## Features

Live readouts (V, I, P, resistance, energy, capacity) with a trend chart; set &
apply with presets and keyboard shortcuts; hardware protection (OVP/OCP/OPP)
read/write; a **battery charger** (CC/CV) with chemistry presets and a pack
preflight; an **I-V curve tracer**; a **voltage ramp**; a **sequence** runner
with a ramp generator; a **data logger** with CSV/JSON export; software **alarm**
thresholds with optional auto-kill; and a **simulation mode** that reproduces the
instrument's measured quirks, including a battery model for exercising the
charger.

## Quick start

1. Open `hm310t_controller.html` in **Chrome or Edge** (Web Serial is required;
   Firefox and Safari do not implement it). A secure context is needed, so serve
   over `https://` or `localhost`, or just open the local file.
2. Plug the HM310T into the PC (its rear USB port; it enumerates as a CH340
   serial device on most units).
3. Click **Connect USB** and pick the port. Or click **Simulation: OFF** to flip
   it on and try everything with no hardware.

Serial defaults: 9600 baud, 8N1, slave address 1.

## Charging — read this first

The charger runs a constant-current / constant-voltage cycle: hold the charge
current until the pack reaches the target voltage, then hold the target voltage
while the current tapers, stopping at the termination current (C/x) or a timeout.

The reason it is safe enough to offer: **the supply's CV ceiling is enforced in
hardware.** It physically cannot drive the pack above the target voltage,
whatever the software does — a crash, a dropped port, or a stalled loop leaves
the output at CV float, not a runaway. Software owns only *termination* and
*monitoring*, and a failure there leaves the pack at its correct voltage.

Even so, this is a **supervised bench aid, not a proper charger.** It has no
independent hardware protection of its own beyond the supply's OVP. Before any
real current flows it checks the configuration against the instrument's true
range, then reads the pack's voltage under a gentle probe current and refuses to
proceed if the pack is already at/above target (wrong cell count or already
full), or below a per-chemistry floor (reversed, dead, or nothing connected).
**NiMH and NiCd are approximate** — their correct termination is −ΔV or
temperature, which a source-only supply cannot see, so those lean on the timeout,
and the app says so. Do not leave lithium charging unattended.

## Keyboard shortcuts

`Space` output on/off · `↑`/`↓` nudge voltage 0.1 V (`Shift` = 0.01 V) ·
`Ctrl/⌘+Enter` apply · `Ctrl/⌘+S` logger · `Ctrl/⌘+P` sequence ·
`Ctrl/⌘+K` or `Esc` kill output.

## Note on the register map

The map used here was measured on a real HM310T; see the table above. The
instrument's own maximum ratings live in 0x0014–0x0017 and are read on connect,
so the app clamps to the true ratings rather than an assumed 30 V / 10 A. If your
unit reports a different model code or ratings, the app adapts to what it reads.

## Launch from GitHub Pages

The app is a single static file, so it can be served straight from **GitHub
Pages** — anyone can then run it from a URL with no download. This repository
includes a GitHub Actions workflow (`.github/workflows/static.yml`) that deploys
the site on every push to `main`.

### Enable it

1. Push this repository to GitHub.
2. Go to **Settings → Pages**.
3. Under **Build and deployment**, choose **Source: GitHub Actions**.
4. The workflow runs on the next push and publishes the site.

Once deployed, open the app at the Pages URL for your repository, for example:

```
https://GWB2025.github.io/hm310t-controller/hm310t_controller.html
```

(The URL is `https://<user>.github.io/<repo>/hm310t_controller.html` — substitute
your own username and repository name. The **Launch App** badge at the top of
this README points at that address.)

Because Web Serial needs a secure context, this works only over HTTPS — which
GitHub Pages provides by default — and only in Chrome or Edge.

## License

MIT.
