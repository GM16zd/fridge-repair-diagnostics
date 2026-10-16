# Phase 3: Error Code Reference

Samsung's inverter fridges are smart enough to tell you what's wrong, and the way they do it is by blinking an LED. If you can read the blinks, you can skip a lot of guesswork.

This page covers how to read the LEDs, what each code means, what usually causes it, and what to test first. Most of the codes come from my reference notes and a technician's video walkthrough of the same Samsung inverter platform. The two codes I personally saw on my own fridge were **11** and **5**.

> **Important:** blink tables can differ between models and board generations. Treat this as a reference for the RT32 platform and check your own model's service manual or label before trusting any single code.

---

## 1. How to read the LEDs

My fridge has two boards, each with an LED (left and right as seen from the back on my unit; your layout may differ).

| Board | Position on my unit | What its LED tells you |
|---|---|---|
| **IPM PCB** (inverter board) | Left | Steady = compressor is being driven. Blinks = inverter-side fault |
| **Power PCB** (main board) | Right, LED near the top | Blinks = a fault detected by the main board |

### What "normal" looks like at power-up

Learn this first, because it's your baseline:

1. You plug the fridge in. The IPM LED **flashes once and goes out**, meaning power has reached the board and the system is starting.
2. Shortly after, the IPM LED **glows steadily**, meaning the board is powering the compressor and the compressor is starting.
3. If something is wrong, an LED starts **blinking a code** instead.

### How to count a code

- Watch the LED and count the flashes in one burst.
- The LED then **pauses** and repeats.
- If there are **two codes**, you'll see one burst, a pause, another burst, then a longer pause. My fridge showed **11 blinks, pause, 5 blinks**.
- Count through at least two full cycles to be sure, and write the pattern down.

### How faults behave over time

Not every fault stops the fridge right away, which is worth knowing:

- **Some codes are ignored for hours.** The outdoor sensor fault, for example, may not stop the fridge for three to four hours. A fridge that "worked fine for a while and then quit" may have had a code the whole time.
- **Some codes stop everything.** A cabinet (fresh-food) sensor fault can shut down both compartments, with the IPM LED off and the compressor stopped.
- **The fan-feedback fault** (11 blinks) can let the compressor keep running for a while, but the fault eventually stops the system.

---

## 2. A mindset for using codes

**A blink code tells you what the board perceives, not necessarily what is physically broken.**

An 11-blink "freezer fan fault" means "I'm not getting feedback from the freezer fan." That has several possible causes: a jammed fan, a cut or burnt wire, a faulty fan, or a fan that simply isn't being powered. In my case the fan tested perfectly healthy on its own. Use the code to choose *where to test*, then let measurements decide.

---

## 3. Power PCB (main board) codes

| Blinks | Meaning | Common causes | First things to test |
|---|---|---|---|
| **1** | Freezer sensor | Open or shorted NTC sensor, loose connector, damaged wire | Sensor resistance, connector seating |
| **2** | Freezer evaporator sensor | Open or shorted NTC sensor, loose or disconnected connector | Sensor resistance (about 5.4 kΩ at 22 °C on the unit in the video) |
| **3** | Outdoor / ambient sensor | Open, shorted, or disconnected NTC sensor | Sensor resistance (about 5.4 kΩ at about 23 °C) |
| **4** | Freezer thermostat | Faulty thermostat, loose connector, wiring problem | Connector, wiring continuity, thermostat |
| **5** | **Defrost fault** | Open heater, blown thermal fuse, drifted NTC, opened bimetal, failed relay, or no drive signal | Full defrost chain (see [Phase 4](4-testing-equipment-and-methods.md)) |
| **6** | Cabinet (fresh-food) sensor | Open or shorted sensor | Sensor resistance (about 5.7 kΩ at 22 °C) |
| **7** | Cabinet sensor | Listed as a cabinet sensor in my notes, same as code 6. I couldn't confirm how the two differ | Check your model's wiring diagram, then test the sensors |
| **9** | Cabinet thermostat (electronic temperature control) | Faulty thermostat, disconnected wire, broken or loose connector, loose wire at the PCB | Reseat the connector, check the wires, test the thermostat |
| **10** (with 3 on the IPM board) | Compressor motor / compressor not starting | See the compressor section below | See below |
| **11** | **Freezer fan** | No feedback from the fan: jammed by ice, cut or burnt wire, faulty fan, or no supply to the fan | Spin by hand, test on a separate 12 V supply, then check the board's output |
| **12** | Fresh-food compartment fan | Same logic as code 11, for a fan in the fresh-food side | Same as code 11 |

