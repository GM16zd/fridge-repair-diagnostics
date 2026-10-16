# Phase 4: Testing Equipment and Methods

A blink code tells you where to look. This phase is about how to look properly.

The goal of testing isn't to find a part to replace. It's to **replace the exact part that failed, and nothing else.** Replacing components on a hunch is the fastest way to spend money and still have a broken fridge. Every test below has a clear expected value, so the result is a decision, not an opinion.

---

## 1. Equipment: only what matters

You don't need a workshop. These four things covered nearly everything in this project.

| Tool | Used for | Why it earns its place |
|---|---|---|
| **Digital multimeter** | Resistance (Ω), continuity, DC voltage | Every test in this guide is one of these three measurements |
| **12 V DC power supply with clip leads** | Running the fan and LED on their own; energizing a relay coil | Separates "is the part good?" from "is the board feeding it?" |
| **Screwdrivers and a plastic pry tool** | Opening covers and releasing clips | Interior covers are held by plastic clips that break easily |
| **Notebook or notes app** | A test log | Readings mean little if you can't compare them later, and it's what let the technician confirm my conclusion quickly |

---

## 2. Ground rules

1. **Unplug first.** The only exception is checking DC voltage on the board's low-voltage output connectors (section 4.8), and only there.
2. **Disconnect the part from the board before measuring resistance.** Otherwise the circuit around it creates parallel paths and you read a wrong value.
3. **Temperature matters.** NTC sensors and bimetal switches change with temperature, so note the room temperature next to every reading.
4. **Test in isolation.** A part that fails "in the fridge" can be perfectly healthy on its own. Powering it separately is the cleanest way to know.
5. **Write down every reading**, including the ones that pass.
6. **Replace only what fails.**

---

## 3. Components at a glance

| Component | Test | Healthy result |
|---|---|---|
| Evaporator fan | 12 V on a separate supply | Spins freely and smoothly |
| Defrost heater | Resistance across terminals | About 200 to 400 Ω |
| Thermal fuse | Continuity at room temperature | About 0 Ω |
| Bimetal thermostat | Continuity, warm and frozen | Open (∞ Ω) at room temperature, about 0 Ω when frozen below 0 °C |
| NTC thermistor | Resistance | About 5 kΩ at 25 °C, about 13 kΩ at 0 °C |
| Defrost relay | Coil resistance, contacts, energize test | See section 4.6 |
| Air passages | Visual and by hand | Drain clear, damper moves freely |
| 12 V rail | DC volts at an output connector | Close to 12 V |

---

## 4. Testing each component

### 4.1 Evaporator fan

**Why:** Code 11 means "no feedback from the fan." That doesn't necessarily mean the fan is dead.

**How:**
1. Look first. With the cover off, spin the blades by hand. Ice can jam them, and a bearing that grinds is a bad sign. Inspect the wires for cuts or burns.
2. Check the fan's label for its rated voltage (12 V DC on my unit).
3. Connect it to your 12 V supply, matching the polarity of the wires. Leave any feedback wire unconnected. Wrong polarity can damage a DC fan's electronics.

**Reading the result:**
- Spins smoothly: the fan is healthy, so the problem is upstream (power, control, or the feedback line).
- Doesn't spin: check the wires and connector, then consider replacing the fan.

### 4.2 Defrost heater

**Why:** It's the part that actually melts the frost.

**How:** Unplug the heater's connector and measure resistance across its terminals.

**Expected:** about **200 to 400 Ω**.

**Bad:** an open circuit (∞ Ω) means a blown element.

### 4.3 Thermal fuse

**Why:** It's the last-resort safety. It blows permanently if the heater overheats.

**How:** Continuity test at room temperature.

**Expected:** about **0 Ω**.

**Bad:** open (∞ Ω) means it has blown. **Replace the fuse, and inspect the PCB relay too.** A fuse only blows when something let the heater run too long, and a relay welded shut is a classic cause.

