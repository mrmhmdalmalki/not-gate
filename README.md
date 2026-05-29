# NOT Gate

A NOT gate **outputs the opposite of its input** (`Y = Ā`). It is the most basic
active logic gate, and the building block for every other gate.

### Symbol

A triangle with a small **bubble** on the output. The bubble always means *inversion*.

<img src="images/symbol.png" width="380">

### Truth table

| Input `A` | Output `Y` |
|:---------:|:----------:|
| 0 | 1 |
| 1 | 0 |

---

## What `0` and `1` really mean

`0` is **not** an empty wire. Both levels are real voltages the output is connected to:

| Level | Connected to | Voltage |
|:-----:|:-------------|:-------:|
| `1` (HIGH) | the supply rail **Vcc** | `+5 V` |
| `0` (LOW)  | **ground (GND)**          | `0 V`  |

So `0` means the output is **actively pulled down to ground (0 V)** through a conducting
transistor — not "no electricity." A wire connected to *nothing* is a third, undefined
state called **floating**, which we always avoid.

---

## How it is built

A single **common-emitter NPN stage**: emitter to ground, collector pulled up to `+5 V`
through a resistor, output taken at the collector.

<img src="images/circuit.png" width="620">

How it works:

- **Input `1` (+5 V):** the transistor turns ON (saturated) and conducts, so the collector
  is **pulled down to ground** → output `0`.
- **Input `0` (0 V):** the transistor is OFF, no current flows in `R_C`, so the collector is
  **pulled up to +5 V** → output `1`.

That is exactly the inverting behaviour `Y = Ā`. (Two of these stages in series make a
[buffer](../buffer).)

---

## Components

### Transistor — 2N3904  (×1: Q1)

- **Type:** **NPN** *bipolar junction transistor* (BJT) — a current-controlled switch: a
  small current into the **base** lets a much larger current flow from **collector** to
  **emitter**. Here it is used fully on/off, as a switch.
- **Package:** TO-92 (small black half-cylinder of plastic with 3 legs).
- **Pinout:** hold it with the **flat face toward you and the legs pointing down** — the pins
  are **E – B – C** (Emitter, Base, Collector) from left to right.
- **Key ratings:** V_CE ≈ **40 V** max, I_C ≈ **200 mA** max, current gain *hFE* ≈ **100–300**.
- **Why NPN (not PNP)?** The emitter sits at **ground**, so a HIGH (+5 V) on the base turns
  the transistor ON and drags the output **down to ground**. A PNP works upside-down
  (emitter at +5 V, on when the base is LOW) and would need the circuit re-wired.
- **Substitutes:** 2N2222, PN2222, BC547 — any general-purpose NPN. **Re-check the pinout.**

### Resistors

| Ref | Value | Job |
|:---:|:-----:|:----|
| R_B | **10 kΩ** | **Base resistor** — limits base current to a safe level while still switching the transistor fully on (`I_B ≈ (5 − 0.7)/10k ≈ 0.43 mA`). |
| R_C | **1 kΩ**  | **Collector pull-up** — provides the HIGH (+5 V) level and limits current when the transistor pulls the output low (`I_C ≈ (5 − 0.2)/1k ≈ 4.8 mA`). |

### Power

- A **+5 V** supply rail and a common **GND** (0 V) reference.

---

## Regenerating the diagrams

```bash
pdflatex circuit.tex
pdflatex symbol.tex
pdftoppm -png -r 600 circuit.pdf images/circuit   # -> images/circuit-1.png
pdftoppm -png -r 600 symbol.pdf  images/symbol     # -> images/symbol-1.png
```

> Use `pdftoppm`, not `pdftocairo` — at high DPI the Cairo backend can garble the fonts.
