# NOT Gate

A NOT gate **outputs the opposite of its input** (`Y = Ā`). It is the most basic
active logic gate, and the building block for every other gate.

This version is a **push-pull (totem-pole) design** built to be cascaded into bigger
circuits (adders, registers, a CPU): the output is **actively driven** both ways, so it
stays a strong logic level even while feeding several downstream gates and lighting an LED.
It also carries an **indicator LED on the input and on the output** so you can see the logic
state on the board.

### Symbol

A triangle with a small **bubble** on the output. The bubble always means *inversion*.

<img src="images/symbol.png" width="460">

### Truth table

| Input `A` | Output `Y` | LED in | LED out |
|:---------:|:----------:|:------:|:-------:|
| 0 | 1 | off | **on** |
| 1 | 0 | **on** | off |

---

## What `0` and `1` really mean

`0` is **not** an empty wire. Both levels are real voltages the output is connected to:

| Level | Connected to | Voltage |
|:-----:|:-------------|:-------:|
| `1` (HIGH) | the supply rail **Vcc** (through the top transistor) | `≈ 4 V` |
| `0` (LOW)  | **ground (GND)** (through the bottom transistor)      | `≈ 0.2 V`  |

In this push-pull design the output is **never left floating**: one of two output transistors
is always holding it — the **top** transistor pulls it up toward `+5 V`, or the **bottom**
transistor pulls it down to `0 V`. That is what makes the output strong enough to drive many
gates at once. (`1` is about `4 V`, not a full `5 V`, because the top transistor drops about
`0.7 V` across itself — that drop is normal and harmless; `4 V` reads as a solid HIGH.)

---

## How it is built

Three NPN transistors (all 2N3904):

> **Q1 = inverter**, then a **Q2 / Q3 push-pull output**.

- **Q1 (inverter):** a common-emitter stage. Emitter to ground, collector pulled up to `+5 V`
  through `R_C1`. Its collector is the **Ā node** (a clean `0.2 V`/`5 V` swing).
- **Q2 (top, pull-up):** an *emitter follower*. Its collector goes **straight to `+5 V` with no
  resistor**, its base is driven by the Ā node, and its emitter is the **output**. When Ā is
  HIGH, Q2 turns on and pulls the output up to `≈ 4 V`.
- **Q3 (bottom, pull-down):** a common-emitter switch. Base driven by the input `A`, collector
  is the **output**, emitter to ground. When `A` is HIGH, Q3 turns on and pulls the output
  down to `≈ 0.2 V`.

<img src="images/circuit.png" width="900">

How it works — the **bottom** transistor is driven by `A`, and the **top** transistor is
driven by `Ā`, so the two are never both fully on at a steady input (no contention, no sag):

- **Input `1` (+5 V):** Q1 turns on → Ā goes **low** → Q2 (top) **off**; meanwhile `A` is high
  so Q3 (bottom) is **on** → output is **pulled down to ≈ 0 V** → `Y = 0`. The **input LED**
  lights.
- **Input `0` (0 V):** Q1 is off → Ā goes **high** → Q2 (top) **on** → output is **pulled up to
  ≈ 4 V**; Q3 (bottom) is **off** → `Y = 1`. The **output LED** lights.

