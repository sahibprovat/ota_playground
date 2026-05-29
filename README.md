# Two-Stage CMOS Op-Amp Design Studio

An interactive, single-file web tool for designing **and understanding** a two-stage,
Miller-compensated CMOS operational amplifier (a classic NMOS-input-pair OTA).

Drag any slider and the governing equation appears live, with *your* numbers
substituted in — so you can see exactly *why* gain, phase margin, ICMR, slew rate,
and output swing move the way they do. No build step, no dependencies, no server.

> Replace the placeholders marked `<like-this>` below (your username, repo name,
> demo GIF, and author) before publishing.

![No dependencies](https://img.shields.io/badge/dependencies-none-brightgreen)
![Single file](https://img.shields.io/badge/build-none%20·%20single%20HTML-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-black)

---

## 🔗 Live demo

**[Open the tool »](https://<your-username>.github.io/<repo-name>/)**

<!-- Record a short (10–20 s) screen capture of dragging a slider while the
     equation panel reacts, save it as assets/demo.gif, and it will show here. -->
![Design studio demo](assets/demo.gif)

---

## What it is

A hand-analysis playground for the textbook two-stage op-amp. It is meant for
learning and first-cut sizing — the kind of "what happens to phase margin if I
widen M6?" question you would otherwise answer with a notebook and a calculator.
Everything updates in real time as you move the sliders.

It is **not** a SPICE replacement. It uses square-law (Level-1) hand equations to
build intuition fast; always confirm a real design in SPICE / Cadence.

## Features

- **Live equation panel** — drag a slider (or click its name) and the relevant
  formula pops up with your current values plugged in, plus a one-line note on the
  trade-off. Click any metric card to see how that number is derived.
- **Seven live metrics** — DC gain, phase margin, GBW, slew rate, ICMR⁻, ICMR⁺ and
  output swing, each colour-coded by health (e.g. PM turns amber < 52°, red < 45°).
- **Schematic + region inspector** — the full topology with **M1–M7** labelled,
  live node voltages and branch currents, and every transistor coloured
  **saturation / triode / off**. Separate `Vcm`, `Vd`, `Vout` sliders sweep the
  operating point, with a per-device "why / how to fix" hint.
- **Bode plot** — gain and phase with the dominant pole, output pole `p₂`, Miller
  zero `z₁`, GBW marker and a phase-margin bracket.
- **Headroom view** — input common-mode range and output-swing bars, the
  body-effect note explaining the real-silicon ICMR⁻ wall, and a live table of the
  `W/L` you'd need to hit each ICMR⁻ target at the current bias.
- **Sensitivity matrix** — click any cell to read the equation and mechanism behind
  how each parameter moves each metric.
- **Light / dark theme** (follows your OS), and a **Reset** button.

## The circuit

A textbook two-stage Miller OTA:

| Device   | Type | Role                                            |
| -------- | ---- | ----------------------------------------------- |
| M1, M2   | NMOS | Input differential pair                         |
| M3, M4   | PMOS | Active current-mirror load                      |
| M5       | NMOS | Tail current source                             |
| M6       | PMOS | Second-stage common-source driver               |
| M7       | NMOS | Second-stage load (current source)              |
| C_c      |  —   | Miller compensation capacitor (pole splitting)  |
| C_L      |  —   | External load capacitor                         |

## The model

Default process is roughly a 0.35 µm-class CMOS node at `L = 1 µm`:

```
µCoxN = 170 µA/V²     µCoxP = 58 µA/V²
λN    = 0.014 V⁻¹     λP    = 0.042 V⁻¹    (scaled for L = 1 µm)
VTN   = 0.54 V        |VTP| = 0.71 V
VDD   = 3.3 V         L     = 1 µm

Default bias:  I5 = 20 µA  (I_D1 = I_D2 = 10 µA),  I_D6 = 100 µA
               Cc = 1.4 pF,  CL = 2 pF
```

Core hand equations the engine evaluates:

```
Transconductance   gm  = √(2 · µCox · (W/L) · I_D)
Output resistance  Ro1 = 1 / ((λN + λP) · I_D1)        (stage 1)
                   Ro2 = 1 / ((λN + λP) · I_D6)        (stage 2)
DC gain            Av  = (gm2 · Ro1) · (gm6 · Ro2)

Gain–bandwidth     GBW = gm2 / (2π · Cc)
Output pole        p2  = gm6 / (2π · CL)
Miller (RHP) zero  z1  = gm6 / (2π · Cc)
Phase margin       PM  ≈ 90° − atan(GBW/p2) − atan(GBW/z1)

Slew rate          SR  = I5 / Cc          (input-pair limited)
Output slew        SRo = I_D6 / CL

Overdrive          Vov = √(2 · I_D / (µCox · (W/L)))
ICMR⁻              = VTN + Vov12 + Vov5         (hard floor = VTN)
ICMR⁺              = VDD − |VTP| − Vov34 + VTN
Vout,min           = Vov7
Vout,max           = VDD − Vov6
```

The Sensitivity tab and the live panel both derive every effect from these.

## Running it

It is a single static file — there is nothing to install.

- **Locally:** download or clone, then open `index.html` in any modern browser.
- **Online (GitHub Pages):** push the repo, then go to
  **Settings → Pages → Deploy from a branch → `main` / root**. Your tool goes live
  at `https://<your-username>.github.io/<repo-name>/` within a minute or two.

## Project structure

```
.
├── index.html      # the entire application (HTML + CSS + vanilla JS, ~900 lines)
├── README.md
└── assets/
    └── demo.gif    # optional screen recording shown above
```

The JavaScript is organised as a small set of pure functions:

- `computeSS()` — small-signal metrics at the balanced bias point (gain, PM, GBW, …)
- `computeOP()` — large-signal operating-point solver for the schematic/regions
- `SLIDER_STORY` / `METRIC_STORY` — the live-equation text for each control
- `drawBode()`, `drawHR()`, `drawSchematic()` — the three `<canvas>` views
- `buildSens()` — the sensitivity matrix

If you want to change the physics, start at the constants block and `computeSS()`.

## Known limitations & modeling caveats

These are deliberate simplifications — and good entry points for contributions:

- **Square-law (Level-1) model only.** Great for intuition, not for sign-off.
  No short-channel effects, no `gm/ID` methodology, no BSIM. Validate in SPICE.
- **Metric cards assume the balanced quiescent point** (`I_D1 = I_D2 = I5/2`,
  `I_D6` fixed). The schematic tab runs an independent large-signal solver, so its
  device currents can differ when you push `Vd` hard — that's expected.
- **The Miller RHP zero is modelled as added phase lag**, with no compensation
  resistor. A real design adds a series nulling resistor `Rz ≈ 1/gm6` to push or
  cancel the zero — not yet implemented.
- **The mirror-pole marker (`p_mirror`) currently sits far off-screen** due to a
  capacitance-scaling quirk carried over from the original widgets, so it barely
  affects PM. Fixing it to a physical value (~hundreds of MHz) is a great first PR.
- **`λ` is treated as constant** (not length-dependent), and **body effect on
  M1/M2 is shown as an explanatory note** in the Headroom tab rather than folded
  into a live `V_T` shift.

## Roadmap / good first contributions

- [ ] Give `p_mirror` a physically meaningful value and show it on the Bode plot.
- [ ] Add a series nulling resistor `Rz` and let it cancel/relocate the RHP zero.
- [ ] Design **presets** (e.g. "high gain", "fast slew", "wide ICMR") + share-by-URL.
- [ ] Export the Bode/headroom canvas as PNG and the parameters as CSV/JSON.
- [ ] Add CMRR, PSRR, and an input-referred noise estimate.
- [ ] Process-corner / temperature toggles.
- [ ] Tighter mobile layout for the slider + equation panel.
- [ ] A small unit-test harness for `computeSS()` against known textbook values.

Found a bug or have an idea? Please [open an issue](https://github.com/<your-username>/<repo-name>/issues).

## Contributing

Contributions are very welcome — especially from the analog / mixed-signal crowd.

1. Fork the repo and create a branch: `git checkout -b my-feature`.
2. Edit `index.html`. **Keep it dependency-free and single-file** — no build tools,
   no frameworks, vanilla JS only. That's the whole point of this project.
3. Test by opening `index.html` in a browser; check both light and dark themes.
4. If you change the physics, note the equation and source (textbook / paper) in
   your PR description so reviewers can check it.
5. Open a pull request describing what changed and why.

For larger changes (a new tab, a new model), open an issue first so we can agree on
the approach.

## License

Released under the **MIT License** — see [`LICENSE`](LICENSE). Add a `LICENSE` file
with the standard MIT text, `Copyright (c) <year> <your name>`.

## Disclaimer

This is an **educational** tool built on first-order hand equations. The numbers it
produces are for learning and initial sizing only and will differ from a real
foundry process. Do not use them for tape-out without full SPICE / Cadence
verification.

---

*Built as a study aid for two-stage CMOS op-amp design. PRs and issues welcome.*
