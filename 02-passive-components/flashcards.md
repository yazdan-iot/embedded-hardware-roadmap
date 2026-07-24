# Interview Flashcards — Part 2: Passive Components

A set of 20 questions and detailed answers covering passive electronic components (resistors, capacitors, inductors, crystals, fuses, transformers, thermistors, etc.) for preparing for hardware/embedded technical interviews.

---

## Question 1: Why are resistors only manufactured in specific standard values (like 100Ω, 220Ω, 470Ω) instead of any arbitrary number? What exactly are the E12/E24/E96 standards based on?

**Short answer:** Because these series are arranged with **logarithmic spacing** (not linear) so that, given their respective tolerance, they cover the entire value range with the fewest possible values, without leaving large gaps or excessive overlap.

**Full explanation:**

If resistor values were spaced linearly (e.g., 10, 20, 30, 40...), large gaps would appear in the high-number range (say, between 1000 and 2000), while in the low-number range (between 10 and 20) values would overlap so much within their tolerance that they'd become practically redundant.

The industry solution is **logarithmic** spacing: each successive value is approximately a fixed multiple of the previous one, so that — accounting for tolerance — the range of each resistor begins exactly where the range of the previous one ends. A 10% tolerance requires 12 values per decade (E12 series), 5% requires 24 values (E24), and 1% requires 96 values (E96) — the tighter the tolerance (smaller number), the more values are needed per decade to avoid gaps.

**Formula / calculation:**
```
Ratio between consecutive values in a series with N values per decade:
ratio ≈ 10^(1/N)
E12 (N=12) → tolerance ±10%
E24 (N=24) → tolerance ±5%
E96 (N=96) → tolerance ±1%
```

**🎯 Interview tip:** This is exactly why, when a calculation gives a number like 86.6Ω, you have to round to the nearest standard value (e.g., 100Ω) — 86.6 simply doesn't exist in any standard series.

---

## Question 2: What exactly does resistor tolerance (e.g., 1% vs. 5%) mean, and when should you choose which? Give two real-world examples of each.

**Short answer:** Tolerance is the maximum allowed deviation of the actual resistance from the value printed on it. Use 1% tolerance wherever measurement accuracy matters (like a precise battery voltage divider); use 5% tolerance wherever only current limiting matters (like an LED resistor) — it's plenty adequate and cheaper.

**Full explanation:**

A resistor labeled "100Ω, 5% tolerance" means its actual value could be anywhere between 95Ω and 105Ω — the manufacturer offers no guarantee of anything more precise than that.

**Example 1 (5% tolerance is sufficient):** An LED current-limiting resistor. The human eye simply can't perceive the brightness difference caused by a 5% current variation, so there's no need for a more expensive resistor.

**Example 2 (1% tolerance is required):** A voltage divider meant to precisely measure battery voltage (as in the Part 1 example). If both divider resistors have 5% tolerance, the worst-case combined error can add roughly 9-10% error to the final result — meaning the value you calculate as the battery voltage could be noticeably wrong.

**✅ Practical tip:** Rule of thumb: wherever the circuit directly depends on numerical accuracy (voltage reference, measurement voltage divider, precision filter), use 1% tolerance (E96 series); wherever only general behavior matters (pull-up, LED), 5% tolerance (E24 series) is enough.

---

## Question 3: What exactly is the Temperature Coefficient of Resistance (TCR), and in which type of embedded project does it actually matter?

**Short answer:** TCR indicates how many parts per million (ppm) the resistance shifts per degree Celsius of temperature change. It's completely irrelevant for most hobbyist projects, but in precision analog circuits operating in variable-temperature environments (e.g., an outdoor project), it can degrade the overall accuracy of the circuit.

**Full explanation:**

For example, a TCR of 100ppm/°C means that with a 10-degree temperature rise, the resistance changes by about 0.1% — a number that's practically meaningless for an LED current-limiting resistor.

