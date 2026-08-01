# Interview Flashcards — 🔧 Part 11 | Sensors

A set of 20 questions and detailed answers on sensors, for hardware/embedded technical interview prep.

---

## Question 1: Why is the industrial 4-20mA current loop preferred over a standard voltage output for long cables? Give a complete physical explanation.

**Short answer:** Because per KCL (Part 1), in a series circuit, current stays the same at every point along the path, regardless of how much resistance the wire has (only the voltage drops across that resistance, the current doesn't change). This means that even on a hundred-meter cable with noticeable voltage drop, the current arriving at the end of the line is exactly the same current the sensor sent.

**Full explanation:**

If the sensor gave a voltage output, the voltage drop across the cable's own resistance (which becomes significant over long distances) would directly cause a measurement error — the receiver at the end of the line would see a voltage lower than what the sensor actually sent.

But in a current loop, instead of changing voltage, the sensor changes the current flowing through a series circuit, proportional to the measured value (say, 4mA=minimum value, 20mA=maximum value). Per KCL, current stays constant throughout the entire length of a series circuit — the signal is nearly immune to cable voltage drop or induced noise (which usually adds voltage, not current).

That's why in factories and industrial facilities where cables can be hundreds of meters, the current loop is the golden standard — a beautiful application of a basic law (KCL) you saw at the very beginning of your learning path.

---

## Question 2: What's the difference between Accuracy, Precision, and Resolution? Why can a sensor have high Resolution but low Accuracy?

**Short answer:** Accuracy means how close the average of results is to the true value; Precision means how close together and clustered the results are (regardless of whether they're close to the true value or not); Resolution is the smallest difference the sensor can detect at all. These three are completely independent of each other.

**Full explanation:**

Archery analogy: imagine shooting several arrows at a target. Accuracy means how close the average landing point of the arrows is to the target's true center. Precision means how tightly clustered the arrows landed together — even if they're all far from the center but very close to each other, Precision is high, even though Accuracy is low. Resolution is the smallest distance you can distinguish on the target at all (recall LSB from Part 10).

A sensor can have high Resolution (say, it can show 0.01-degree changes — meaning it has very fine steps) but low Accuracy (its actual value could be off from the real temperature by 2 degrees — meaning it always shows a wrong value, just with very fine steps). Every sensor's datasheet usually gives all three numbers separately.

**🎯 Interview tip:** Weak answer: "these three all mean accuracy." Strong answer: a clear distinction of all three with an example, and an explanation of why they're independent — exactly the mistake the question says people "keep mixing up."

---

## Question 3: What is a sensor's Transfer Function? What's the difference between a linear transfer function (like an RTD) and a nonlinear one (like an NTC)?

**Short answer:** A transfer function is the relationship between an actual physical quantity (say, temperature) and the sensor's electrical output (say, voltage or resistance). An RTD has this relationship nearly linear (a change in temperature causes a proportional, predictable change in resistance); an NTC is nonlinear (its relationship is exponential, not linear).

**Full explanation:**

Recall from Part 2: an NTC is a resistor whose value decreases as temperature increases, but this relationship isn't a simple straight line — it's exponential, meaning the slope of change differs at different temperatures.

An RTD (which we'll see more of in the next question) is made of pure metal, and the resistance-temperature relationship of pure metals, unlike NTC, is almost perfectly linear and extremely stable and repeatable — that's why it's preferred over the cheaper NTC in industries where accuracy and long-term stability are critical.

Important practical difference: with a linear transfer function, converting the read voltage into a final physical value in software code is very simple (a simple linear equation is enough). With a nonlinear transfer function (like NTC), you need to use a more complex formula (like Steinhart-Hart) or a calibration table — more software complexity, exactly because of the component's different physics.

---

## Question 4: What's the difference between Offset (zero) calibration and Span/Gain (range) calibration? How does the two-point calibration method calculate both simultaneously?

**Short answer:** Offset calibration corrects a fixed error (say, your sensor always reads 2 degrees higher — you subtract a fixed number). Span/Gain calibration corrects an error that changes with the magnitude of the value (you apply a coefficient, not a fixed number). By measuring the sensor at two known points, both parameters are calculated simultaneously.

**Full explanation:**

Offset calibration: if your sensor, across its entire measurement range, is reading exactly a fixed amount wrong (say, always 2 degrees higher), it's enough to always subtract that same 2 degrees from the output — this is a simple additive/subtractive correction.

Span/Gain calibration: if the error changes with the magnitude of the value (say, your sensor is correct at low temperature but reads 5% higher at high temperature), subtracting a fixed number isn't enough — you need to apply a coefficient (not a fixed number) that applies a larger correction the larger the value is.

Common practical method — two-point calibration: you measure the sensor at two known values (say, melting ice=exactly 0 degrees, boiling water=exactly 100 degrees). Having two "true value versus read value" points, you can solve the equation of the straight line between these two points — which gives you both the Offset (y-intercept) and the Gain (slope of the line) simultaneously, exactly like the factory calibration of an ADC mentioned in Part 10 (ENOB).

---

## Question 5: What is sensor Drift, and why is there still sometimes a need for periodic calibration, even after a precise calibration?

**Short answer:** Drift means a sensor's physical characteristics slowly shift over time (or with changes in temperature, ambient humidity) — meaning a calibration that was precise today might be somewhat inaccurate six months from now, without any sudden event or specific failure having occurred.

**Full explanation:**

Calibration is a snapshot of the sensor's behavior at one specific moment — it assumes this behavior stays constant over time. But no real physical component is ever completely "constant": the chemicals inside a sensor gradually change slightly, soldered connections change their resistance slightly, and environmental factors (temperature, humidity, even UV light over the long term) gradually affect the component's exact characteristics.

More expensive sensors usually have less Drift due to higher-quality materials and construction, and their calibration stays valid for a longer time. For critical applications (industrial, medical, laboratory) where long-term accuracy matters, periodic calibration (say, every few months, again with the two-point method) is a standard part of system maintenance, not an extra, unnecessary task.

---

## Question 6: Why does a sensor's slow response time cause exactly the same Aliasing/Nyquist problem (Part 10)? Give a practical example of this mismatch.

**Short answer:** Because if you want to read a physical quantity that changes rapidly with a sensor whose response time is slow, the sensor can't keep up with those rapid changes — exactly like an ADC that, with a very low sampling rate, doesn't see a signal's high-frequency changes and incorrectly shows them as "smoothed out."

**Full explanation:**

Every sensor has a response time — how long it takes for its output to reach the new value after a sudden change in the actual physical quantity. This response time effectively imposes a frequency limit (bandwidth) on the sensor, exactly like an ADC's sampling frequency.

Practical example: a soil moisture sensor with a response time of a few seconds is perfectly suitable, because soil moisture changes slowly (the underlying physical phenomenon itself is slow). But this same sensor is completely unsuitable for measuring a motor's rapid vibration — motor vibration can oscillate hundreds of times per second, while the moisture sensor can't even track one full oscillation within its several-second response time. There, you need an IMU with a millisecond-level response time.

---

## Question 7: Why must an RTD be powered with a fixed, very small current source, not direct voltage like an NTC?

**Short answer:** Because if too much current passes through an RTD, it heats up slightly itself due to heat from P=I²R (Part 1) — and because an RTD's whole job is measuring temperature, this extra heat causes "Self-heating," meaning you end up measuring "the actual ambient temperature plus its own heat," not the actual ambient temperature.

**Full explanation:**

An RTD (most commonly Pt100/Pt1000) is made of a pure platinum wire, because the resistance-temperature relationship of pure metals, unlike NTC, is almost perfectly linear and extremely stable and repeatable — that's why it's preferred over the cheaper NTC in industries where accuracy and long-term stability are critical.

But there's an important practical point: unlike an NTC that you can place directly in a simple voltage divider (since its base resistance is usually higher and it draws less current), an RTD must be powered with a fixed, very small current. The reason is exactly that same P=I²R formula: if too much current passes through the RTD, it heats up slightly itself — and because its job is precisely measuring temperature, this extra heat directly affects the measurement result.

Professional RTD measurement circuits usually use a precision current source to keep this current always small enough that the self-heating error is negligible.

---

## Question 8: What is the Seebeck Effect? Why does a thermocouple without Cold Junction Compensation show an incorrect temperature?

**Short answer:** When you weld two wires of two different metals together at one point, and this point has a temperature difference from the free ends of the wires, a small voltage is automatically generated between the two free ends — this is the Seebeck effect. The voltage depends only on the temperature difference between the two junctions, not the absolute temperature of the hot junction; that's why if you don't separately measure the cold junction's temperature, you only know the "temperature difference," not the "actual temperature."

**Full explanation:**

Simple physics: the point where two different metals are welded together is called the "Hot Junction"; the free ends of the wires (which are at another temperature, usually room temperature) are called the "Cold Junction." If these two points have a temperature difference, a very small voltage (a few microvolts to millivolts per degree) is automatically generated, with no external power source at all.

Key and misleading point: this voltage depends only on the temperature difference between the two junctions. If you only read this voltage, you actually know the "temperature difference between the two ends," not the "actual temperature of the measurement point." For example, if the cold junction is at room temperature (25 degrees) and the read voltage corresponds to "a 500-degree difference," the hot junction's actual temperature is 25+500=525 degrees, not 500 degrees.

To find the actual temperature, you need to separately measure the cold junction's temperature too (usually with another sensor, like an RTD or a simple digital sensor next to the thermocouple connector) and add it to the calculation — this is exactly what dedicated thermocouple interface ICs (like the MAX31855) do automatically inside themselves.

---

## Question 9: Despite the complexity of cold junction compensation, why is a thermocouple still a popular temperature sensor?

**Short answer:** Because a thermocouple can measure extremely high temperatures (a few hundred up to even a few thousand degrees, like industrial furnaces) that a standard NTC or RTD simply cannot physically withstand — for extreme temperatures, it's practically the only viable option.

**Full explanation:**

Standard NTC and RTD sensors are made of materials that simply melt at extremely high temperatures (say, above a few hundred degrees) or their physical properties completely break down — they're physically incapable of measuring at those temperatures.

A thermocouple, because it consists of just two simple metal wires (which can be made from alloys highly resistant to heat) and a single weld point, can operate at temperatures that no standard semiconductor sensor can even get close to.

So despite its measurement circuit being somewhat more complex than a simple NTC (because of the need for cold junction compensation), in applications like industrial furnaces, combustion engines, and very high-temperature processes, it remains practically the only viable and economical choice.

---

## Question 10: Why is a capacitive soil moisture sensor far more durable than the resistive type? Give a complete physical explanation (electrolysis).

**Short answer:** Because the resistive sensor passes direct DC current from its bare electrodes through the moist soil, which causes an electrolysis chemical reaction and gradual corrosion of the electrode metal. The capacitive sensor measures moisture through a change in dielectric constant; its sensing surface is covered with an insulating layer, so no direct DC current ever passes through the soil, and electrolytic corrosion doesn't occur at all.

**Full explanation:**

Resistive type: two bare metal electrodes are inserted into the soil, and the resistance (or conductance) of the path between them is measured — wetter soil means better conductance (lower resistance). Problem: because direct DC current passes from the bare metal electrodes through the moist soil, an electrolysis chemical reaction slowly occurs that corrodes the electrode metal — exactly like accelerated rusting. After a few weeks to a few months of continuous use, the electrodes corrode enough that the reading becomes completely inaccurate.

Capacitive type: measures moisture through a change in dielectric constant (recall the physics of ceramic capacitors from Part 2), not through direct current passing through the soil with bare electrodes. The sensing surface is usually covered with an insulating layer, so no direct DC current ever passes through the soil, and electrolytic corrosion doesn't occur at all.

For a project that's going to stay outdoors for months in constant contact with moist soil, this durability difference is exactly what separates a sensor that fails after a few weeks from one that works for years.

**🎯 Interview tip:** This question is exactly one of the most common mistakes in amateur agriculture/gardening IoT projects — knowing the physical reason (not just "capacitive is better") demonstrates genuine understanding.

---

## Question 11: What's the difference between a photodiode and a phototransistor for sensing light? How does this relate to the internal mechanism of an optocoupler (Part 3)?

**Short answer:** A photodiode is a standard diode that, when light hits its junction, generates a current proportional to the light intensity — fast, but its signal is weak. A phototransistor is like a standard BJT, but light plays the role of "base current" instead of physical base current — because the transistor is itself an amplifier (hFE), it gives a much stronger signal, but for the same reason it's slower.

**Full explanation:**

Photodiode: a standard diode (Part 3) that, when light hits its PN junction, generates a current proportional to the light intensity. Faster and more precise than an LDR, but its signal (current) is weak and often needs an op-amp (Parts 3/10) to convert it into a measurable voltage.

Phototransistor: like a standard BJT transistor, but instead of base current coming from a physical wire, light plays the role of "base current" — this is exactly the same mechanism inside an optocoupler! Because the transistor is itself an amplifier (hFE, recall Part 3), a phototransistor gives a much stronger signal than a photodiode, but for the same reason (internal amplification, which is itself somewhat slow), its response is slower than a photodiode's.

So you see that same classic trade-off of signal strength versus speed here too — the choice between these two depends exactly on whether your project needs a stronger signal or a faster response.

---

## Question 12: How does a piezoresistive pressure sensor work? What's its direct connection to the instrumentation amplifier and strain gauge (Part 10)?

**Short answer:** A piezoresistive sensor is made of materials whose resistance changes when placed under mechanical pressure or tension — exactly the same physical principle of the strain gauge we saw in Part 10, just here packaged to measure fluid/gas pressure instead of direct mechanical tension.

**Full explanation:**

This is a good example of how one basic physical principle (resistance change with mechanical tension/pressure) finds different applications in different packagings: in Part 10, we saw this same principle as a strain gauge for directly measuring a structure's tension/bending; here, that same pressure-sensitive material sits behind a small diaphragm that converts the surrounding fluid or gas pressure into a mechanical force on that same material.

Because these sensors' output signal is also usually very weak (like the strain gauge), that same Part 10 solution (instrumentation amplifier) is used to amplify it before the ADC.

Common application in altimetry: atmospheric air pressure decreases fairly predictably with increasing altitude. Sensors like the BME280 use this same principle to estimate approximate altitude — their accuracy is usually a few meters, not centimeter-level, but that's completely sufficient for many projects.

---

## Question 13: Exactly how does a MEMS accelerometer convert a physical acceleration into a readable electrical signal?

**Short answer:** A microscopic structure (MEMS) contains a small mass connected to ultra-fine springs; when the device experiences acceleration (or even when stationary, under Earth's gravitational acceleration), this mass shifts slightly, and this microscopic displacement is measured through a change in an internal capacitor's capacitance.

**Full explanation:**

This is another example of capacitor physics (Part 2) reappearing in a completely different context: a small mass, suspended on ultra-fine springs at a microscopic scale, when the device accelerates, "lags behind" the main sensor body slightly according to Newton's second law (F=ma) — exactly like how your body gets thrown forward when a car suddenly brakes.

This microscopic displacement of the mass slightly changes the distance between two plates of a small internal capacitor; because a capacitor's capacitance depends on the distance between its plates, this capacitance change is measurable and directly proportional to the applied acceleration.

Important point: even when the device is completely stationary on a table, the accelerometer doesn't read zero — because it's always under Earth's gravitational acceleration (about 9.8 meters per second squared on the vertical axis). This is exactly the same principle that allows an accelerometer to also detect the direction of "down."

---

## Question 14: What's the problem with an accelerometer alone and the problem with a gyroscope alone for estimating angle? Exactly how does Sensor Fusion compensate for these two weaknesses together?

**Short answer:** An accelerometer is stable long-term (no Drift) but sensitive to momentary vibration and noise. A gyroscope is very smooth and precise short-term, but because angle is obtained by integrating the rate of rotation, small errors accumulate and the angle slowly deviates (Drifts). An algorithm (complementary filter or Kalman) uses the gyroscope for the short term and the accelerometer for long-term correction.

**Full explanation:**

These two sensors have exactly complementary weaknesses — each is strong exactly where the other is weak.

An accelerometer can directly calculate the "down" angle from the direction of the gravity vector, with no integration or error accumulation at all — so it stays precise and stable long-term. But any vibration or other momentary acceleration (like the device being shaken or genuinely moving) directly affects this calculation and makes it noisy.

A gyroscope gives the rate of rotation (not the absolute angle); to reach angle, you need to sum this rate over time (integrate). This process is very smooth and precise short-term, but because a small amount of error is also added to this sum at every moment and never self-corrects, these small errors gradually accumulate and the final angle slowly deviates.

A Sensor Fusion algorithm (most commonly a complementary filter for simplicity, or a Kalman filter for higher accuracy) intelligently combines both: it uses the gyroscope for fast, smooth short-term changes, and periodically "corrects" with the accelerometer (which is stable long-term) so the gyroscope's Drift doesn't accumulate.

---

## Question 15: Why does the angle calculated from a gyroscope alone slowly Drift over time? (This relates exactly to the integration process.)

**Short answer:** Because a gyroscope only gives the instantaneous rate of rotation, not the absolute angle; to reach angle, you have to continuously sum this rate over time (integration). Every raw gyroscope measurement always has a small amount of error/noise; because these errors are summed every time and never return to zero, the total error grows larger and larger over time.

**Full explanation:**

Imagine you have a slightly inaccurate scale that always reads 1 gram more than the actual weight. If you weigh just once, the error is only 1 gram — negligible. But if you want to calculate the total weight of an entire day's food by summing the weight of each meal, and this same 1-gram error repeats every time, after 20 meals the total error has reached 20 grams — error accumulates with repetition.

Integration is exactly this same "repeated summing," just instead of meals, you're summing very small values of rotation rate at very short time intervals (say, every few milliseconds) one after another. Even if each sample's error is extremely tiny, because this process repeats continuously without stopping, the cumulative error grows over time — this is exactly Drift.

Unlike an accelerometer, which calculates angle directly from gravity each time without accumulation (without this accumulation problem), a gyroscope is inherently subject to this accumulation problem — and exactly for this reason it needs Sensor Fusion.

---

## Question 16: What's the difference between the resistive shunt method and the Hall-effect sensor for measuring current? What advantage and disadvantage does each have?

**Short answer:** A resistive shunt must be placed in series in the current path (requiring the circuit to be opened) and always imposes some voltage drop and power loss. A Hall-effect sensor is completely non-contact — it senses the magnetic field around the wire from a distance, without opening the circuit and without any voltage drop, and it naturally provides a kind of free galvanic isolation too.

**Full explanation:**

Resistive shunt + op-amp (recall Parts 2 and 10): a very low-value shunt resistor is placed in the current path; the very small voltage drop across it (per V=IR) is amplified with an instrumentation amplifier and made readable by an ADC. Advantage: simple, precise, cheap. Disadvantage: you have to "open" the circuit and place the resistor in series, and it always dissipates some voltage and power (recall the formula P=I²R).

Hall-effect sensor: any wire that current passes through generates a magnetic field around itself (recall the coil/inductor principle from Part 2). A Hall-effect sensor can sense this magnetic field from a distance, with no direct electrical contact with the wire at all, and estimate the passing current from the field's strength.

Why this is great: unlike a shunt, which requires opening the circuit, a Hall sensor is completely non-contact and has no voltage drop — you can measure a wire's current without opening the circuit at all, and you naturally get a kind of free galvanic isolation too (recall Part 5). Its disadvantage is usually somewhat lower accuracy and higher price compared to a simple shunt.

---

## Question 17: An ultrasonic sensor measures a pulse round-trip time of 6ms. With a speed of sound of 340 meters per second, what's the distance to the obstacle?

**Short answer:** Using the formula Distance=(Speed×Time)/2: distance=(340×0.006)/2=1.02 meters.

**Full explanation:**

An ultrasonic sensor (like the famous HC-SR04) sends an ultrasonic sound pulse, times how long it takes for this pulse to hit an obstacle and return, and calculates distance from the speed of sound (which is nearly constant and known, about 340m/s at room temperature) and this round-trip time.

Important formula note: you must divide by 2, because the measured time is the total round-trip time of the pulse (to the obstacle and back), not just the one-way travel time. If you forget this division by 2, the calculated distance becomes exactly double the actual distance — a common, simple mistake that's easily avoidable.

These sensors are cheap and common, but their accuracy is limited (a few millimeters to centimeters), and they're sensitive to the obstacle's surface smoothness/orientation (if the obstacle's surface is angled or very soft, the sound scatters instead of returning directly).

**Formula / Calculation:**
```
Distance = (Speed_of_Sound × Time_of_Flight) / 2
Distance = (340 × 0.006) / 2 = 1.02m
```

---

## Question 18: What's the difference between an Incremental Encoder and an Absolute Encoder? How can an Incremental Encoder detect direction of rotation in addition to amount of rotation?

**Short answer:** An Incremental Encoder only counts "change" and forgets the actual position after power loss. An Absolute Encoder has a unique code for every physical position and immediately knows the current exact position right after powering back on. Direction detection in Incremental is done with two output channels (A and B) with a 90-degree phase difference.

**Full explanation:**

Incremental Encoder: has two output channels (usually A and B) that generate square pulses as the shaft rotates — but these two channels are set with a 90-degree phase difference relative to each other (called Quadrature). By counting the number of pulses, you know the amount of rotation; by seeing which channel changed first (A before B, or vice versa), you also know the direction of rotation — because this order depends exactly on the direction of rotation. Recall from the software roadmap: the PCNT hardware peripheral is designed exactly for counting these same pulses without involving the CPU with an interrupt for every single pulse.

Absolute Encoder: unlike Incremental, which only counts "change" (and forgets the actual position after power loss, needing to be "zeroed" again), an Absolute Encoder has a unique code for every physical position — even immediately right after powering back on, with no need for an initial rotation, it knows the current exact position.

Common application: motor position feedback (Servo/Stepper with precise feedback), rotary control knobs (like a digital volume knob), rotation speed measurement (by counting pulses per unit time).

---

## Question 19: For power-hungry sensors in a battery-powered project, why should you completely cut sensor power with a Load Switch, rather than just going into its own internal Sleep mode?

**Short answer:** Because some sensors don't have a good internal sleep mode, or don't have this capability at all — even in their "lowest power" state, they still draw a significant amount of current. By completely cutting power through a switching MOSFET, consumption during that interval genuinely reaches zero (or near zero), not just "less."

**Full explanation:**

Recall power budgeting from Part 4: some sensors (like some simple digital sensors) stay on all the time and constantly draw current; others (like the BME280 with Forced Mode) can sleep between measurements and get their consumption down to microamps — but not all sensors have this good internal capability.

Practical solution: recall the Load Switch from Part 4 — for sensors that don't themselves have a good internal low-power mode, you can use a simple switching MOSFET (Part 3) to completely cut the sensor's power while the microcontroller is in Deep Sleep, and turn it back on just moments before reading.

Practical difference: a sensor's internal Sleep mode usually still has some residual consumption remaining (even if very small); completely cutting power with a Load Switch brings consumption to genuinely near zero — this difference, over the course of months, can make a significant difference in the entire project's battery life, especially if the sensor has noticeable continuous consumption.

---

## Question 20: Design scenario: for an industrial project that needs to measure a furnace's temperature up to around 800 degrees, which would you choose among NTC, RTD, and thermocouple, and why? What extra peripheral circuit is needed?

**Short answer:** A thermocouple is the only viable option — because standard NTC and RTD simply cannot physically withstand temperatures this high (Question 9). Required peripheral circuit: cold junction compensation, usually with a dedicated interface IC like the MAX31855 that performs this calculation automatically.

**Full explanation:**

First you need to compare the temperature range against each option's physical tolerance: standard NTC and RTD simply aren't built for temperatures above a few hundred degrees — at this temperature they either melt or their physical properties completely break down (recall Question 9). A thermocouple, because of its simple structure (just two heat-resistant metal wires), is the only option that can physically operate at such a temperature.

But choosing a thermocouple alone isn't enough — you must also add the cold junction compensation peripheral circuit (recall Question 8), otherwise the read temperature will be significantly wrong (because the thermocouple's raw voltage only shows the temperature difference, not the absolute temperature).

Common practical industrial solution: instead of manually implementing this compensation (which needs a second temperature sensor next to the connector and precise calculations), a dedicated interface IC like the MAX31855 is used, which does this completely internally and automatically, and delivers a ready-made temperature number directly (usually over SPI, recall Part 7).

**🎯 Interview tip:** This kind of "component selection scenario" question shows exactly whether you've only memorized each individual option, or whether you actually know how to compare each one's physical limitations against the project's real needs — and importantly, whether you remember to mention the necessary peripheral circuit (compensation) too, not just the sensor's name.

---
