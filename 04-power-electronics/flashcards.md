# Interview Flashcards — 🔧 Section 4 | Power Electronics & Power Supplies

A set of 20 questions with detailed answers on power electronics and power supplies, for hardware/embedded technical interview prep.

---

## Question 1: Why does a switching regulator physically have higher efficiency than a linear regulator (LDO)? Give the physical explanation, not just the definition.

**Short answer:** Because an LDO converts the extra voltage between input and output directly into heat (a total loss), while a switching regulator transfers that extra energy more efficiently by switching rapidly and temporarily storing it in an inductor/capacitor — almost none of the energy is wasted as useless heat.

**Full explanation:**

An LDO is like a valve that always stays half-open to keep the output pressure constant, no matter how much the input pressure varies — every extra volt it "closes off" turns directly into wasted heat.

Now imagine that instead of leaving this valve half-open, you open and close it at very high speed (say, thousands of times per second) — sometimes fully open, sometimes fully closed, never half-open. On its own, this produces a choppy, uneven current. Now you place a flywheel (the inductor) and a storage tank (a capacitor) after this valve — the flywheel takes these irregular chunks and turns them into a nearly uniform current, and the tank smooths it out completely.

This is exactly a switching converter: instead of wasting the extra energy as heat, it transfers energy between the input and output voltage more efficiently by switching rapidly (via a MOSFET) and temporarily storing it in an inductor/capacitor.

---

## Question 2: Explain the Buck Converter topology step by step — what happens when the switch is closed, what happens when it's open? Why is the inductor essential in this circuit?

**Short answer:** When the switch is closed (on), current flows from the input through the inductor to the output, and the inductor simultaneously stores energy. When the switch opens (off), the inductor, not wanting its current interrupted, continues that current through the diode. Without the inductor, when the switch opens, current to the load would be suddenly cut off, and the output would become a series of instantaneous pulses instead of smooth DC.

**Full explanation:**

A Buck converter is made of a switch (a MOSFET), an inductor, a second diode or MOSFET, and an output capacitor.

When the switch is closed, current flows directly from the input through the inductor to the load — at this moment the inductor is both supplying current to the load and storing some energy in its own magnetic field.

When the switch opens, the inductor, which "doesn't like" its current being suddenly interrupted (that same flywheel behavior), keeps that current flowing, but now completes its path through the diode (or second MOSFET) instead of the input.

Without the inductor, when the switch opens, nothing has stored energy to keep the current going — current to the load would be suddenly cut off, and instead of a relatively smooth DC output, you'd get a series of instantaneous pulses. By repeating this cycle rapidly (thousands to millions of times per second), the output becomes a relatively smooth DC voltage that's lower than the input.

---

## Question 3: You want to use a Buck Converter to go from a 12V source to a 3.3V output. What Duty Cycle is needed for this conversion?

**Short answer:** Using the Buck formula: D = V_out / V_in = 3.3 / 12 = 0.275, meaning the switch needs to be on for about 27.5% of each cycle.

**Full explanation:**

For an ideal Buck converter, the input-output relationship depends directly on the Duty Cycle of the switch's control signal (a reminder of PWM from Section 1): V_out = D × V_in.

Plugging in the numbers, D = V_out / V_in = 3.3 / 12 = 0.275. A Buck controller IC calculates and applies exactly this Duty Cycle by continuously measuring the output and automatically adjusting the PWM to keep V_out constant — this is exactly the concept of feedback: if the output sags slightly, the IC itself increases the Duty Cycle a bit to compensate.

**Formula / calculation:**
```
D = V_out / V_in = 3.3 / 12 = 0.275 (≈27.5%)
```

---

## Question 4: How does the Boost Converter topology make the output higher than the input? With a 3.7V battery and Duty Cycle=0.55, what does the output voltage become?

**Short answer:** In a Boost converter, while the switch is closed, a lot of energy is stored in the inductor (without going to the output); when the switch opens, the inductor adds this stored energy on top of the input voltage and sends it toward the output. With V_in=3.7V and D=0.55: V_out = 3.7/(1−0.55) ≈ 8.2V.

**Full explanation:**