But imagine you have a precision voltage reference circuit for sensor calibration, or a sensitive voltage divider in an outdoor project (say, a smart planter sensor that gets hot under the sun and cold at night). If the resistors in that circuit have poor TCR, the overall measurement accuracy will degrade every time the air temperature changes — even if all the connections and values are electrically perfect. This is exactly the kind of error that's very confusing to track down without knowing the concept of TCR.

**🎯 Interview tip:** Showing that TCR isn't just a number on a datasheet, but directly affects the stability of an outdoor project, demonstrates the difference between a surface-level answer and a practical one.

---

## Question 4: You have a resistor with color bands "brown - black - red - gold." Calculate its resistance value and tolerance.

**Short answer:** The resistance is 1000Ω (i.e., 1kΩ) with a tolerance of ±5%.

**Full explanation:**

In the four-band color code, the first two bands are significant digits, the third band is the multiplier (number of added zeros), and the fourth band is the tolerance.

Brown = 1, black = 0, so the first two digits give "10." The third band, red, means a multiplier of ×100. So the final value is: 10 × 100 = 1000Ω. The fourth band, gold, means ±5% tolerance, so the actual value of this resistor lies somewhere between 950Ω and 1050Ω.

**Formula / calculation:**
```
Digit1=1(brown), Digit2=0(black) → 10
Multiplier=×100(red)
R = 10 × 100 = 1000Ω = 1kΩ
Tolerance = ±5%(gold) → actual range: 950Ω to 1050Ω
```

**✅ Practical tip:** Practice this skill several times with an online color code chart until you can read it at a glance. In a work setting, when you have several bins of mixed resistors in front of you, this is the only fast way to identify them, unless you have a multimeter handy.

---

## Question 5: How exactly does a current sense (shunt) resistor work? If you have a 0.1Ω shunt and measure a 50mV voltage drop across it, what is the current flowing through it?

**Short answer:** Using Ohm's law: I = V/R = 0.05V / 0.1Ω = 0.5A.

**Full explanation:**

A shunt resistor is a very low-value resistor (e.g., 0.1Ω or even 0.01Ω) deliberately placed in the path of a circuit's main current. Because current flows through this small resistor, Ohm's law produces a very small (millivolt-level) voltage drop across it.

By measuring this small voltage drop (usually with a dedicated IC called a Current Sense Amplifier) and applying Ohm's law in reverse (I=V/R), you can calculate the current flowing through the circuit — without having to break the circuit and connect an ammeter in series, which is exactly what the first question in Part 1 explained about ammeters, except this time it's done permanently and built directly into the PCB.

Why must the shunt value be so low? Because the higher its value, the more voltage drop and power dissipation it creates on itself, disrupting the main circuit — so it needs to be small enough to be practically negligible, yet large enough for its voltage drop to be accurately measurable.

**Formula / calculation:**
```
I = V_shunt / R_shunt = 0.05V / 0.1Ω = 0.5A
```

**🎯 Interview tip:** This is exactly the technique used by battery current-measurement circuits (fuel gauges) and overcurrent protection — knowing it shows you understand Ohm's law not just for simple calculations, but also its industrial application.

---

## Question 6: What is the difference between the C0G/NP0, X7R, and Y5V ceramic capacitor classes? Why should Y5V never be used for critical decoupling?

**Short answer:** C0G is extremely stable and precise but only available in very small values; X7R is a medium-value type with about ±15% deviation over temperature/voltage and is sufficient for general decoupling; Y5V is cheap and high-capacity but its actual capacitance can be 50-80% lower than its labeled value.

**Full explanation:**

**C0G/NP0:** Practically constant with respect to temperature and voltage, very precise, but only manufactured in small values (a few picofarads to a few nanofarads). This class is used for precision timing circuits (like crystal load capacitors).

**X7R:** Medium to large values (up to a few microfarads), but its value shifts somewhat with temperature and voltage (around 15%). Perfectly sufficient for general decoupling and the most common industrial choice.