That is the inverting behaviour `Y = Ā`, now with a strong, cascadable output. (Two of these
NOT gates in series make a [buffer](https://github.com/mrmhmdalmalki/buffer-gate).)

### Why the indicator LEDs sit where they do

Each LED is on its **own branch to ground** (`A → R_in → LED → GND`, and
`Y → R_out → LED → GND`), separate from the logic. That way the LED brightness and the
transistor switching never fight each other, and — importantly — the **output LED hangs after
the output**, so it does not load the pull-up the way an LED across an ordinary collector would.
The base resistors can stay at a safe `10 kΩ` because the LEDs are not in the base path.

---

## Building it on a breadboard

Three transistors: `Q1` (inverter) on the left, then the push-pull pair `Q3` (bottom
pull-down) and `Q2` (top pull-up). Identify each 2N3904's legs with the pinout (flat face
toward you, legs pointing down, **E B C** from left to right):

<img src="images/pinout.png" width="360">

The wiring picture below is an actual **breadboard build**: the two power rails (+5 V red,
GND blue), the three transistors plugged in (legs **E B C**, left to right), and every
connection drawn as a **colour-coded jumper wire** (see the legend — +5 V, GND, input `A`,
`Ā`, and output each have their own colour). Remember that **each column of five holes in a
bank is one electrical node**, so a transistor leg and any wire or resistor sharing its column
are all connected.

<img src="images/wiring.png" width="900">

Connect each 2N3904 as follows:

| Transistor | E (emitter) | B (base) | C (collector) |
|:-----------|:------------|:---------|:--------------|
| **Q1 (inverter)** | GND | through R_B1 (10 kΩ) to Input A | through R_C1 (1 kΩ) to +5 V; this node is **Ā** and drives Q2's base |
| **Q2 (top, pull-up)** | **Output Y** (joined to Q3's collector) | the Ā node (Q1's collector) | **+5 V directly** (no resistor) |
| **Q3 (bottom, pull-down)** | GND | through R_B2 (10 kΩ) to Input A | **Output Y** (joined to Q2's emitter) |

Then add the indicators and the output tap:

- **Input LED:** Input A → R_in (470 Ω) → LED → GND.
- **Output LED:** Output Y → R_out (470 Ω) → LED → GND.
- **Output Y** is the node where Q2's emitter and Q3's collector meet; that is what you carry
  to the next gate.

Reminder: `+5 V` and `GND` are **nodes** (named connections), not physical positions, so the
+5 V rail can be the top or the bottom rail of your board. If a result is wrong, the usual
cause is a transistor's legs in the wrong holes, so re-check **E B C** against the pinout.

Quick test once wired: Input tied to **+5 V** → Output near **0 V** (input LED on, output LED
off); Input tied to **GND** → Output near **+4 V** (output LED on, input LED off). If the
output cannot stay HIGH under load, check that Q2's collector goes **straight to +5 V** with no
resistor in that leg.

---

## Components

### Transistors: 2N3904  (×3: Q1, Q2, Q3)

- **Type:** **NPN** *bipolar junction transistor* (BJT), a current-controlled switch: a
  small current into the **base** lets a much larger current flow from **collector** to
  **emitter**. Q1 and Q3 are used as on/off switches; Q2 is used as an emitter-follower
  (a current "valve" that passes the supply through to the output).
- **Package:** TO-92 (small black half-cylinder of plastic with 3 legs).
- **Pinout:** hold it with the **flat face toward you and the legs pointing down**, and the pins
  are **E, B, C** (Emitter, Base, Collector) from left to right.
- **Key ratings:** V_CE ≈ **40 V** max, I_C ≈ **200 mA** max, current gain *hFE* ≈ **100–300**.
- **Why NPN (not PNP)?** Q1 and Q3 have their emitters at **ground**, so a HIGH (+5 V) on a
  base turns them on. Q2 (the pull-up follower) has its collector on +5 V and passes the supply
  down to the output when its base goes high. A PNP works upside-down and would need re-wiring.
- **Substitutes:** 2N2222, PN2222, BC547, or any general-purpose NPN. **Re-check the pinout.**

### Resistors

| Ref | Value | Job |
|:---:|:-----:|:----|
| R_B1, R_B2 | **10 kΩ** | **Base resistors** for Q1 and Q3; limit base current while switching them fully on. |
| R_C1 | **1 kΩ**  | **Collector pull-up** for Q1; forms the Ā node and drives Q2's base. |
| R_in, R_out | **470 Ω** | **LED current limiters** (~4–6 mA): bright enough to read, light enough not to load the circuit. |

### LEDs (×2)

- Any standard indicator LED (e.g. 3 mm / 5 mm red, forward voltage ≈ 1.8–2 V). One shows the
  **input** is HIGH, one shows the **output** is HIGH. Lower `R_in`/`R_out` (e.g. 330 Ω) for
  brighter LEDs, or raise them (e.g. 1 kΩ) to draw less current.

### Power

- A **+5 V** supply rail and a common **GND** (0 V) reference.

---

## Standards and references

**Gate symbol.** The distinctive-shape symbol follows the ANSI/IEEE standard for logic graphic symbols:

- IEEE Std 91-1984 and 91a-1991, *Graphic Symbols for Logic Functions* ([standards.ieee.org](https://standards.ieee.org/ieee/91_91a/241/)). The distinctive shapes originate from US MIL-STD-806; the international equivalent is IEC 60617-12.
- Free explainer: Texas Instruments, *Overview of IEEE Standard 91-1984* (PDF) ([ti.com](https://www.ti.com/lit/ml/sdyz001a/sdyz001a.pdf)).
- Symbols and truth tables overview: *Logic gate*, Wikipedia ([wikipedia.org](https://en.wikipedia.org/wiki/Logic_gate)).

**Transistor circuit.** This NOT gate is a common-emitter RTL inverter (Q1) followed by a
**totem-pole / push-pull output** (Q2 pull-up emitter follower + Q3 pull-down), the same output
structure used by TTL logic so the gate can drive a real load and cascade:

- *Resistor-Transistor Logic (RTL)*, Wikipedia ([wikipedia.org](https://en.wikipedia.org/wiki/Resistor%E2%80%93transistor_logic)).
- *Totem-pole output / push-pull output*, Wikipedia ([wikipedia.org](https://en.wikipedia.org/wiki/Push%E2%80%93pull_output)).
- *Logic Gates using Transistors*, Electronics Tutorials ([electronics-tutorials.ws](https://www.electronics-tutorials.ws/logic/logic-gates-using-transistors.html)).
- P. Horowitz and W. Hill, *The Art of Electronics*, 3rd ed., Cambridge University Press, 2015 (the BJT used as a switch, and the emitter follower).
- A. S. Sedra and K. C. Smith, *Microelectronic Circuits*, Oxford University Press (BJT switch, emitter follower, and the logic NOT gate).
- T. L. Floyd, *Digital Fundamentals*, Pearson (logic-gate symbols and truth tables).

**Transistor part.** 2N3904 NPN, onsemi datasheet ([PDF](https://www.onsemi.com/pdf/datasheet/2n3904-d.pdf)), product page ([onsemi.com](https://www.onsemi.com/products/discrete-power-modules/general-purpose-and-low-vcesat-transistors/2n3904)).

**Highlighted source (additional).** The exact building block this design uses, scroll-to-text highlighted on the Wikipedia RTL page: [“a common-emitter stage with a base resistor”](https://en.wikipedia.org/wiki/Resistor%E2%80%93transistor_logic#:~:text=common-emitter%20stage%20with%20a%20base%20resistor).

---

## Regenerating the diagrams

```bash
pdflatex circuit.tex
pdflatex symbol.tex
pdflatex wiring.tex
pdftoppm -png -r 600 circuit.pdf images/circuit   # -> images/circuit-1.png
pdftoppm -png -r 600 symbol.pdf  images/symbol     # -> images/symbol-1.png
pdftoppm -png -r 600 wiring.pdf  images/wiring     # -> images/wiring-1.png
```

> Use `pdftoppm`, not `pdftocairo`, at high DPI the Cairo backend can garble the fonts.