The Boost topology is similar to Buck, but the component arrangement is different: the inductor is placed so that while the switch is closed, a lot of energy gets stored in it. When the switch opens, the inductor, holding this stored energy, adds it on top of the input voltage and sends it toward the output — the result is an output voltage higher than the input.

Analogy: imagine pulling someone back with a spring (storing energy, switch closed), then letting go — they launch forward faster than their original speed (the spring's/inductor's extra energy adds on top of the original motion).

Common use: boosting a single lithium battery's voltage (3.0-4.2V) up to a fixed higher voltage, like 5V to power a USB module.

**Formula / calculation:**
```
V_out = V_in / (1 − D)
V_out = 3.7 / (1 − 0.55) = 3.7 / 0.45 ≈ 8.2V
```

---

## Question 5: Why can neither a Buck alone nor a Boost alone convert a lithium battery's voltage (which swings both above and below the target voltage) into a perfectly steady value? What's the difference between Buck-Boost and SEPIC?

**Short answer:** Because a lithium battery drops from 4.2V (input higher than target) down to 3.0V (input lower than target), and a Buck alone can only step down while a Boost alone can only step up. A Buck-Boost can do both, depending on Duty Cycle, but usually outputs with reversed polarity; a SEPIC does the same job without reversing polarity.

**Full explanation:**

Say you want to always get a steady 3.7V out of a lithium battery. The problem: early in the charge cycle (4.2V), the input is higher than the output (needs Buck); but near the end of discharge (3.0V), the input is lower than the output (needs Boost). A Buck alone or a Boost alone can't cover both cases.

Buck-Boost is a topology that, depending on the Duty Cycle, can either step up or step down — but its simple version usually produces an output with reversed polarity (a limitation you have to account for in the design).

SEPIC (Single-Ended Primary-Inductor Converter) is a more complex version that does the same job without reversing polarity — which makes it more popular in battery-powered projects where the battery voltage crosses over the target voltage from both sides.

**✅ Practical note:** if your project's input is always higher than its output (like a 3.7-4.2V battery targeting 3.3V, where the input always stays higher), a simple Buck is enough and you don't need the complexity of Buck-Boost/SEPIC.

---

## Question 6: How does a synchronous switching regulator improve efficiency by replacing the diode with a second MOSFET? How is this related to the body diode?

**Short answer:** In the non-synchronous version, a diode (with a Vf of around 0.3-0.7V) supplies the second current path, and this voltage drop is wasted as heat. In the synchronous version, instead of a diode, a second MOSFET turns on at exactly the same moment the first switch turns off; since a MOSFET's Rds-on can be much lower than a diode's voltage drop, losses in this part drop significantly.

**Full explanation:**

In the Buck Converter explanation, we used an ordinary diode (or a Schottky) for the second path. This is the simplest and most common version, but that diode, even a Schottky, still has some Vf that's wasted as heat.

