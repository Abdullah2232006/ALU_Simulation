# ALU_Simulation
```markdown
# 2-Bit ALU — Breadboard Simulation

An interactive web-based simulation of a 2-bit Arithmetic Logic Unit built from discrete 7400-series TTL logic gates. Designed as a companion tool for breadboard builders and report writers working with the Proteus 8 reference design.

**Live demo:** Open `index.html` in any modern browser — no server or build step required.

---

## Overview

This project implements a 2-bit ALU that accepts two 2-bit operands (`A`, `B`) and a carry input (`Cin`), then performs one of six operations selected by a 3-bit control word (`S2`, `S1`, `S0`).

The ALU is partitioned into three stages:

| Stage | Function |
|---|---|
| **Logic Unit (LU)** | AND, OR, XOR, NOT — active when `S2 = 0` |
| **Arithmetic Unit (AU)** | Addition & subtraction via 2's complement — active when `S2 = 1` |
| **Output Multiplexer** | 2:1 MUX selects between LU and AU result based on `S2` |

---

## Features

- **Interactive DIP switches** — toggle all 8 input lines (A1, A0, B1, B0, S2, S1, S0, Cin) with realistic lever animation
- **Real-time block diagram** — canvas-rendered schematic with animated signal wires that glow green when HIGH and stay dim when LOW
- **Active path highlighting** — the selected unit (LU or AU) glows and an animated dashed trace follows the active data path through the MUX
- **Output LEDs** — F1, F0 (red) and Cout (orange) with glow effects
- **Computation readout** — human-readable math display (e.g. `3 - 2 = 1`) with signed 2's complement handling
- **Internal signals panel** — shows B', intermediate carry C0, LU result, and AU result at a glance
- **Operation reference table** — all 8 select-line combinations with the active row highlighted
- **Keyboard accessible** — all switches respond to Enter and Space keys
- **Zero dependencies** — single HTML file, runs offline

---

## Operation Table

| S2 | S1 | S0 | Cin = 0 | Cin = 1 | Unit |
|:--:|:--:|:--:|---------|---------|------|
| 0 | 0 | 0 | A AND B | A AND B | Logic |
| 0 | 0 | 1 | A OR B | A OR B | Logic |
| 0 | 1 | 0 | A XOR B | A XOR B | Logic |
| 0 | 1 | 1 | NOT A | NOT A | Logic |
| 1 | 0 | 0 | A + B | A + B + 1 | Arith |
| 1 | 0 | 1 | A + B' | **A − B** | Arith |
| 1 | 1 | 0 | A + B | A + B + 1 | Arith |
| 1 | 1 | 1 | A + B' | **A − B** | Arith |

> **Subtraction** uses 2's complement: `S2=1, S1=0, S0=1, Cin=1` computes `A + B' + 1 = A − B`.

---

## How to Use

1. **Open `index.html`** in Chrome, Firefox, Edge, or Safari.
2. **Set Operand A** — toggle A1 and A0 switches to input the first 2-bit number.
3. **Set Operand B** — toggle B1 and B0 switches for the second operand.
4. **Select Operation** — set S2, S1, S0 to choose the operation (reference the table above).
5. **Set Cin** — required for subtraction (`Cin = 1`) and optional increment (`Cin = 1` with add).
6. **Read outputs** — the F1/F0 LEDs and decimal readout show the result; Cout indicates arithmetic overflow.

### Example: Compute 3 − 1

| Input | Value |
|---|---|
| A1=1, A0=1 | A = 3 |
| B1=0, B0=1 | B = 1 |
| S2=1, S1=0, S0=1 | Select subtraction |
| Cin=1 | Enable 2's complement |
| **Result** | **F = 10₂ = 2**, Cout = 0 |

---

## ALU Architecture

### Logic Unit

The LU uses `S1` and `S0` to select one of four bitwise operations:

```
S1 S0 → Operation
 0  0 → AND  (7408)
 0  1 → OR   (7432)
 1  0 → XOR  (7486)
 1  1 → NOT A (7404, B ignored)
```

Each gate operates on both bits independently: bit 0 computes `F0`, bit 1 computes `F1`.

### Arithmetic Unit

The AU contains two 1-bit full adders in ripple-carry configuration:

```
Bit 0:  A0 ⊕ B0' ⊕ Cin → Sum0,  Carry0
Bit 1:  A1 ⊕ B1' ⊕ Carry0 → Sum1,  Cout
```