**Y5V/Z5U:** The cheapest and highest-capacity in the smallest package, but its actual value (especially under high DC voltage or high temperature) can be 50 to 80% lower than the number printed on it. This means a "10µF" capacitor of this class might, in practice, only provide 2-3µF.

---

## Question 7: What is the "DC bias derating" phenomenon in ceramic capacitors? Why might a "10µF/16V" capacitor operating under a real 5V might only have an actual capacitance of 6 to 7 microfarads in practice?

**Short answer:** Because in ceramic capacitors (especially X7R and Y5V), as the applied DC voltage gets closer to their rated voltage, their actual capacitance drops noticeably below the value printed on the body — a phenomenon that runs counter to most beginners' expectations.

**Full explanation:**

This is a physical property of the dielectric material inside ceramic capacitors: the closer the DC voltage across the capacitor gets to its rated voltage, the more its actual capacitance decreases. For a "10µF/16V" capacitor operating on an actual 5V rail (about 31% of its rated voltage), this drop can easily bring the actual capacitance down to 6-7 microfarads.

The problem is that this reduction isn't obvious from the capacitor's main datasheet unless you look at a specific DC bias curve chart, so many beginner designers are simply unaware this phenomenon exists.

**✅ Practical tip:** The practical solution experienced engineers use: always choose a capacitor's rated voltage to be much higher than the actual operating voltage, not just slightly higher. For example, for a 5V rail, instead of a 6.3V capacitor (which is only slightly above), use a 16V or even 25V capacitor to operate on the flatter, more stable part of the DC bias curve.

---

## Question 8: What is ESR (Equivalent Series Resistance) of a capacitor, and why must you pay close attention to it for the output filter of a switching (buck) converter?

**Short answer:** ESR is the unwanted internal resistance that every real capacitor has (due to the material of its plates and internal electrolyte). In a switching converter's output filter, where large, fast currents constantly flow through the capacitor, high ESR causes extra heat and voltage drop, so a low-ESR capacitor (usually ceramic) should be chosen.

**Full explanation:**

Much like a battery's internal resistance causing its voltage to sag when drawing high current, ESR plays the same role for a capacitor: when a large current flows through it rapidly, this internal resistance causes an extra voltage drop (ripple) and generates extra heat within the capacitor itself.

Ordinary electrolytic capacitors have relatively higher ESR and aren't sufficient on their own for filtering the output of switching converters. That's why professional designs typically pair a low-ESR ceramic capacitor alongside a larger electrolytic capacitor — the electrolytic stores bulk energy while the ceramic absorbs fast fluctuations.

---

## Question 9: Why is reversing the polarity on a tantalum capacitor far more dangerous than making the same mistake with a regular electrolytic capacitor?

**Short answer:** Because unlike a regular electrolytic capacitor, which typically just gets damaged or slightly bulges under reverse polarity, a tantalum capacitor can suddenly short-circuit under the same conditions and even spark or catch fire.

**Full explanation:**

This difference comes down to the internal structure of tantalum's dielectric material; when these capacitors are powered in the wrong direction or above their rated voltage, unlike electrolytics which fail in a relatively "gentler" and more gradual way, tantalum capacitors can exhibit sudden and dangerous failure behavior.

That's why, whenever working with tantalum capacitors — whether during hand assembly or schematic design — you need to treat them with more caution than regular electrolytics: always double-check the polarity (the negative lead is usually marked with a stripe on the body), and always maintain a safe voltage margin (derating).

---

## Question 10: What is the difference between a bulk capacitor and a decoupling capacitor? Why can't one replace the other?

**Short answer:** A bulk capacitor (usually electrolytic, tens to hundreds of microfarads) is placed at the input of each power rail to store energy against slow, large fluctuations (like the whole board powering up or a motor starting); a decoupling capacitor (100nF near each IC) is for fast, localized pulses. The two complement each other rather than replace one another.

**Full explanation:**

These two capacitors cover two completely different time/frequency ranges of a circuit's current demand.