In the synchronous version, a second MOSFET is used, which turns on at exactly the same moment the first switch turns off (this precise timing coordination is handled by the controller IC, which is why it's called "synchronous"). Since a MOSFET's Rds-on can be much lower than a diode's voltage drop, losses in this part drop significantly — the converter's overall efficiency usually improves by a few percentage points, which can make a noticeable difference in battery life for battery-powered devices.

A reminder from Section 3: every MOSFET has an internal body diode. This body diode is actually the same backup path that naturally conducts during switching transition moments (before the second MOSFET is fully on) — meaning the synchronous design also intelligently takes advantage of the MOSFET's own inherent body diode.

**✅ Practical note:** most ready-made Buck modules you find on the market today are the synchronous type, since modern chips integrate both MOSFETs with precise control into a single IC — you no longer need to make this decision yourself.

---

## Question 7: You have a converter with 70% efficiency that needs to deliver 500mA to a load at 3.3V. What's the real power drawn from the battery, and how much power is wasted as heat?

**Short answer:** P_out=1.65W. At 70% efficiency: P_in = P_out/Efficiency = 1.65/0.70 ≈ 2.36W. Power wasted = P_in − P_out ≈ 0.7W.

**Full explanation:**

Efficiency formula: Efficiency(%) = (P_out/P_in) × 100, where P_out=V_out×I_out and P_in=V_in×I_in.

First we calculate P_out: P_out = 3.3V × 0.5A = 1.65W. Now, inverting the efficiency formula, we find the real power drawn from the battery: P_in = P_out/Efficiency = 1.65/0.70 ≈ 2.36W.

So about 0.7 watts is wasted purely as heat — this energy comes from that same limited battery you're trying to make last as long as possible. With a 90%-efficient converter, P_in would only be 1.83W — a direct difference in a real battery-powered project's battery life.

**Formula / calculation:**
```
P_out = V_out × I_out = 3.3 × 0.5 = 1.65W
P_in = P_out / Efficiency = 1.65 / 0.70 ≈ 2.36W
Power wasted ≈ 2.36 − 1.65 = 0.71W
```

---

## Question 8: Why isn't a switching converter's efficiency a fixed number? Why does efficiency drop at both very low load and very high load?

**Short answer:** At very low load, the converter's own control circuitry consumes a nearly fixed amount of energy (regardless of load), which becomes a large share relative to such a tiny load; at very high load, resistive losses (I²R in the inductor and switches) drag efficiency down again. The best efficiency usually happens somewhere in the middle of the converter's capacity range.

**Full explanation:**

At very low loads (like a microcontroller in Deep Sleep drawing only a few microamps), efficiency usually drops, because the converter's own control circuitry consumes a fixed amount of energy (regardless of load), which becomes a large share relative to such a tiny load.

At very high loads (near the converter's maximum capacity), efficiency drops again due to resistive losses (I²R in the inductor and switches) — the higher the current, the more these resistive losses grow, quadratically (I²).

For a battery-powered project that spends most of its time in Deep Sleep and occasionally has a WiFi consumption peak, this means you need to pay attention to the converter's efficiency at both very low load and peak load, not just average load.

**🎯 Interview tip:** demonstrating that efficiency "is a curve, not a fixed number," and explaining the two different reasons for its drop at each end of the load spectrum, shows more depth of understanding than just saying "efficiency is usually 90%."

---

## Question 9: What is a switching converter's output voltage ripple, and why does it exist at all? Why does the output capacitor's quality (ESR) directly affect it?

**Short answer:** Because a switching converter works by switching rapidly rather than a perfectly continuous current, the output always has a small, fast fluctuation riding on top of it (at the switching frequency) — this is called ripple, and it's an inherent characteristic of every switching converter, not a design flaw. If the output capacitor's ESR is high, that internal resistance itself adds to the output ripple.

**Full explanation:**

Unlike a linear regulator, which has practically no ripple (since its current is continuous), a switching converter inherently has this small fluctuation because rapid switching is the basis of how it works.

A reminder of ESR from Section 2: the output capacitor needs to be able to "smooth" these fast fluctuations. If the capacitor's ESR is high (like a cheap electrolytic), that internal resistance itself adds to the output ripple, since the oscillating current passes through the ESR and creates an extra voltage drop. That's why switching-converter outputs usually use low-ESR ceramic capacitors (or a combination of a large electrolytic + a small ceramic).

Why ripple matters: if ripple is large, sensitive circuits (like a precision ADC or an RF circuit) can pick up this fluctuation as noise in their measurement or signal.

---

## Question 10: What's the difference between Load Regulation and Line Regulation? When do you check each one in a regulator's datasheet?

**Short answer:** Load Regulation shows how much the output changes when the load current changes (with input voltage held constant). Line Regulation shows how much the output changes when the input voltage fluctuates (with the load held constant).

**Full explanation:**

These two specs quantify "how truly stable a regulator's output stays" — something you'll see in every regulator's datasheet (linear or switching).

Load Regulation: a number like "0.5%" means the output fluctuates by at most half a percent when the load swings from minimum to maximum.

Line Regulation: for example, when a battery drops from 4.2V to 3.7V, how truly stable does the output actually stay?

Why these numbers matter for component selection: for a sensitive analog circuit (like an ADC voltage reference), even a few millivolts of fluctuation can ruin measurement accuracy — here, low Load/Line Regulation (meaning high stability) is critical. For powering a relay or motor that already generates plenty of noise on its own, this level of precision isn't necessary.

---

## Question 11: Why can Inrush Current cause a microcontroller to reset? Exactly how does Soft-Start solve this problem?

**Short answer:** Because at the moment of power-up, the board's empty capacitors need to suddenly charge, drawing a momentary current far higher than steady-state; if the power supply can't deliver this current without a noticeable voltage drop, the source voltage sags and can cross the Brown-out threshold. Soft-Start raises the output voltage gradually over a few milliseconds so this momentary current is drastically reduced.

**Full explanation:**

At the moment a circuit powers on, it draws a momentary current far higher than its steady-state current — because the board's empty capacitors all need to suddenly charge up, and the more/bigger bulk capacitors are on the board, the bigger this momentary current is.

If the power supply or battery can't deliver this large momentary current without a noticeable voltage drop, the source voltage itself sags within that first fraction of a second — and if this sag crosses the Brown-out threshold, the microcontroller resets again even before it's really finished "turning on," and this cycle can repeat.

Instead of the converter suddenly starting at maximum Duty Cycle, a Soft-Start circuit raises the output voltage gradually and in a controlled way from zero to its final value over a span of a few milliseconds to a few tens of milliseconds — exactly like opening a water valve slowly instead of all at once, so the water pipe (the power supply) doesn't get shocked.

**✅ Practical note:** most modern switching-regulator ICs have Soft-Start built in and automatic; some even let you tune its duration with a small external capacitor.

---

## Question 12: What is Power Sequencing, and why does it only matter for some chips (not every project)?

**Short answer:** Power Sequencing means managing the precise order in which several different voltage rails on a board power up. It matters for complex chips (like memories and multi-core processors) that have several voltages at once; for simple projects with just one voltage rail (like an ESP32 with a few I2C sensors on 3.3V), it isn't an issue at all.

**Full explanation:**

Say you have a board with both 3.3V (for the microcontroller) and 1.8V (for, say, an external flash memory chip) — both powered from a shared 5V source, but through two separate regulators. Question: which one should turn on first?

Many ICs (especially memory chips and complex digital chips) require their different voltages to power up in a specific order (or at least with a very small time gap between them), or else, at the moment of power-up, some internal connections in the chip can lock into an invalid state (like a digital pin reading HIGH while the analog rail is still at zero) or even suffer long-term damage.

Practical solutions: the simplest method is designing the circuit so that one regulator only activates after another regulator reaches a certain threshold (using an Enable pin). For more complex systems, a dedicated Power Sequencer IC manages this.

For most simple projects with just one voltage rail, this isn't an issue at all — it starts to matter once multiple different rails and more complex chips enter the picture.

---

## Question 13: Why is Brown-out Detection a deliberate protective feature, not a hardware bug? If a project keeps resetting unpredictably, what should you investigate?

**Short answer:** Because digital circuits need a minimum supply voltage to work correctly; if the voltage drops below this level, the processor might execute instructions incorrectly or corrupt flash data. The Brown-out Detector circuit deliberately resets the system in a controlled way before this happens. If it keeps occurring, you should check the power supply, peak current, and decoupling — not disable Brown-out itself.

**Full explanation:**

Digital circuits (including the CPU inside a microcontroller) need a minimum supply voltage to work correctly, so their internal transistors can switch properly. If the supply voltage drops below this level, the processor might execute instructions incorrectly — for example, misreading a byte of memory, or getting interrupted in the middle of writing to flash (which can corrupt program data).

The Brown-out Detector (BOD) circuit continuously monitors the supply voltage; the moment it drops below a certain threshold, it resets the system itself before the processor enters unpredictable, faulty behavior. This is exactly like a driver, seeing the fuel warning light before the car's tank runs completely dry and the engine dies mid-road, choosing to stop the car themselves in a controlled way.

If your project keeps resetting unpredictably, a Brown-out is usually a symptom of a problem (a weak power supply, an unaddressed current peak, insufficient decoupling), not the problem itself.

---

## Question 14: Using the table below for a smart plant project, calculate the total average current and the real "worst-case simultaneous" peak current:
ESP32 idle: 100mA average | ESP32 WiFi TX: 600mA peak | Moisture sensor: 10mA average | Relay coil: 75mA | Pump driver: 800mA peak.

**Short answer:** Total average current (normal mode) ≈ 110mA. Real worst-case peak current (if WiFi + relay + pump activate simultaneously) ≈ 600+75+800 = 1475mA ≈ 1.5A.

**Full explanation:**

Calculation steps: total average current in normal mode (no active pump/relay) is simply the sum of average consumptions: 100 (ESP32 idle) + 10 (sensor) = 110mA.

The real peak current considers the worst possible case — the moment everything happens at once: 600mA (WiFi TX peak) + 75mA (relay coil) + 800mA (pump start peak) = 1475mA, roughly 1.5A.

Practical conclusion: even if the system's average consumption is only around 110mA, the power supply (or battery + converter) needs to be able to deliver a 1.5A peak without a noticeable voltage drop. If all of these happen at once and the power supply was only chosen for 500mA, a Brown-out Reset happens at exactly that moment.

**Formula / calculation:**
```
Total average current ≈ 100 + 10 = 110mA
Worst-case peak current ≈ 600 + 75 + 800 = 1475mA ≈ 1.5A
```

**✅ Practical note:** if you know the relay and pump will never turn on at exactly the same moment as a WiFi peak (say, because you control them with software logic), you can budget more realistically — but this has to be a conscious decision, not a random assumption.

---

## Question 15: Why does a real embedded board usually have several different voltage rails at once (like 5V, 3.3V, and 1.8V)?

**Short answer:** Because every chip has a different optimal operating voltage based on its manufacturing technology; 5V is the older standard voltage for some sensors and older interfaces, 3.3V is today's standard for most microcontrollers and modern sensors, and 1.8V or lower is for advanced, high-speed digital chips that need lower power consumption and less heat.

**Full explanation:**

5V: the older standard voltage, still common for some sensors, motors, and older interfaces (like some ready-made modules).

3.3V: today's standard for most microcontrollers (ESP32, STM32, most modern I2C/SPI sensors).

1.8V (or even lower): very advanced, high-speed digital chips (powerful processors, fast memories) often have their core running at much lower voltage — because at the ultra-small scale of modern transistors, a lower voltage means less power consumption and less heat generated for the same operating speed, which is critical in chips with millions of transistors.

Practical result: a real embedded board (not a very simple project with just an ESP32) often has several separate regulators — you need to account for each one separately, sum them up, and manage their power-up order (if needed, per Power Sequencing).

---

## Question 16: What is PSRR, and why does it matter when powering a sensitive analog circuit (like a sensor signal amplifier) from a switching converter's output?

**Short answer:** PSRR describes how well a circuit (like a regulator or an op-amp) can block noise on its supply input from reaching its output. If a sensitive op-amp with poor PSRR is powered from a switching converter's noisy output, that same switching ripple can appear directly in its output signal.

**Full explanation:**

PSRR (Power Supply Rejection Ratio) is measured in decibels (dB); the larger the number, the better.

Say you power a sensitive analog circuit (like a sensor signal amplifier built with an op-amp) from the output of a switching converter (which inherently has ripple). If the op-amp (or any other IC) has poor PSRR, that same switching ripple and noise can appear directly in the op-amp's output signal — meaning you're measuring your sensitive sensor, but you're also measuring your power supply's noise!

Practical solution: for sensitive analog circuits, either use regulators with high PSRR (often special-purpose LDOs), or add an extra filter (like a small RC or LC filter) between the switching converter's output and the sensitive circuit's power input, to filter out the remaining noise further.

---

## Question 17: Why is it rare in practice for a junior or mid-level embedded engineer to actually design a Buck/Boost converter from scratch? What two alternative solutions are usually used instead?

**Short answer:** Because designing a discrete switching converter from scratch is complex and requires specialized knowledge of magnetics design and high-power-signal PCB layout; instead, an integrated switching regulator IC or a complete ready-made module is usually used.

**Full explanation:**

Integrated Switching Regulator IC: a ready-made chip that itself contains the MOSFET(s), control circuitry, and sometimes even Soft-Start — you just add the appropriate external inductor and capacitors (per the datasheet). Common examples: the MP1584 family, TPS563201 (both common, cheap synchronous Buck ICs).

Complete Off-the-shelf Module: a small ready-made board (like the well-known LM2596 modules) where all the components (IC + inductor + capacitors) are already soldered onto a small board — you just connect the input and output, no PCB design needed at all.

**✅ Practical note:** using an integrated IC or a ready-made module isn't just acceptable, it's often the more professional choice — because designing a discrete switching converter's magnetics/PCB from scratch carries high risk (EMI noise, feedback instability) that the specialist teams building these ICs/modules have already solved and tested.

---

## Question 18: What's the practical rule for choosing between an LDO and a switching regulator? What two factors mainly drive this decision?

**Short answer:** The practical rule is based on two factors: the input-output voltage difference and the load current. If the voltage difference and current are both low, an LDO is often the better choice for its simplicity and low noise; if the voltage difference is large or the current is high, a switching regulator becomes nearly mandatory.

**Full explanation:**

If the input-output difference is small and the current is low (like 5V to 3.3V at under 200mA), an LDO is often the better choice thanks to its simple circuit (often just one IC + two capacitors), low noise, and low cost.

If the voltage difference is large or the current is high (like a 12V battery down to 3.3V at 1A), a switching regulator becomes nearly mandatory — otherwise the LDO gets so hot it's practically unusable — exactly the same P=(Vin−Vout)×I calculation we saw in Section 3.

The comparison table that summarizes this decision: a linear regulator has low efficiency (when the difference is large), very low noise, a very simple circuit, is cheap and small, and generates a lot of heat. A switching regulator has 85-95% efficiency, relatively more noise, more complexity, is more expensive and larger, and generates very little heat.

---

## Question 19: Why is it usually safe to assume "I'll always get exactly 5V from a USB-C port"? Even so, why should you know at least the basic concept of USB Power Delivery (USB PD)?

**Short answer:** Because most USB-C chargers default to 5V unless the device explicitly requests a different voltage, so this assumption usually holds without explicit PD negotiation. But modern USB-C ports can also offer higher voltages (like 9V or 12V), depending on the "negotiation" between the device and the charger — if you ever want to use these higher voltages, you'll need a dedicated USB PD controller IC.

**Full explanation:**

A lot of modern embedded projects (including many ESP32 development boards) are powered through a USB-C port. Modern USB-C ports can offer different voltages (not just the old fixed 5V, but sometimes 9V, 12V, or even more) and different currents, depending on the negotiation between the device and the charger — this process is called USB Power Delivery (USB PD).

If you design your project assuming it will always get exactly 5V from a USB-C port (without explicit PD negotiation), this assumption usually holds, since most chargers default to 5V unless the device explicitly requests a different voltage.

But if you ever want to draw a higher voltage from a PD adapter (say, for faster battery charging, or directly powering a stronger motor), you'll need a dedicated USB PD controller IC to carry out this "negotiation."

---

## Question 20: Why is disabling Brown-out Detection, even if it "solves" repeated resets, actually a wrong and even dangerous solution?

**Short answer:** Because Brown-out Detection is only the warning sign of a real underlying problem (a weak power supply, an unaddressed current peak, insufficient decoupling), not the problem itself. Disabling it just means the system keeps running under insufficient voltage, but this time without protection — which can lead to incorrectly executed instructions or corrupted flash data.

**Full explanation:**

When Brown-out keeps happening, a temptation many beginners fall into is: "since these resets are annoying, let's just disable Brown-out Detection in the microcontroller's settings so it stops resetting."

The problem is that this only turns off the warning sign; it doesn't fix the real cause of the voltage drop (which is still happening). Now the microcontroller keeps running despite the real voltage drop — without Brown-out Detection there to protect it. The result can be the processor executing an instruction incorrectly, misreading a byte of memory, or even worse, getting interrupted in the middle of writing to flash memory and permanently corrupting program data.

The correct solution is always finding and fixing the real cause of the voltage drop: strengthening the power supply, adding sufficient decoupling, or budgeting peak current more accurately — not turning off the warning system.

---
