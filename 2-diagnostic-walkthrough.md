# Phase 2: Diagnostic Walkthrough

This is the full story of how I went from "the fridge is warm" to "there is no 12 V on this board." I've written it in the order things happened, and for each step I've tried to explain **why** we did it, not just what we did.

The guiding principle was simple: **narrow the problem, don't guess at it.** Every step either ruled something out or pointed at something new.

---

## Step 0: Read the symptoms before touching anything

**What we saw:**

- Freezer: cold, but not cold enough to keep food frozen
- Fresh-food compartment: at room temperature
- Context: summer heat and unstable mains voltage from an overloaded grid

**What I concluded, and why:**

Because the freezer was still cold-ish, the cooling itself was still happening somewhere. A dead refrigeration loop would have left both compartments warm. And because the fridge section was at room temperature, cold air wasn't reaching it. In a single-evaporator fridge (see [Phase 1](1-refrigeration-cycle.md)), the fresh-food section gets all of its cold from the freezer's evaporator via a fan and duct, so my first hypothesis was an **airflow or defrost problem**.

That was a hypothesis, not a conclusion. It told me where to look first, not what was broken.

---

## Step 1: Visual inspection

**What I did:** I pulled the fridge out and inspected the back: the compressor, the wiring, and the two circuit boards.

**What I found:** The compressor was running fine. Nothing looked burnt, bulged, or obviously damaged.

**Why this is always the first step:**

- It's free and it can't break anything.
- It gives you a baseline. You'll know whether the compressor is running, whether anything smells or looks scorched, and whether any connector has come loose.
- It rules out the loudest suspect quickly. A running compressor meant the compressor was very unlikely to be the root cause.

A clean visual inspection doesn't mean "nothing is wrong." It means "nothing is wrong *that I can see*." That distinction ended up mattering a lot at the end of this project.

---

## Step 2: Read the two LEDs

**What I found:** The Samsung has two boards at the back. The **IPM board** (inverter) is on the left and the **power PCB** (main board) is on the right. Each has an LED.

| Board | LED behavior | Meaning |
|---|---|---|
| IPM board | Steady | The compressor is being driven normally |
| Power PCB | Blinking a pattern | The main board has detected a fault |

**Why this narrowed things down a lot:** The steady IPM LED cleared the entire inverter and compressor side. I could stop worrying about the compressor drive and focus on the main board and whatever it controls.

The power PCB pattern was **11 blinks, a pause, then 5 blinks**, repeating.

---

## Step 3: Decode the blinks

I looked up the blink codes (full table in [Phase 3](3-error-code-reference.md)):