### 4.4 Bimetal thermostat

**Why:** It's the mechanical backup that cuts heater power if the NTC sensor lies.

**How and what to expect:**
- **At room temperature:** open (∞ Ω). This is **normal**, not a fault.
- **When frozen below 0 °C:** closed (about 0 Ω).

To test the closed state, let the thermostat get properly cold, for example by leaving it in the freezer for a while, then measure again.

**Bad:** it stays open even when cold, or stays closed when warm.

### 4.5 NTC thermistor (temperature sensors)

**Why:** The board relies on it to know when defrost is finished. A drifted sensor makes the board misjudge everything.

**How:** Unplug the sensor and measure its resistance.

**Expected:** about **5 kΩ at 25 °C** and about **13 kΩ at 0 °C**. Values on the unit in the video are listed in [Phase 3](3-error-code-reference.md#5-sensor-reference-values).

**Two checks that catch a drifted sensor:**
1. **Test at two temperatures.** A drifted NTC can look right at room temperature and be wrong when cold. A glass of ice water (about 0 °C) is a cheap way to get a second point.
2. **Watch the change.** Warming the sensor gently (for example, holding it in your fingers) should make the resistance fall smoothly. Jumpy or erratic readings mean the sensor is damaged.

**Bad:** open (OL), near 0 Ω, or erratic values.

### 4.6 Defrost relay

**Why:** The relay is the switch that sends mains power to the defrost heater. It lives on the power PCB.

**A note before you start:** the cleanest test is with the relay out of the board. Testing it in place is possible, but parts around it can distort your readings, especially on the coil.

#### What a 4-pin relay is

A 4-pin PCB relay is a single-pole, normally-open (SPST-NO) relay. It's really **two separate circuits inside one box**:

```
  A ──[ coil ]── B          C ──/ ──── D
  (electromagnet)           (switch, open at rest)
```

- **The coil (2 pins):** an electromagnet driven by the low-voltage control circuit (12 V on my board).
- **The contacts (2 pins):** a switch that the electromagnet closes. On the defrost circuit, it carries the heater's mains current.

The two circuits are **electrically isolated from each other.** That fact is what lets you tell the pins apart.

#### How to tell which pins are which

**Method 1: read the case.** Most relays have a small schematic printed on them. A rectangle is the coil, and a slanted line with a gap is the switch. The case also prints ratings, like a coil voltage (for example "12 VDC") and a contact rating (for example "10 A 250 VAC"). Match each drawn symbol to the pins it connects to.

**Method 2: look at the PCB.** The contact pins usually connect to thick, wide traces, often reinforced with extra solder, because they carry the heater current. The coil pins usually connect to thin traces leading to a driver transistor and a small diode. The contact pins are often spaced wider apart to keep the mains circuit isolated.

**Method 3: use the multimeter (this one is definitive).** With the relay out of the circuit, set the meter to resistance and measure the six possible pairs of pins. Label the pins A, B, C, D and expect this:

| Pair | Reading at rest |
|---|---|
| The **coil pair** | A steady, finite resistance (typically a few hundred ohms, and anywhere from about 100 to 1000 Ω is normal for a 12 V relay) |
| The **contact pair** | Open (OL / ∞) at rest, because it's a normally-open switch |
| Any pair mixing **one coil pin and one contact pin** (four combinations) | Open (OL / ∞) |

So the logic is: **the only pair with a finite resistance is the coil. The other two pins are the contacts.** If a coil-to-contact pair shows continuity, the relay is damaged.

#### Prove it: the energize test

1. Connect your 12 V supply to the two coil pins. Polarity doesn't matter on a standard relay, but if the case marks + and −, follow the marking.
2. **You should hear a click.**
3. Measure across the contact pins: the reading should drop to about 0 Ω (well under 1 Ω).
4. Disconnect the supply. You should hear it release, and the contacts should go back to open.

#### Reading the results

| What you see | What it means |
|---|---|
| Coil reads open (OL) | Burnt or broken coil. Replace the relay |
| Coil reads near 0 Ω | Shorted coil. Replace the relay |
| Coil is normal but there's no click | Check your supply voltage. If it's correct, the mechanism is stuck |
| Clicks, but the contacts stay open | Worn or burnt contacts |
| Contacts read a few ohms or flicker | Pitted or burnt contacts (high resistance makes heat) |
| Contacts stay closed after you remove power | **Welded contacts.** This is the failure the thermal fuse exists to protect against |

Also check the **relay's solder joints**. Because the contact pins carry heavy current, cracked or cold joints there are a classic cause of intermittent behavior.

#### The lesson my relay taught me

My relay passed every test. That only proved the relay was healthy. It did not prove the board was driving it. On my board there was no 12 V available to drive anything. **A part passing its own test doesn't prove the circuit around it is doing its job.**

Some boards also have a service or forced-defrost mode that lets you trigger defrost on demand. Check your manual for that.

### 4.7 Air passages (the boring but important one)

If frost has already built up, the parts can be perfect and the fridge will still fail.

- **Drain trough:** make sure it isn't clogged with ice, or meltwater has nowhere to go.
- **Damper duct:** confirm the mechanical damper leading to the fresh-food compartment opens freely by hand.

### 4.8 The 12 V rail: follow the voltage

This is the test that cracked my case. Once every part tested healthy, I stopped testing parts and measured what the board was giving them.

**How:**
1. Identify a low-voltage output connector, such as the fan or the fridge-light connector.
2. If you're unsure which wire is which, test the load on a separate supply first. That shows you the polarity the load expects.
3. With the board powered, set the multimeter to **DC volts**. Put the black probe on the ground pin and the red probe on the 12 V pin.

**Expected:** close to 12 V.

**Bad:** 0 V (or close to it). Then the board isn't producing the rail, and every 12 V load on it will be off, even if each of them is healthy.

**Safety:** measure **only at the low-voltage connectors**. Never probe the mains-side circuits. Keep the probes steady, because slipping across two adjacent pins on a live board can damage it.

### 4.9 Older units: mechanical defrost timer

On non-inverter fridges, turn the flathead screw on the timer housing by hand until it clicks into defrost mode. The compressor should stop and the heater should start to warm. If it doesn't, the timer (or the heater circuit) is the suspect.

---

## 5. Testing other models

Because all frost-free fridges share the same building blocks, this method transfers to almost any model:

| Function | What to look for on any model | Typical test |
|---|---|---|
| Temperature sensing | A 2-wire NTC thermistor | Resistance at a known temperature, compared with its siblings |
| Air movement | A DC fan with 2 to 4 wires | Separate 12 V supply, plus feedback and power checks |
| Defrost heating | A heater, a bimetal thermostat, and a thermal fuse in a series loop | Resistance and continuity of each |
| Switching | A relay on the main board | Coil resistance, contact check, energize test |
| Power | A supply stage feeding the low-voltage rails | DC voltage at the output connectors |

**A universal order of operations:**

1. Visual inspection.
2. Read the LEDs and decode the blinks.
3. Test the parts the code points at, in isolation.
4. If they all pass, measure the board's low-voltage outputs.
5. Only then decide what to replace, or where to get help.

Connector numbers, wire colors and blink counts will differ between brands and generations. **The mechanism doesn't.** If you understand how it works, the small differences won't stop you.

---

## Takeaways from this phase

- Four tools were enough: a multimeter, a 12 V supply, screwdrivers and pry tool, and a notebook.
- Every test has an expected value, so every result is a decision.
- Isolate parts before you condemn them, and isolate the board's output before you condemn the board.
- A part passing its bench test doesn't mean the circuit around it is doing its job.

**Next:** [Phase 5, repair and lessons learned](5-repair-and-lessons-learned.md)
