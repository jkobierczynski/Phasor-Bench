# Phasor Bench

A schematic editor and phasor (vector) diagram tool for linear AC circuits, in a single web page.

Draw a network of resistors, inductors and capacitors, connect it to a sinusoidal or other periodic
voltage or current source, and read the voltage and current phasors of every part and every
harmonic. Everything runs in the browser; there is nothing to install and no server.

## Running it

Open `index.html` in a current browser (Firefox, Chrome, Edge or Safari). That is all.

The page loads three typefaces from Google Fonts when it is online and falls back to system fonts
when it is not. Nothing else is fetched, and no data leaves the browser.

## What it does

- **Schematic editor** on a snap grid, with zoom and pan: resistor, inductor, capacitor, general impedance `R + jX`,
  voltage source, current source, wires, named connect-points and a reference (0 V) symbol.
- **Sources** with a selectable waveform: sine, square (block), triangle, sawtooth, pulse train with
  duty cycle, half-wave and full-wave rectified sine.
- **Fourier analysis** of the non-sinusoidal sources, with a selectable highest harmonic (1 to 25).
  Each harmonic is solved at its own frequency and the results are superposed.
- **Phasor diagram** of the voltages and currents you tick, with three views (U + I, U only,
  I only) and two layouts:
  - *From origin*: every phasor starts at the origin.
  - *Tip to tail*: voltages run between the node potentials, so Kirchhoff's voltage law shows as
    closed polygons. In the current view, the currents into and out of a chosen connect-point are
    chained and meet at one point, which is Kirchhoff's current law.
- **One small phasor diagram per harmonic**; click one to show it in the large diagram and in the
  readings.
- **Frequency slider** (logarithmic) that moves the phasors live, with optional curves that trace
  where each phasor tip travels over the slider's range.
- **Waveforms** over two periods as the sum of the harmonics, next to the ideal source shape, with a
  time cursor and rotating phasors.
- **Readings table**: voltage, current, impedance, active and reactive power per part and per
  harmonic, plus total RMS voltage, total RMS current and total power over all harmonics.
- **SPICE netlist export** for PSpice, LTspice and ngspice.
- **Movable and resizable panels** in one to five columns with draggable dividers, and colour themes (Solarized
  Light/Dark, Nord, Catppuccin Latte/Frappé/Macchiato/Mocha, plus the built-in light and dark).

## Using it

### Drawing

Pick a part in the toolbar and drag on the grid from one terminal to the other. A single click
places a part of default length. Wires drawn diagonally become an L-shaped pair.

Parts are connected where their ends share a grid point. A part end or connect-point that lies on a
wire is connected to that wire; two wires that merely cross are not.

With the **Select** tool, click a part to edit it, drag it to move it, or drag one of its end
handles to stretch it.

| Key | Action |
| --- | --- |
| `S` | Select |
| `W` | Wire |
| `R` `L` `C` `Z` | Resistor, inductor, capacitor, impedance |
| `U` `I` | Voltage source, current source |
| `P` | Connect-point |
| `G` | Reference (0 V) |
| `Del` / `Backspace` | Delete the selected part |
| `Esc` | Back to Select |
| `Ctrl+Z` / `Ctrl+Y` | Undo / redo |
| `+` / `-` | Zoom in / out |
| `0` | Zoom back to 100% |
| `F` | Fit the whole circuit in view |

### Zoom and pan

The board is a grid of 59 by 35 points; at 100% you see a quarter of it. Zoom with the buttons in
the Schematic header, with `Ctrl` + mouse wheel (or a trackpad pinch), or with two fingers on a
touch screen. A plain mouse wheel keeps scrolling the page. To pan, drag empty space with the
Select tool or drag with the middle mouse button. **Fit** shows the whole circuit.

### Values

Values accept SI prefixes and a decimal comma: `4,7k`, `100n`, `2.2u`, `15m`, `1M`.
Note that `m` is milli and `M` is mega.

### Conventions