| Blinks | Meaning |
|---|---|
| 11 | Freezer fan fault (the board isn't getting feedback from the fan) |
| 5 | Defrost fault |

**Why the pair convinced me:** The two codes told one coherent story. If the defrost system fails, frost builds up on the evaporator. That frost can choke airflow and even jam the fan with ice, which triggers the fan code. A defrost failure that leads to a fan problem, and a fan problem that leads to a warm fridge section, matched every symptom.

**The trap I didn't notice at the time:** A coherent story is not the same as a confirmed one. The codes fit my physical hypothesis so neatly that I trusted it more than I should have. That is exactly the kind of moment where you need to test, not assume.

---

## Step 4: Test the physical parts inside the freezer

I opened the freezer, removed the inner cover over the evaporator area, and tested the parts one at a time. I used the defrost troubleshooting routine (full methods in [Phase 4](4-testing-equipment-and-methods.md)):

| Part | How I tested it | Expected | Result |
|---|---|---|---|
| **Evaporator fan** | Powered it from a separate power supply, away from the board | Spins freely and smoothly | Healthy |
| **Defrost heater** | Measured resistance across its terminals | About 200 to 400 Ω | Healthy |
| **Thermal fuse** | Continuity check | About 0 Ω | Healthy |
| **Bimetal thermostat** | Continuity check (open at room temperature is normal) | Open at room temperature, closed when frozen | Healthy |
| **NTC defrost sensor** | Measured resistance | About 5 kΩ at 25 °C | Healthy |

**Why the fan was tested on a separate supply:** If the fan spins fine when I power it myself, then its motor, bearings and windings are fine. Any fault has to be in what feeds it or in how the board reads it. It removes the board from the equation.

**Why I tested everything instead of replacing the "obvious" part:** Replacing a heater or a fan on a hunch could have cost real money and fixed nothing. The whole point was to replace the exact faulty part, or find out none of them were.

**Result: every single part was healthy.**

---

## Step 5: The "everything is fine" moment

I'll be honest: this is where I got frustrated. The board said the fan and defrost were faulty, and the fan and defrost parts were healthy. Both can't be true.

There were three ways to react:

1. Replace the fan or heater anyway "just in case," which wastes money.
2. Assume I'd measured something wrong and repeat the tests, which wastes time.
3. **Treat the contradiction as information.**

I went with the third. If the parts are healthy and the system still behaves as if they're broken, then the fault is in what **drives or supplies** them. Physical parts were now ruled out, which meant the problem must be on the board.

---

## Step 6: Check the defrost relay

**What I did:** The defrost relay on the power PCB is the switch that sends mains power to the heater. I tested it (procedure in [Phase 4](4-testing-equipment-and-methods.md#defrost-relay)).

**Result:** It passed.

**Why this was a useful negative:** A relay that passes its bench test is a healthy relay. It doesn't prove the board is *commanding* it, and that would turn out to be the real issue. At the time, though, I felt like I was out of tricks. Every component I could think of was good, and the system still didn't work.

---

## Step 7: My dad's contribution, and the turning point

While I was searching for more tips and watching repair videos, my dad had taken the fridge's interior LED light off and noticed it wasn't working either. He suggested testing it separately.

**What I did:**

1. Plugged the LED into a power supply. **It lit up immediately.** So the LED itself was fine.
2. Went back to the board and measured the LED's output cable to see where the electricity faded.
3. Found that **the circuit wasn't producing 12 V at all.**

That was the moment everything reorganized in my head. The fan runs on 12 V. The LED runs on 12 V. Both were "dead," and both were perfectly healthy when powered separately. It was the same pattern twice: a part that looked faulty *from the board's point of view* but was fine *on its own*.

Relay coils on boards like this are typically driven from the same low-voltage rail as well. I didn't verify that on my board, but it fits the picture. The blink codes for the fan and defrost weren't describing two separate broken parts. They were describing **two symptoms of one missing supply.**

> Think of the board as a small town. I had been inspecting the houses (fan, heater, sensor) one by one, and every house was fine. The power station that feeds all of them was the thing that had stopped.

---

## Step 8: Inspect the power stage, and know when to stop

Now I knew the problem was the 12 V generating section of the power PCB. I inspected it for anything obviously dead: burnt components, bulged capacitors, damaged traces. I didn't find anything.

This was the honest limit of my tools and experience. Some failures in a switching power supply don't show up as visible damage, and I didn't have the equipment or the knowledge to dig into it safely. So I took the board out and brought it to an experienced repair technician, along with my notes: the blink codes, what I tested, the results, and the 0 V measurement.

What happened next is in [Phase 5](5-repair-and-lessons-learned.md).

---

## How each step narrowed the search

| # | Question | Test | Result | What it ruled out or pointed to |
|---|---|---|---|---|
| 1 | Is the compressor running? | Look and listen at the back | Running | Compressor is very unlikely to be the root cause |
| 2 | Is the inverter drive healthy? | IPM board LED | Steady | Inverter side is fine |
| 3 | Is the main board reporting a fault? | Power PCB LED | 11, pause, 5 | Points at fan and defrost |
| 4 | Are the fan and defrost parts faulty? | Individual tests, fan on a separate supply | All healthy | Not the parts |
| 5 | Is the defrost relay bad? | Relay test | Passed | Not the relay |
| 6 | Do the "dead" loads work on their own? | Fan and LED on a separate supply | Yes | The loads are fine, so the supply to them is suspect |
| 7 | Is 12 V present at the board's output? | DC voltage at the LED output | 0 V | Fault is in the 12 V generation stage |
| 8 | Is anything visibly damaged there? | Visual inspection | Nothing | Needs specialist measurement |

Each row eliminated something. By row 7 I wasn't guessing anymore. I was holding a measurement.

---

## What I take from this phase

- **Read symptoms before touching parts.** They tell you which subsystem to suspect first.
- **Error codes describe what the board perceives, not necessarily what is broken.** An 11-blink "fan fault" meant "I'm not getting feedback from the fan." It did not mean "the fan is dead."
- **A pile of healthy parts is a clue.** When everything tests fine but the system fails, the problem is usually shared and upstream.
- **Test in isolation.** A part that looks faulty from the board's point of view may be perfectly healthy on its own.
- **Write it down.** The notes I took here are what let the technician confirm my conclusion quickly.

**Next:** [Phase 3, the error code reference](3-error-code-reference.md)