A bulk capacitor is physically larger and stores a lot of energy, but because it's larger, its internal inductance and trace path are also greater — it can't respond very quickly to a sudden, very short current pulse. That's why it's suited for slow, large fluctuations (like the moment the whole board starts up).

A decoupling capacitor is small, and because it sits close to the IC, its trace inductance is very low, so it can respond very quickly to extremely short pulses, but it has limited energy and isn't enough for large, sustained demands. If you only had one of these two, part of the circuit's current-demand spectrum would always go uncovered.

**🎯 Interview tip:** This is exactly the reasoning behind why you see both a 100nF and sometimes a larger capacitor (like 10µF) next to every IC — each covers a different part of the current-demand time spectrum.

---

## Question 11: What exactly does a coupling capacitor do, and how does its behavior relate to a concept discussed earlier?

**Short answer:** A coupling capacitor allows only the varying (AC) part of a signal to pass from one stage of a circuit to the next, while blocking the constant (DC) part — precisely due to capacitive reactance behavior (Xc is infinite at DC, very low at the signal's operating frequency).

**Full explanation:**

When you want to transfer only the variations of a signal from one analog circuit stage to the next, but the DC level (bias voltage) of the two stages differs, you place a capacitor in series in the signal path.

The reason this works comes down to the same reactance concept: at DC (zero frequency), the capacitor behaves like a completely open circuit, so the constant part of the signal is fully blocked. But at the signal's operating (AC) frequency, the capacitive reactance becomes low enough that the signal passes through with almost no loss. Common use case: separating the DC levels between two audio amplifier stages that have different bias reference voltages.

---

## Question 12: Why does suddenly cutting the current through an inductor (like a relay coil) produce a large voltage spike? How does this relate to the flywheel analogy?

**Short answer:** An inductor resists sudden changes in current. When the current path is suddenly broken, the inductor tries to keep the current flowing, and to do so, it discharges the energy stored in its magnetic field by producing a very high voltage (sometimes hundreds of volts).

**Full explanation:**

Imagine trying to spin a heavy flywheel — it's hard to start spinning because you have to put in a lot of energy to get it up to speed, but once it's spinning, it's also hard to stop it suddenly; it "wants" to keep rotating.

An inductor exhibits exactly this behavior, but with electrical current instead of mechanical rotation: it resists a sudden start of current, and once current is established and you suddenly try to break its path (say, by switching off a transistor or a switch), the inductor "resists" and tries to keep the current going by producing a very high voltage spike — because the energy stored in its magnetic field has to be discharged somewhere, even if its normal path has been closed off.

---

## Question 13: What is inductor core saturation, and why can choosing an inductor whose Isat is too close to the circuit's actual current damage a buck converter?

**Short answer:** An inductor's core can only sustain a magnetic field up to a certain limit, called Isat. Beyond this limit, the core saturates and no longer behaves like an inductor — only the very low resistance of its internal wire remains, causing the current to rise sharply and suddenly.

**Full explanation:**

Imagine a sponge absorbing water — it absorbs up to a point, but once it's fully soaked, it can no longer absorb more water, and the extra water passes right over it as if the sponge weren't even there.

Exactly the same thing happens to an inductor's core: when the current flowing through it exceeds Isat, the core saturates and no longer restrains the rapid rise in current. For a buck converter meant to supply, say, 2A, if you mistakenly choose an inductor with Isat=1.5A, the core will saturate at high load currents, the converter will lose control over the current, and typically either the inductor itself overheats badly or the converter's switching transistor gets damaged.

**✅ Practical tip:** Industry rule of thumb: always choose an inductor whose Isat is at least 20-30% higher than the circuit's actual maximum current, not exactly equal to or close to it.

---

## Question 14: In an EMI filter, why do inductors and capacitors behave in exactly opposite ways, and why does this very difference lead to combining them (a Pi filter)?

**Short answer:** Capacitive reactance decreases as frequency increases (the path opens up), while inductive reactance increases as frequency increases (the path closes off). The inductor is placed in series on the line to block high-frequency noise, and the capacitor is connected to ground to shunt that same noise away — this combination creates a stronger filter than either component alone.

**Full explanation:**

According to the reactance formulas (Xc=1/2πfC and XL=2πfL), these two components behave in exactly opposite ways: as frequency rises, the capacitor's reactance decreases (so at high frequency it acts like an open path to ground), while the inductor's reactance increases (so at high frequency it acts like a barrier in the main path).

A Pi filter (shaped like the Greek letter π) exploits this opposition: the inductor is placed in series on the main path and blocks high-frequency noise from passing; meanwhile, capacitors connected to ground on either side of the inductor shunt away any high-frequency noise that still managed to get through the inductor. The result is a filter that both blocks noise and shunts it away, unlike using just a single capacitor or a single inductor alone.

---

## Question 15: What is a crystal's load capacitance (CL), and why, if you choose its external capacitors incorrectly, does the crystal not "fail to work" but instead "work at the wrong frequency"?

**Short answer:** A crystal's actual oscillation frequency doesn't depend only on the crystal itself, but also on the total capacitance it "sees" (its two external capacitors plus the PCB's own parasitic capacitance). The crystal's datasheet specifies a particular CL value, and the external capacitors must be chosen so as to provide this value.

**Full explanation:**

If the external capacitors differ slightly from the value calculated from CL, the crystal will still oscillate (it won't stop), but its actual frequency will shift slightly — meaning that instead of oscillating at exactly, say, 8.000 MHz, it might oscillate at something like 7.998 MHz.

For a simple blinking LED, this small shift is completely imperceptible. But for fast, timing-sensitive communication (like USB or high UART baud rates), this small frequency error can cause real communication failures, because the two sides of the link no longer agree precisely on the same time interval.

**Formula / calculation:**
```
C_external ≈ 2 × (CL − C_stray)
Example: CL=12pF, C_stray≈3pF  →  C_external ≈ 2×(12−3) = 18pF
```

**🎯 Interview tip:** This question reveals whether you just know that "a crystal needs two capacitors" or actually understand why the exact value of those capacitors matters — the difference between memorizing a rule and understanding the physics behind it.

---

## Question 16: What problem does a crystal's frequency tolerance, expressed in ppm (e.g., ±20ppm), actually solve? Why does it matter in high-baud-rate serial communication between two devices?

**Short answer:** ppm indicates how much the actual frequency deviates from the nominal frequency. If the two ends of a communication link (say, two microcontrollers) have clocks with different errors, bit timing gradually drifts apart over time and communication starts to fail, especially at high baud rates where each bit's time window is very small.

**Full explanation:**

UART is an asynchronous protocol, meaning there's no shared clock wire between the two sides — each one estimates the timing of each bit based solely on its own internal clock. If each crystal has even a few ppm of error, this error is, by itself, very small.

But the problem shows itself when a long frame of bits comes in succession: each additional bit accumulates a bit more timing error, until at high baud rates (where each bit's time window is very small), this accumulation can cause the receiver to sample a bit's edge at the wrong point — misinterpreting the entire byte. A crystal with lower (more precise) ppm noticeably reduces this risk of accumulated error.

---

## Question 17: What exactly does the galvanic isolation provided by a transformer mean? Why is this feature critical for the safety of power supplies connected to mains electricity?

**Short answer:** It means two circuits can exchange energy or signals without having any direct electrical connection (not even a shared ground) — energy is transferred solely through the shared magnetic field between the two windings.

**Full explanation:**

A transformer consists of two (or more) windings wound around a shared core, but there's no shared copper wire between the two sides. Energy is exchanged only through the magnetic field, not through direct electrical contact.

This is exactly what guarantees electrical safety in power supplies connected to mains electricity (220V AC): the dangerous high-voltage side (connected to the wall outlet) is completely electrically separated from the low-voltage side (which you directly handle, like the 5V output of a charger). If a fault occurs on the high-voltage side, this separation prevents that dangerous voltage from being directly transferred to the output side that the user touches.

**🎯 Interview tip:** Even if in modern embedded projects (where you typically use a ready-made USB adapter or battery) you don't directly deal with transformer design, understanding this concept is essential for grasping why an isolated power supply is safer than a non-isolated one.

---

## Question 18: In a circuit with a pump or DC motor (which naturally draws a brief inrush current when turned on), why might choosing a fast-blow fuse instead of a slow-blow fuse be a mistake?

**Short answer:** Because a fast-blow fuse trips right at the moment of startup (when the natural, brief inrush current occurs), even if there's no actual fault in the circuit; a slow-blow fuse allows this brief peak to pass, but still trips if the high current continues for a longer duration (a sign of an actual fault).

**Full explanation:**

Motors and pumps draw a current much higher than their normal operating current for a brief moment (a few tens to a few hundred milliseconds) when they start up — this is called inrush current and is completely normal and expected.

A fast-blow fuse is designed to trip within a few milliseconds, without tolerating any short-term peak — meaning it trips at exactly the moment the motor starts, even when the circuit is completely healthy.

A slow-blow (time-lag) fuse is designed to tolerate these brief, natural peaks and only trips when the high current continues for a longer duration (which usually indicates an actual fault, like a jammed motor or a short circuit).

**✅ Practical tip:** For any circuit with a natural brief current peak at startup (motor, pump, transformer, incandescent bulbs), a slow-blow fuse is the correct choice.

---

## Question 19: What's the difference between a PTC (resettable fuse/polyfuse) and a regular fuse? Why is a PTC more commonly used than a regular fuse in a USB port or battery protection circuit?

**Short answer:** A regular fuse is single-use (it must be physically replaced after tripping); a PTC, when the current exceeds the allowed limit, heats itself up and its resistance suddenly increases by several hundred times, limiting the current, but once the fault is cleared and it cools down, it automatically returns to its normal state — no physical replacement needed.

**Full explanation:**

A PTC is made of a polymer material whose resistance rises sharply with increasing temperature. When the current exceeds the allowed limit, the PTC itself heats up due to that same excess current, its resistance increases almost like a switch opening — by several hundred times — and it sharply limits the current.

Imagine a protective muscle that tenses and stiffens when it senses excessive pressure to prevent further injury, and relaxes again once the pressure is relieved — a PTC behaves with exactly this kind of self-regulating response.

In USB ports (where a user can connect any device, even a faulty one) and battery protection circuits, this "automatic reset" feature is very valuable, because there's no need to open the device and physically replace a fuse after every potential short circuit.

---

## Question 20: How do you use an NTC thermistor to measure temperature with a microcontroller's ADC? Where does the relationship between the measured voltage and the actual temperature come from?

**Short answer:** You connect the NTC in series with a fixed resistor (as a voltage divider), read the voltage at the midpoint with the ADC, and since the NTC's resistance-temperature relationship is known (given in the datasheet as a table or the Steinhart-Hart formula), you calculate the actual temperature from the measured voltage.

**Full explanation:**

An NTC (Negative Temperature Coefficient) is a type of resistor whose resistance decreases as temperature increases — a completely predictable, well-defined behavior that the manufacturer documents in the datasheet.

Exactly like the voltage divider example for measuring battery voltage (Part 1), you connect an NTC in series with a fixed resistor and read the voltage at the junction between them with the ADC. Since the exact relationship between NTC resistance and temperature is known, you can calculate the NTC's instantaneous resistance from the measured voltage, and from that, the actual ambient temperature.

This is the cheapest and most common method of temperature sensing in hobbyist and semi-professional projects, before moving on to more expensive digital sensors like the DS18B20, which perform these calculations internally and simply deliver a ready-made number over a digital protocol.

**✅ Practical tip:** For outdoor projects (like a smart planter) that need to sense ambient temperature, this method is a good low-cost starting point, but keep in mind that its accuracy depends on the precision of the fixed series resistor and the correctness of the Steinhart-Hart formula used in the code.

---