- A sine source is entered as **RMS**; every other waveform is entered as **peak value**.
- Every phasor is the RMS value of one harmonic. Phase angles are referred to `cos(ωt)`, so a sine
  source with phase 0° is `√2·U·cos(ωt)`.
- A passive part counts voltage and current from its first terminal to its second, in the direction
  of the small arrow on its lead (load convention).
- A source shows the voltage, current and power it delivers. The `+` mark is the positive terminal
  of a voltage source; the arrow in a current source is the direction of its current.
- If no reference symbol is placed, the negative terminal of the first voltage source is taken as
  0 V, and the page says so under the schematic.

### Panels

The **Columns** box in the header sets the number of columns, from one to five; below them is a
full-width strip. If you have not moved any panels, changing the number switches to a standard
arrangement for that count. Otherwise your panels stay where they are, and panels from a column
that disappears move to the last remaining column.

Drag a panel by the dotted grip in its top-left corner to move it to any column or to the
full-width strip underneath. Drag the hatched corner at its bottom right to resize it; panels that
are made narrower can sit side by side. Drag a bar between two columns to change their widths.
Double-click a resize corner or a bar to reset it, or use **Reset panels** to restore the standard
arrangement for the current number of columns. The
grips, the corners and the bar also respond to the arrow keys.

### Saving

The circuit, the theme and the panel arrangement are kept in the browser's local storage. To keep a
circuit in a file, use **Copy current circuit** and paste the text somewhere; paste it back and use
**Load from the box** to restore it.

### SPICE export

**Export SPICE netlist** writes the circuit as a netlist with an `.AC` sweep and a `.TRAN` run.
Save the text as a `.cir` file.

| On the page | In the netlist |
| --- | --- |
| Reference node | Node `0` |
| Connect-points | Nodes with the same names |
| Sine source | `AC` value (RMS) plus `SIN(...)` |
| Square, pulse train | `PULSE(...)` |
| Triangle, sawtooth | `PWL(...)` covering the first five periods |
| Rectified sines | Their Fourier series up to the chosen harmonic |
| Impedance `R + jX` | Resistor in series with the L or C that gives the same X at the fundamental |

Part names get the letter SPICE expects, so the voltage source `E1` becomes `VE1` and the current
source `J1` becomes `IJ1`.

## How it works

The circuit is solved with complex modified nodal analysis: one linear system per harmonic, with
node voltages and voltage-source currents as unknowns. Source waveforms are expanded numerically
into Fourier coefficients; each harmonic is solved separately and the results are added, which is
valid because the circuit is linear.

The exported netlists for the built-in examples were checked against ngspice: the `.AC` results and
the Fourier components of the `.TRAN` results match the page's readings in magnitude and phase.

## Limitations

- Linear parts only: no diodes, transistors, transformers, coupled inductors or controlled sources.
- Periodic steady state only. Switch-on transients are not simulated.
- One fundamental frequency for the whole circuit.
- The impedance part `R + jX` keeps the same value at every harmonic.
- The DC part of a pulse or rectified source is solved with inductors as shorts and capacitors as
  open circuits. If the circuit cannot settle it, it is left out and the page says so.
- Currents in wires are not shown, only currents in parts.
- The drawing board is a fixed grid of 59 by 35 points.
- A SPICE netlist can be exported but not imported.

## Files

| File | Contents |
| --- | --- |
| `index.html` | The whole application: markup, styles and script |
| `README.md` | This file |
| `LICENSE` | GNU General Public License, version 3 |

There is no build step and there are no dependencies.

## Licence

Copyright (C) 2026 Jurgen Kobierczynski

Phasor Bench is free software: you can redistribute it and/or modify it under the terms of the GNU
General Public License as published by the Free Software Foundation, version 3.

It is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the
implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the file `LICENSE` for
the full text.

### Third-party material

- Typefaces Barlow, Barlow Semi Condensed and IBM Plex Mono are loaded from Google Fonts and are
  licensed under the SIL Open Font License 1.1. They are not included in this repository.
- The colour values of the Solarized, Nord and Catppuccin themes come from those projects, which
  publish their palettes under the MIT licence.
