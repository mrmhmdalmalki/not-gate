# NOT Gate

A NOT gate **outputs the opposite of its input** (`Y = Ā`). It is the most basic active logic
gate, and the building block for every other gate.

This version is a **complementary (CMOS‑style) push‑pull design** — the cleanest, simplest way
to build a logic gate from transistors. It uses a matched **NPN + PNP pair** (the `2N3904`
and its partner the `2N3906`), so the output is **actively driven almost rail‑to‑rail**
(`~4.8 V` for `1`, `~0.2 V` for `0`) and draws **almost no current at rest** — perfect for
cascading into adders, registers and a CPU. It also carries an **indicator LED on the input
and on the output** so you can see the logic state on the board.

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

`0` is **not** an empty wire. Both levels are real voltages the output is actively connected to:

| Level | Connected to | Voltage |
|:-----:|:-------------|:-------:|
| `1` (HIGH) | the supply rail **Vcc** (through the PNP) | `≈ 4.8 V` |
| `0` (LOW)  | **ground (GND)** (through the NPN)         | `≈ 0.2 V`  |

One of the two transistors is **always** holding the output — the **PNP** pulls it up close to
`+5 V`, or the **NPN** pulls it down close to `0 V`. Because each transistor saturates, the
HIGH is `~4.8 V` (basically a full 5 V) and the LOW is `~0.2 V`. The output is never left
**floating**, and it is strong enough to drive many gates and an LED at once.

---

## How it is built

This is the bipolar‑transistor version of a **CMOS inverter** — just **two transistors**:

> a **2N3906 (PNP)** on top and a **2N3904 (NPN)** on the bottom, both collectors tied together
> as the output.

- **Q2 — 2N3906 (PNP, pull‑up):** emitter to `+5 V`, collector to the output.
- **Q1 — 2N3904 (NPN, pull‑down):** emitter to `GND`, collector to the output.
- Each base is driven from the input through its **own** base resistor (`R_B2` for the PNP,
  `R_B1` for the NPN).

<img src="images/circuit.png" width="900">

How it works — the input turns exactly **one** transistor on:

- **Input `0` (0 V):** the PNP sees its base pulled low (base 5 V *below* its emitter) → **PNP
  on**, NPN off → output **pulled up to ≈ 4.8 V** → `Y = 1`. The **output LED** lights.
- **Input `1` (+5 V):** the NPN sees its base pulled high → **NPN on**, PNP off → output
  **pulled down to ≈ 0.2 V** → `Y = 0`. The **input LED** lights.

That is the inverting behaviour `Y = Ā`, with a strong, almost rail‑to‑rail output.

### Why the bases need *separate* resistors

This is the one detail that makes or breaks the circuit. You must **not** tie both bases to one
shared node. If you did, the NPN (when on) would clamp that shared node down to `~0.7 V`, which
would *also* turn the PNP on — both transistors conduct at once, the output never reaches a
clean level, and current pours straight from `+5 V` to ground (**shoot‑through**). Giving the
NPN and PNP **their own base resistors** (`R_B1`, `R_B2`) lets each base sit at the right
voltage, so only one transistor is ever on at a steady input. At rest the gate draws almost no
current — just like CMOS.

### Why the indicator LEDs sit where they do

Each LED is on its **own branch to ground** (`A → R_in → LED → GND`, and
`Y → R_out → LED → GND`), separate from the logic, so brightness and switching never fight. The
output LED hangs **after** the output and is driven by the strong PNP, so it does not weaken the
logic level.

---

## Building it on a breadboard

Two transistors: `Q1` the **2N3904 (NPN)** and `Q2` the **2N3906 (PNP)**. Both share the **same
TO‑92 pinout** — flat face toward you, legs down, **E B C** from left to right:

<img src="images/pinout.png" width="360">

The wiring picture below is an actual **breadboard build**: the two power rails (+5 V red,
GND blue), the two transistors plugged in (legs **E B C**), and every connection as a
**colour‑coded jumper wire** (see the legend). Remember each column of five holes in a bank is
one electrical node.

<img src="images/wiring.png" width="900">

Connect the two transistors as follows (note the **only** difference between them — where the
emitter goes):