Code 8 isn't in my notes. If your fridge shows a pattern that isn't listed here, use your model's service manual.

### Notes on the codes I know best

**11 blinks: freezer fan.** The board is not getting feedback from the fan. The classic cause is excess ice jamming the blades, and a cut, burnt, or faulty fan can also trigger it. On my fridge the fan was healthy, and the missing feedback was really a symptom of a missing 12 V supply.

**5 blinks: defrost fault.** The main board thinks the defrost system isn't doing its job. This code is a *starting point*, not a verdict. Follow the defrost chain part by part before condemning the board.

**3 blinks: outdoor sensor.** An NTC on the fridge body, plugged into connector 31 in the video. Because the system tolerates this fault for a while, it can be misleading.

**9 blinks: thermostat.** The electronic thermostat has a circuit on its back and is connected by a red and a black wire. A disconnected wire or a loose connector is enough to trigger the code, so start with the cheap checks.

**6 blinks: cabinet sensor.** In the video, this fault stopped the entire fridge, with no cooling and the IPM LED off. The sensor sits under a small cover next to the electronic thermostat and uses a blue wire.

**2 blinks: freezer evaporator sensor.** Mounted on the evaporator's return pipe, just after the accumulator, and connected by a yellow wire.

---

## 4. Inverter (IPM) board codes and compressor faults

| Pattern | Meaning | Common causes |
|---|---|---|
| **IPM LED: 3 blinks, and power PCB: 10 blinks** | Compressor not starting (the boards don't detect rotation) | Seized compressor pump; U, V, W motor wires cut, shorted or burnt; shorted IPM; faulty overload protector or PTC part |
| **IPM LED: 1 or 2 blinks** | IPM fault | The IPM has likely shorted. Replace the IPM, or the entire IPM board if the IPM isn't available separately |
| **IPM LED: 5 blinks** | IPM board microcontroller fault | Faulty microcontroller on the IPM board |

### Compressor start-up problems (3 + 10 blinks)

What to check, from cheapest to most involved:

1. **The U, V, W wires** that feed the compressor: look for cuts, burns, and loose connections.
2. **The compressor's electrical box:** the overload protector and the PTC part live here. If either is bad, this code can appear.
3. **The IPM** itself, which can short out even when the compressor is healthy.
4. **The compressor**, if its pump is stuck and can't rotate.

### Board-to-board communication

The two boards communicate through connecting cables (connectors 201 and 73 on the unit in the video). If that link fails, both boards stop and the whole fridge stops working. It's worth checking the connectors and wires early when both boards misbehave together.

---

## 5. Sensor reference values

All of the temperature sensors on this platform are **NTC thermistors**: resistance goes **down** as temperature goes **up**. Here are the values measured on the unit in the video, plus the general figures from my notes.

| Sensor | Where it is | Measured resistance | At what temperature |
|---|---|---|---|
| Outdoor / ambient | On the fridge body, connector 31 | 5.41 kΩ | About 23 °C |
| Cabinet (fresh-food) | Under the small cover with the electronic thermostat, blue wire | 5.68 kΩ | 22 °C |
| Freezer evaporator | On the return pipe after the accumulator, yellow wire | 5.36 kΩ | 22 °C |

**General NTC rule of thumb:** about **5 kΩ at 25 °C** and about **13 kΩ at 0 °C**.

**A useful trick:** all the sensors are the same type, so at the same temperature they should read about the same. If one sensor is far from its siblings, it's the suspect. Your exact curve is in your service manual, so treat these numbers as sanity checks.

See [Phase 4](4-testing-equipment-and-methods.md) for how to test an NTC properly, including a two-temperature check that catches a drifted sensor.

---

## 6. Using this on other models

The specifics change from model to model, but the method doesn't:

1. **Get your model's blink table.** Look in the service manual, the label inside or on the back of the fridge, or the manufacturer's support pages.
2. **Group the codes by subsystem:** sensors, fans, defrost, thermostats, compressor and inverter, communication.
3. **Learn the normal LED behavior** at power-up on your board before you look for faults.
4. **Test the part the code points at, in isolation.**
5. **If every part is healthy, check the supply** rather than replacing anything.

All modern frost-free fridges share the same building blocks, so once you understand the mechanism, differences in code numbers and connector labels are just labels.

---

## Takeaways from this phase

- A blink code is a **starting point**. It tells you where to test, not what to replace.
- Learn the normal power-up LED behavior first.
- Some faults stop the fridge immediately, while others can hide for hours.
- Codes for a fan or a sensor can be caused by a missing supply rather than a broken part.

**Next:** [Phase 4, testing equipment and methods](4-testing-equipment-and-methods.md)