Where `B'` is the bitwise inversion of B controlled by `S0`:

- `S0 = 0` → B passed through → addition
- `S0 = 1` → B inverted → with `Cin = 1`, performs 2's complement subtraction

Full adder gate equations:

```
Sum  = A ⊕ B ⊕ Cin
Cout = AB + Cin(A ⊕ B)
```

### Output Multiplexer

A 2:1 MUX steers the final output:

```
S2 = 0 → F = LU result,  Cout = 0
S2 = 1 → F = AU result,  Cout = carry out
```

---

## IC Reference

| IC | Function | Gates | VCC | GND |
|----|----------|-------|-----|-----|
| 7408 | Quad 2-input AND | 4 × AND | Pin 14 | Pin 7 |
| 7432 | Quad 2-input OR | 4 × OR | Pin 14 | Pin 7 |
| 7486 | Quad 2-input XOR | 4 × XOR | Pin 14 | Pin 7 |
| 7404 | Hex Inverter | 6 × NOT | Pin 14 | Pin 7 |

> **Important:** VCC and GND connections are implicit in Proteus but **must be wired on breadboard**. Pin 14 → VCC, Pin 7 → GND for every IC.

---

## Breadboard Notes

### Pull-Down Resistors

Every DIP switch output requires a **10 kΩ pull-down resistor to GND**:

```
CORRECT:  VCC → Switch → Signal wire → 10kΩ → GND
WRONG:    VCC → 10kΩ → Switch → Signal wire  (this is a pull-up)
```

Without pull-downs, disconnected switch outputs float to undefined voltages that TTL often reads as HIGH.

### Bypass Capacitors

- **100 nF ceramic** capacitor between VCC and GND at every IC — place as close to the IC as possible
- **10 µF electrolytic** capacitor across the main power rail

### Common Pitfalls

| Issue | Cause | Fix |
|---|---|---|
| Floating inputs read HIGH | Disconnected gate pin | Tie to GND (AND/NAND) or VCC (OR/NOR) |
| Switch always reads HIGH | Pull-down missing or wired as pull-up | Verify resistor goes from signal to GND |
| IC gets hot / dies | Inserted backwards | Notch/dot marks pin 1 — verify before powering |
| Erratic outputs | Missing bypass caps | Add 100 nF at each IC |
| Momentary glitches on input change | Ripple carry delay | Expected behavior, not a fault |

### Unused Gates

Tie unused gate inputs to a fixed level:
- **AND / NAND** inputs → GND (output stays HIGH for NAND)
- **OR / NOR** inputs → GND (output stays LOW for OR)
- **XOR** inputs → GND (output follows the other input)
- **NOT** inputs → can be left unconnected (no effect) but tying to GND is safer

---

## Input / Output Pin Summary

### Inputs (DIP switches with 10 kΩ pull-downs)

| Signal | Switch | Description |
|--------|--------|-------------|
| A1 | DIPSW top #1 | Operand A, bit 1 (MSB) |
| A0 | DIPSW top #2 | Operand A, bit 0 (LSB) |
| B1 | DIPSW bottom #1 | Operand B, bit 1 (MSB) |
| B0 | DIPSW bottom #2 | Operand B, bit 0 (LSB) |
| Cin | DIPSW-4 #1 | Carry input |
| S0 | DIPSW-4 #2 | Select bit 0 |
| S1 | DIPSW-4 #3 | Select bit 1 |
| S2 | DIPSW-4 #4 | Select bit 2 (AU/LU select) |

### Outputs (LEDs)

| Pin | Description |
|-----|-------------|
| F0 | Result bit 0 (LSB) |
| F1 | Result bit 1 (MSB) |
| Cout | Carry out from bit-1 adder (AU only, 0 when LU selected) |

---

## Technology

- **Simulation:** HTML5 Canvas, vanilla JavaScript
- **Logic family:** 7400-series TTL
- **Reference tool:** Proteus 8
- **Browser support:** Chrome 90+, Firefox 90+, Edge 90+, Safari 15+
- **No build step, no framework, no server required**

---

## File Structure

```
├── index.html          # Complete simulation (single file)
├── README.md           # This file
└── ALU_Documentation.docx.pdf  # Original design reference
```

---

## License

This project is provided for educational and internal team use. See original documentation for any additional restrictions.
```