| Transistor | E (emitter) | B (base) | C (collector) |
|:-----------|:------------|:---------|:--------------|
| **Q1 — 2N3904 (NPN)** | **GND** | through R_B1 (10 kΩ) to Input A | **Output Y** (joined to Q2's collector) |
| **Q2 — 2N3906 (PNP)** | **+5 V** | through R_B2 (10 kΩ) to Input A | **Output Y** (joined to Q1's collector) |

Then add the indicators:

- **Input LED:** Input A → R_in (220 Ω) → LED → GND.
- **Output LED:** Output Y → R_out (220 Ω) → LED → GND.
- **Output Y** is the node where the two collectors meet; that is what you carry to the next gate.

Reminder: `+5 V` and `GND` are **nodes** (named connections), not physical positions. If a
result is wrong, the usual causes are a transistor's legs in the wrong holes, or **mixing up the
2N3904 and 2N3906** (they look identical — mark them!), so re‑check **E B C** against the pinout
and check each part number.

Quick test once wired: Input tied to **GND** → Output near **+5 V** (output LED on); Input tied
to **+5 V** → Output near **0 V** (input LED on). The gate should run cool — if a transistor
gets hot, a base resistor is missing or the two bases are shorted together.

---

## Components

### Transistors: one 2N3904 (NPN) + one 2N3906 (PNP)

The 2N3904 and 2N3906 are a **complementary pair** — same TO‑92 package, same **E B C** pinout,
opposite polarity. They are the most common general‑purpose pair in the world.

- **2N3904 — NPN:** turns on when its base is **high** (emitter at ground); pulls the output
  **down**.
- **2N3906 — PNP:** turns on when its base is **low** (emitter at +5 V); pulls the output **up**.
- **Package:** TO‑92 for both. **Pinout** (flat face toward you, legs down): **E, B, C** left to
  right.
- **Key ratings:** V_CE(O) ≈ **40 V** max, I_C ≈ **200 mA** max, current gain *hFE* ≈ **100–300**.
- **Substitutes:** BC547 / BC548 (NPN) with BC557 / BC558 (PNP), or 2N2222 (NPN) with 2N2907
  (PNP) — any matched NPN/PNP pair. **Re‑check the pinout**, as some PNP parts use a different
  leg order.

### Resistors

| Ref | Value | Job |
|:---:|:-----:|:----|
| R_B1, R_B2 | **10 kΩ** | **Base resistors**, one per transistor; set the base current and (being separate) stop the two transistors fighting. |
| R_in, R_out | **220 Ω** | **LED current limiters** (~13 mA at these levels): bright indicators. |

> Note on fan‑out: at 220 Ω each input LED draws ~13 mA, so an output that drives several
> gate‑inputs‑with‑LEDs is pushing real current. The 2N3906 can source it, but if you ever drive
> a large fan‑out you can raise the LED resistors (e.g. 470 Ω–1 kΩ) or lower the base resistors
> for more drive.

### LEDs (×2)

- Any standard indicator LED (e.g. 3 mm / 5 mm, forward voltage ≈ 1.8–2 V). One shows the
  **input** is HIGH, one shows the **output** is HIGH.

### Power

- A **+5 V** supply rail and a common **GND** (0 V) reference.

---

## Standards and references

**Gate symbol.** The distinctive-shape symbol follows the ANSI/IEEE standard for logic graphic symbols:

- IEEE Std 91-1984 and 91a-1991, *Graphic Symbols for Logic Functions* ([standards.ieee.org](https://standards.ieee.org/ieee/91_91a/241/)). The distinctive shapes originate from US MIL-STD-806; the international equivalent is IEC 60617-12.
- Free explainer: Texas Instruments, *Overview of IEEE Standard 91-1984* (PDF) ([ti.com](https://www.ti.com/lit/ml/sdyz001a/sdyz001a.pdf)).
- Symbols and truth tables overview: *Logic gate*, Wikipedia ([wikipedia.org](https://en.wikipedia.org/wiki/Logic_gate)).

**Transistor circuit.** This NOT gate is a **complementary push‑pull (totem‑pole) inverter** — a
matched NPN/PNP common‑emitter pair sharing one output, the bipolar analogue of the CMOS
inverter:

- *Push–pull / complementary output*, Wikipedia ([wikipedia.org](https://en.wikipedia.org/wiki/Push%E2%80%93pull_output)).
- *CMOS inverter* (the topology this mirrors), Wikipedia ([wikipedia.org](https://en.wikipedia.org/wiki/CMOS#Inversion)).
- *Logic Gates using Transistors*, Electronics Tutorials ([electronics-tutorials.ws](https://www.electronics-tutorials.ws/logic/logic-gates-using-transistors.html)).
- P. Horowitz and W. Hill, *The Art of Electronics*, 3rd ed., Cambridge University Press, 2015 (the BJT as a switch, and complementary push‑pull stages).
- A. S. Sedra and K. C. Smith, *Microelectronic Circuits*, Oxford University Press (BJT switch, complementary output stages, the logic inverter).
- T. L. Floyd, *Digital Fundamentals*, Pearson (logic-gate symbols and truth tables).

**Transistor parts.** 2N3904 NPN, onsemi datasheet ([PDF](https://www.onsemi.com/pdf/datasheet/2n3904-d.pdf)). 2N3906 PNP, onsemi datasheet ([PDF](https://www.onsemi.com/pdf/datasheet/2n3906-d.pdf)).

---

## Regenerating the diagrams

```bash
pdflatex circuit.tex
pdflatex symbol.tex
pdflatex wiring.tex
pdftoppm -png -r 400 circuit.pdf images/circuit   # -> images/circuit-1.png
pdftoppm -png -r 400 symbol.pdf  images/symbol     # -> images/symbol-1.png
pdftoppm -png -r 400 wiring.pdf  images/wiring     # -> images/wiring-1.png
```

> Use `pdftoppm`, not `pdftocairo`, at high DPI the Cairo backend can garble the fonts.
