# GITT Diffusion Coefficient Analyzer

A single-file, browser-only tool for processing raw GITT (Galvanostatic Intermittent Titration Technique) battery test data. Upload a LAND-exported spreadsheet, and it automatically finds the four key voltage points in every pulse, lets you review and exclude bad cycles, and plots the lithium-ion diffusion coefficient against capacity.

(https://raybenoid.github.io/gitt-diffusion-analyzer/)

No installation, no build step, no backend. It's one HTML file that runs entirely in your browser — your battery data never leaves your computer.

## Why this exists

GITT is a common technique for measuring how fast lithium ions diffuse through an electrode material. The analysis requires pulling four specific voltage values out of *every single pulse* in a test that can have hundreds of pulses — the voltage right before a current pulse starts, the voltage right after (which includes the instantaneous IR drop), the voltage at the end of the constant-current step, and the new resting voltage once the cell relaxes again. Doing that by hand in a spreadsheet, pulse after pulse, is slow and error-prone. This tool automates the whole pipeline, from raw cycler export to a ready-to-use diffusion coefficient curve.

## Features

- **Drag-and-drop Excel upload** — reads the wide-format `.xlsx` files exported by LAND testers, where each sample occupies three columns (`TestTime`, `Current`, `Voltage`). A single file can contain multiple samples; switch between them with the tabs at the top.
- **Automatic pulse detection** — segments the test into rest / charge / discharge steps from the current trace, pairs each pulse with its surrounding rest periods, and flags long formation/activation steps separately so they don't pollute the analysis.
- **Four-point voltage extraction** — for every pulse, finds the steady-state voltage before the pulse, the IR-drop voltage the instant current is applied, the voltage at the end of the constant-current step, and the new steady-state voltage after the following rest.
- **Diffusion coefficient calculation** — fits the quasi-linear V – √t region of each pulse, computes ΔE<sub>s</sub> and ΔE<sub>τ</sub>, and applies the Weppner–Huggins equation. Enter your material's molar volume, moles of active material, and electrode area, and the tool converts the raw "D_proxy" into a real diffusion coefficient in cm²/s.
- **Pulse oscilloscope view** — a CRT-style panel that zooms into one pulse at a time and marks the four voltage points directly on the curve, so you can sanity-check the automatic detection pulse by pulse.
- **Reliability flags** — pulses where the voltage signal is too small (below the instrument's resolution) or the V–√t fit is poor are automatically flagged and excluded from the diffusion coefficient curve.
- **Selective CSV export** — every pulse has a checkbox; uncheck the cycles whose curves look wrong and they're left out of the exported CSV. "Select all / none / reliable only" buttons make this fast.

## How to use it

1. Open the tool (locally or via the live link).
2. Drop in your `.xlsx` GITT export.
3. If the file has more than one sample, pick the one you want from the tabs.
4. Fill in n<sub>m</sub> (moles of active material), V<sub>M</sub> (molar volume), and S (electrode/electrolyte contact area) to see the real diffusion coefficient — or leave them blank to inspect D_proxy first.
5. Step through pulses with the oscilloscope view to confirm the four points look right.
6. Uncheck any pulses you don't trust, then export the CSV.

## Expected data format

Each sample takes three consecutive columns:

| TestTime (h) | Current (mA) | Voltage (V) |
|---|---|---|
| 0.00 | 0.000 | 3.412 |
| 0.01 | 0.527 | 3.445 |
| ... | ... | ... |

Row 1 holds the sample name (only needs to appear in the first of the three columns), row 2 holds the column headers, and data starts on row 3. Multiple samples are simply placed side by side, three columns each, in the same sheet.

## The method

For each constant-current pulse, the Weppner–Huggins equation gives:

```
D = (4 / (π·τ)) · (n_m·V_M / S)² · (ΔE_s / ΔE_τ)²
```

- **τ** — pulse duration
- **ΔE_s** — the true open-circuit voltage change caused by the pulse (steady state after rest minus steady state before the pulse)
- **ΔE_τ** — the IR-drop-free voltage change during the pulse, taken from the slope of the linear V vs. √t fit
- **n_m, V_M, S** — moles of active material, its molar volume, and the electrode/electrolyte contact area

## Tech notes

This is plain React (loaded from a CDN as a UMD build), transpiled in the browser with Babel Standalone, with [SheetJS](https://sheetjs.com/) for reading `.xlsx` files. There's no bundler, no `node_modules`, no build step — the charts and icons are hand-rolled SVG so the whole thing is a single portable HTML file. That also means it needs an internet connection the first time it loads those three CDN scripts; after that, all parsing and calculation happen locally in your browser.

## License

MIT — see [LICENSE](LICENSE).
