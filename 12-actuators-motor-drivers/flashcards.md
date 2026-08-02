# Interview Flashcards — 🔧 Part 12 | Actuators and Motor Drives

A set of 20 questions and detailed answers on actuators and motor drives, for hardware/embedded technical interview prep.

---

## Question 1: A relay is physically made of two completely separate circuits. What are these two circuits, and why does this structure itself count as a form of isolation?

**Short answer:** The coil circuit (a small electromagnet that mechanically pulls a metal lever) and the contact circuit (a completely mechanical switch that this same lever opens/closes). These two circuits have no direct electrical connection to each other — meaning a form of mechanical isolation (not electronic like an optocoupler) between the low-voltage control circuit and the power circuit, which could even be 220V AC.

**Full explanation:**

Coil circuit: a small electromagnet (an inductor, recall Part 2) that, when energized, mechanically pulls a metal lever.

Contact circuit: a completely separate, purely mechanical switch that this same physical lever opens/closes.

The key point that connects this back to Part 5: the low-voltage control circuit (which drives the coil) and the power circuit that the contacts switch are never directly connected to each other — they're only related through the physical motion of a lever, not through a shared wire or current. This is exactly the same goal as isolation (which we saw with light in the optocoupler and with a magnetic field in the transformer), just this time with a physical lever instead of electromagnetic energy.

---

## Question 2: Why are a relay's coil specifications (say, 5V/70mA) and its contact specifications (say, 250VAC/10A) two completely unrelated numbers? Why can mixing them up be dangerous?

**Short answer:** Because these two numbers refer to two completely separate parts of the relay (with no direct electrical connection, per the previous question): the coil specs are used for designing the driver circuit (transistor + base resistor + flyback); the contact specs tell you how much power current the mechanical switch itself can pass without being damaged.

**Full explanation:**

This is exactly what makes a relay useful in the first place: a weak signal (a 5V/70mA coil, easily driven with a GPIO + small transistor) can control a powerful load (a contact that switches 250VAC/10A) — these two numbers have no mathematical or physical relationship to each other; they're just placed together in one physical package.

The danger of mixing them up: if a designer assumes "because the relay coil is weak, the relay isn't safe for heavy loads" or the opposite, "because the contact tolerates 10 amps, any current under 10 amps is safe for this relay" without checking each part's actual specs separately, they might either reject the relay unnecessarily, or worse, connect a load with more current than the contact's actual capacity to it — which leads exactly to the contact welding shut (Question 5).

**🎯 Interview tip:** This is exactly likely the root cause behind your own project's failed-relay problem — always open the datasheet first and find these two numbers separately, don't assume.

---

## Question 3: What's the difference between an NO (Normally Open) and NC (Normally Closed) contact? Recalling the Fail-Safe principle from Part 5, when would you prefer NC?

**Short answer:** NO is open at rest (coil unpowered) and closes when the coil is energized; NC is the opposite, closed at rest and opens when the coil is energized. For a load that needs safe behavior during a power loss (like a safety system that should always stay active unless explicitly told to stop), NC makes more sense.

**Full explanation:**

This is exactly the same golden Fail-Safe question from Part 5: "When this circuit fails or an error (like a total power loss) occurs, which state should the system go into?"

With NO: if the coil fails for any reason (power loss, driver burnout), the contact returns to its default open state — meaning the load is cut off. Suitable when "being off" is the safe state (say, a dangerous load that shouldn't stay on without active control).

With NC: if the coil fails, the contact returns to its default closed state — meaning the load stays connected and active. Suitable when "staying connected" is the safe state (say, a safety system that should always stay active unless you explicitly give a stop command).

The choice between these two should always be based on that same Fail-Safe analysis, not simply whichever is more readily available.

---

## Question 4: What are the three main reasons for real relay failure? Explain each precisely.

**Short answer:** 1) Coil burnout (incorrect power supply or missing flyback diode, which burns out the driver transistor). 2) Contact welding (the most common reason, resulting from an electric arc when switching an excessive or inductive load). 3) Natural mechanical wear after the allowed number of cycles.

**Full explanation:**

**1. Coil burnout:** powering the coil with a voltage/current higher than its rated value, or a missing flyback diode that lets a return voltage spike burn out the driver transistor (full recall from Part 3) — if the transistor burns out, the coil can get stuck in the wrong state (always on or always off).

**2. Contact welding:** if the load being switched by the contacts draws more current than the contact's rated value, or if the load is inductive (like a pump motor) and the contacts are switched without proper protection, the moment the contact opens/closes produces a small electric arc that can melt and weld the metal of the two contacts together — result: the relay gets permanently stuck in the "closed" state, regardless of what the coil does.

**3. Mechanical wear:** even with completely correct use, contacts have a limited lifespan (the datasheet usually gives the allowed number of switching cycles, say 100,000 times) — after this number, natural mechanical wear occurs.

**🎯 Interview tip:** This question is exactly a systematic troubleshooting checklist — instead of guessing, rule out these three reasons one by one to find the actual cause.

---

## Question 5: What's the exact physical mechanism behind relay contact welding? Why does this count as the "most common cause of a mysteriously failed relay"?

**Short answer:** At the moment a contact opens or closes, a small electric arc forms between the two metal surfaces — this arc can be hot enough to actually melt and weld the metal of the two contacts together. It's the most common cause because this type of failure has no obvious external sign (burn marks, burning smell) — the relay just gets permanently stuck in the "closed" state, as if the coil had never turned off.

**Full explanation:**

When two metal surfaces carrying current separate (or come together), during that same fraction of a millisecond transition, a small electric arc forms between them — this phenomenon is more severe at high currents or with inductive loads (which generate a voltage spike upon disconnection, recall inductor physics from Part 2).

This arc produces a very high localized temperature — enough that it can actually melt the surface of the two contacts. If this happens a few times (or even once, under very bad conditions), the two metal surfaces, instead of separating after the arc, stick together and weld shut.

The reason it's "mysterious": from the outside, the relay looks completely healthy — no burn marks, bad smell, or visible damage. Only its behavior is strange: no matter how you turn the coil off, the load stays on, as if the control signal has no effect at all. This is exactly what you need to check: is your load's actual current (say, your pump) higher than your relay contact's rated current?

**🎯 Interview tip:** This is probably exactly what you need to check on your own project — compare your pump's actual current (and especially its startup/stall current, Question 19) against your relay contact's rated current.

---

## Question 6: The flyback diode on the coil and the Snubber on the contact — what's the exact difference between these two protections? Why might a relay need both simultaneously?

**Short answer:** The flyback diode is installed on the coil and protects the driver transistor against the coil's own return spike. A Snubber (series resistor + capacitor) is installed in parallel with the contacts and protects the contacts themselves against the inductive load being switched through them — these two protections are completely independent and solve two completely different problems.

**Full explanation:**

This is a subtle point that many people get wrong: they think "one flyback diode protects the whole relay," but the reality is that the relay's two completely separate parts (coil and contact, recall Question 1) have two completely different sources of risk.

The flyback diode (Part 3) is installed on the coil and protects the driver transistor against the coil's return spike — this is completely separate from any protection the contacts might need.

If an inductive load (like a pump motor) is switched through the relay's contacts (not through a transistor), a Snubber RC (full recall from Part 5) in parallel with the contacts can prevent arcing and welding (Question 5).

Summary: a relay driving a motor can need two separate protections simultaneously — a flyback diode on the coil (protecting the driver transistor), and a Snubber on the contact (protecting the contacts themselves against the inductive load they're switching).

---

## Question 7: Exactly how does a Solid-State Relay (SSR) work? Why does an SSR's failure mode differ from a mechanical relay's, and why does this difference matter in safety-focused designs?

**Short answer:** An SSR is essentially the same optocoupler (Part 3) whose output drives a power TRIAC or MOSFET — a replacement with no moving parts at all. Unlike a mechanical relay, which usually gets stuck in the "open/disconnected" state upon failure, an SSR tends more to fail in the "closed/permanently connected" state.

**Full explanation:**

An SSR combines two concepts you learned separately: the optocoupler (galvanic isolation with light), which receives the control signal, and a power TRIAC or MOSFET, which actually switches the load.

Advantages: no mechanical wear, no arcing/welding of a contact (since there's no physical contact at all), quieter and faster switching, a far higher switching lifespan (millions of cycles, not hundreds of thousands).

Disadvantages: more expensive than an equivalent relay, and because its internal switch is a semiconductor component (not a simple metal contact with nearly zero resistance), it always has some small voltage drop (and resulting generated heat).

Important safety note: when a power semiconductor component (like a TRIAC or MOSFET) fails due to overvoltage or overcurrent, its most common failure mode is an internal short circuit (not an open circuit) — meaning an SSR tends more to fail in the "always on" state, unlike a mechanical relay, which usually (except for welding) gets stuck in the "open/disconnected" state. This difference means that in a safety-focused design where "disconnecting upon failure" is critical, this assumption about the type of failure must be explicitly accounted for in the design.

---

## Question 8: How does an H-Bridge with four switches allow bidirectional rotation of a DC motor? Exactly what phenomenon does the Brake mode use?

**Short answer:** By closing two specific diagonal switches, current passes through the motor in one direction (forward); by closing the other two diagonal switches, the current's direction reverses (backward). In brake mode, both switches on one side (top or bottom) close simultaneously — the motor's two terminals are short-circuited together, and the motor, still spinning, resists this short circuit because of its own Back-EMF and stops quickly.

**Full explanation:**

Four switches (today usually MOSFETs) are arranged in a specific way that creates four possible states:

**Forward:** one pair of diagonal switches closed — current passes through the motor in one direction.

**Reverse:** the other diagonal pair closed — the current's direction reverses.

**Brake:** both switches on one side (top or bottom) closed simultaneously — the motor's two terminals get short-circuited together. The motor, still spinning due to inertia, itself (per the Back-EMF phenomenon we'll see fully in Question 19) acts like a small generator and produces a voltage; this voltage now resists the short circuit it has itself created and quickly converts the motor's kinetic energy into heat — the motor stops quickly.

**Free/Coast:** all switches open — there's no current path at all, and the motor spins freely until it stops on its own (just from natural friction).

---

## Question 9: Despite its high popularity, why isn't the L298N always an optimal choice for new projects?

**Short answer:** Because the L298N uses older technology (bipolar transistor/BJT), not MOSFET; its voltage drop and generated heat are far greater compared to modern MOSFET-based H-Bridges.

**Full explanation:**

Recall from Part 3: when a BJT enters saturation, it still has a fairly noticeable voltage drop on itself (Vce_sat, about a few tenths of a volt to about one volt). A MOSFET in full saturation has a drop of only I×Rds-on, which can reach just a few millivolts — a difference we saw in Part 3 for the general BJT/MOSFET comparison, here directly affecting a real, widely-used IC.

This extra voltage drop in the L298N, especially at higher currents, directly converts into wasted heat (recall P=V×I) — meaning both the overall circuit efficiency drops, and a larger heatsink is needed to keep the IC cool.

For new projects, MOSFET-based H-Bridge drivers (which are abundant and cheap today) usually have far better efficiency — the L298N has remained popular more because of its age and educational availability, not because it's technically the best option available.

---

## Question 10: In a modern MOSFET-based H-Bridge, what role does the MOSFET's Body Diode play that didn't exist in this form in the older L298N design?

**Short answer:** Each of the four MOSFET switches, exactly like a relay coil, needs a current discharge path for the motor's Back-EMF. In modern MOSFET designs, that same internal MOSFET body diode often plays this exact role, with no need for a separate external diode for each one.

**Full explanation:**

Recall from Part 3: every MOSFET, because of its physical construction, has an internal parasitic diode between drain and source called the Body Diode — not intentionally designed in, but always present.

A DC motor (which itself has a winding/inductor) exactly like a relay coil, needs an alternate path for its remaining current during fast switching moments — otherwise the same voltage spike we saw for the relay coil occurs here too, and it can damage the MOSFETs themselves.

In older bipolar-transistor-based H-Bridges (like inside the L298N), separate external protection diodes usually had to be added for each of the four switches. In modern MOSFET designs, each MOSFET's own internal body diode often plays this exact role for free, with no extra component — a beautiful example of intelligently using a component's "side" physical property instead of adding a new component.

---

## Question 11: How does Microstepping work in a stepper motor driver? What advantage does it have over Full Step?

**Short answer:** Instead of fully turning each motor phase on/off (Full Step), the driver divides the current between two adjacent phases proportionally (not zero or a hundred percent) — this creates much finer steps and motion far smoother than normal step-wise motion.

**Full explanation:**

A stepper motor rotates by fully turning its phases on/off in sequence (Full Step) — each step is a fixed and fairly large angle (say, 1.8 degrees), which at low speed can look fairly "step-wise" and jerky.

Microstepping, instead of this complete zero/hundred switch, divides the current between two adjacent phases proportionally — for example, instead of phase A being fully on and phase B fully off, it can activate phase A at 70% and phase B at 30%, which keeps the rotor "suspended" at an intermediate point between two full steps.

By gradually changing this ratio (instead of a sudden jump), the driver can create much finer steps (say, 1/16 or 1/32 of a full step) — the result is motion far smoother, quieter, and with more precise position control than normal Full Step stepping.

---

## Question 12: Why is setting the phase current potentiometer on a stepper driver (like an A4988 or DRV8825) a critical balancing decision?

**Short answer:** If the set current is too high, the motor and driver overheat excessively; if too low, the motor doesn't have enough torque to overcome the load and "loses steps" — meaning its actual position differs from the position you think it's at, without any software error having occurred.

**Full explanation:**

Most stepper drivers have a small trimmer potentiometer on themselves (recall Part 2) that sets each phase's limited current. This setting must exactly match your specific motor's specs.

If too high: per the formula P=I²R (recall Part 1), more current means more heat in the motor's windings and in the driver IC itself — this can cause Thermal Shutdown (Part 5) or even permanent damage.

If too low: the motor doesn't have enough torque to overcome its actual mechanical load — at moments when the load (friction, inertia, mechanical resistance) exceeds the available torque, the rotor "loses steps," meaning the driver thinks it has reached a specific position, but the actual mechanical rotor has fallen behind — this error isn't detectable in software at all (because the driver itself has no feedback, recall that a stepper is open-loop) unless you check with an added Encoder.

---

## Question 13: Why does a servo motor's PWM signal have nothing to do with the concept of standard power PWM (Duty Cycle = percentage of power)? What's actually being encoded?

**Short answer:** A servo signal is usually at a fixed frequency of 50Hz, but what actually matters is the precise pulse width (usually between 1 and 2 milliseconds) — this number encodes the target angular position, not power. The servo's internal circuit compares this pulse width against the actual current position and, in a closed feedback loop, drives its internal motor until these two become equal.

**Full explanation:**

This is one of the most common beginner misunderstandings: because both are called "PWM" and both are generated by a microcontroller, people think their mechanism is the same — completely wrong.

In standard power PWM (Parts 10/6), Duty Cycle is directly proportional to average delivered power — filtering it gives an analog voltage proportional to the percentage of on-time.

In a servo, the frequency is almost always fixed (50Hz), and what changes is the absolute pulse width (not its percentage relative to the whole period) — for example, a 1.5ms pulse means the middle position, a 1ms pulse means one end, a 2ms pulse means the other end.

The servo's internal circuit (which has a small DC motor + a position feedback potentiometer + a simple control circuit) reads this pulse width, compares it against the actual current position (which its internal potentiometer measures), and drives the internal motor until these two become equal — a complete closed feedback loop, not an open signal like standard power PWM.

---

## Question 14: Why does a stepper motor usually operate without feedback (open-loop), while a servo needs internal feedback?

**Short answer:** Because each step of a stepper motor is exactly a specific, pre-known angle — if the driver knows how many steps it has sent, it mathematically knows where the rotor is, with no need for actual measurement. A servo, in contrast, has no such mathematical guarantee at all; to ensure it has reached the requested position, it must directly measure and compare the actual position.

**Full explanation:**

By physical design, a stepper motor's each step (say, 1.8 degrees) is a fixed and precisely engineered value — if you're confident no step has been lost (recall Question 12: correctly setting the phase current is one of the prerequisites for exactly this confidence), simply by counting the number of Step pulses you've sent, you know exactly where the rotor is — no added feedback sensor is needed (unless you want more confidence with an added Encoder, recall Part 11).

A servo has a different structure — a standard DC motor whose rotation speed and direction aren't inherently precisely predictable (depending on load, friction, instantaneous voltage). To make sure it has exactly reached the requested position, the only way is to directly measure the actual position (with the internal potentiometer) and compare it against the target — this is exactly what distinguishes a servo from a stepper.

---

## Question 15: Why does a BLDC (brushless DC motor) need an ESC that a standard DC motor doesn't need?

**Short answer:** A standard DC motor uses mechanical brushes to automatically switch the direction of current in the rotating windings. BLDC eliminates this mechanical switching and instead needs electronic switching — an external controller (ESC) must always know exactly where the rotor is, in order to energize the correct phase at the correct moment; a job that in a standard motor the brushes themselves did mechanically and automatically.

**Full explanation:**

A standard DC motor uses mechanical brushes to switch the direction of current in the rotating windings — these brushes wear down over time and spark (a source of wear and reduced efficiency).

BLDC eliminates this mechanical current switching and instead uses electronic commutation. But this means an external electronic brain now has to do a job that simple mechanics used to do: the ESC (Electronic Speed Controller, essentially a more advanced three-phase H-Bridge) must always know exactly where the rotor is (often with Hall-effect sensors, recall Part 11) in order to energize the correct phase at the correct moment.

This added complexity comes at the cost of higher efficiency, longer lifespan (no brush wear), and more precise control — exactly why it's popular in drones and applications with critical efficiency.

---

## Question 16: Why does the PWM frequency for motor speed control affect both audible sound and efficiency? Where's the practical balance point for most small DC motors?

**Short answer:** When current is switched at the PWM frequency, the windings inside the motor can produce physical mechanical vibration at that same frequency; if this frequency is within the human hearing range, you hear a whining sound — the solution is a frequency above 20kHz. But every MOSFET switch also has some switching losses that increase with frequency — the balance point is usually around 20-30kHz.

**Full explanation:**

**Effect on sound:** when current is switched at the PWM frequency, the windings inside the motor can produce some physical mechanical vibration at that same frequency (electromagnetic force that changes slightly with each switch). If this frequency is within the human hearing range (roughly 20Hz to 20kHz), you hear this vibration as a whining sound. Practical solution: choosing a PWM frequency above 20kHz (ultrasonic, outside human hearing) — the motor still does exactly the same job, you just no longer hear it.

**Effect on efficiency:** recall the switching losses discussion from Parts 5 and 2 (power electronics): every time a MOSFET switches (on/off), some energy is wasted as switching losses. The higher the PWM frequency, the more switches per second, the more switching losses.

This creates a practical trade-off: the frequency must be high enough not to be heard (above 20kHz), but not so high that it needlessly sacrifices efficiency — usually something around 20-30kHz is a good balance point for most small DC motors.

---

## Question 17: Explain the complete circuit for driving a simple DC pump with a MOSFET, step by step — this is exactly the architecture needed for your own project's plant pump.

**Short answer:** The GPIO connects through a small gate resistor (a few tens of ohms) to the gate of a MOSFET (preferably Logic-Level); the MOSFET's drain connects to the pump, and the pump connects to VCC; the MOSFET's source goes to GND; and a flyback diode is placed in parallel with the pump (cathode toward VCC).

**Full explanation:**

This circuit is exactly a combination of several concepts you learned separately in previous sections:

GPIO → gate resistor (a few tens of ohms, limiting the instantaneous gate-capacitor charging current, recall Part 6) → MOSFET gate (preferably Logic-Level so it fully enters saturation with 3.3V, recall Part 3).

MOSFET drain → pump → VCC (Low-side arrangement, recall Part 3 — the pump is always connected to the positive supply, the MOSFET controls the return path to GND).

Flyback diode: in parallel with the pump, with the correct orientation (cathode toward VCC) — exactly the same protection we saw for the relay coil (Part 3), here for the pump motor's winding.

MOSFET source → GND (completing the current path).

This architecture is exactly what we explain in the next question about why it's necessary — not optional — even for a "simple DC" load.

**Formula / Calculation:**
```
GPIO → R_gate → Gate(MOSFET)
Drain(MOSFET) → Pump → VCC
Flyback Diode ‖ Pump (cathode → VCC)
Source(MOSFET) → GND
```

---

## Question 18: Why does even a simple DC pump, controlled by a MOSFET (not a relay), still need a flyback diode? Many people forget this point.

**Short answer:** Because every simple DC pump or fan is a motor with a winding — meaning its inductive nature (recall Part 2) has exactly the same "flywheel" behavior we saw for the relay coil. This isn't limited to relays: any load that has a winding (motor, pump, fan, solenoid, relay) needs this same protection.

**Full explanation:**

This is exactly the same common mistake mentioned in this section's checklist: assuming "a flyback diode is only needed for a relay," because it's usually first learned alongside the relay example.

But the actual physics says something else: the flyback diode has nothing to do with what's switching (relay, transistor, MOSFET) — it depends entirely on whether the load itself is inductive or not. A DC pump or fan, exactly like a relay coil, is built from a winding — so it stores exactly the same magnetic energy and produces exactly the same voltage spike upon a sudden current interruption.

If a MOSFET (instead of a relay) switches this pump and has no flyback diode, exactly the same thing that happened to a transistor without a flyback on a relay coil (Part 3) happens here too for the MOSFET — a voltage spike can destroy the MOSFET at that very first moment of turn-off.

---

## Question 19: What is Back-EMF, and why can a motor's startup or stall current be 5 to 10 times (or more) its normal running current?

**Short answer:** When a DC motor spins, it itself acts like a small generator and produces a voltage in the direction opposite to the supply voltage (Back-EMF), which naturally keeps the current limited during normal running. At the moment of startup, or when the motor mechanically jams (Stall), there's no rotation at all, so there's no Back-EMF either — the only thing limiting current is the winding's very small DC resistance.

**Full explanation:**

When a DC motor spins, it itself (because of the same electromagnetic principle DC motors work on, reversed) acts like a small generator and produces a voltage in the direction opposite to the supply voltage you're feeding it. The faster the motor spins, the larger this Back-EMF becomes, and the more it "opposes" the supply voltage — this is exactly what naturally keeps current limited during normal running.

At the very first moment of turn-on, the motor isn't spinning yet — so there's no Back-EMF at all to hold back the current! The only thing limiting current at that moment is the motor winding's pure DC resistance (recall Ohm's law, Part 1), which is usually a very small number (a fraction of an ohm to a few ohms). Result: the startup current (or when the motor mechanically jams/locks and no longer spins, Stall) can be 5 to 10 times (or even more) the normal running current.

**Formula / Calculation:**
```
I_stall = V_supply / R_winding   (with no opposing Back-EMF at all)
I_running ≈ lower, because Back-EMF cancels out part of the effective voltage
```

**🎯 Interview tip:** This is exactly what this section's practical exercise suggests: measure your pump motor's winding resistance with an ohmmeter (while it's off), and calculate the approximate stall current with this formula — then compare it against your relay contact's or MOSFET driver's rated current.

---

## Question 20: Final design scenario: your pump sometimes jams (Stalls) due to sediment or mechanical freezing. What components need to be chosen to withstand this worst-case current without being damaged?

**Short answer:** Every component responsible for switching the motor's current — whether a relay contact, a MOSFET, or an H-Bridge — must be able to withstand the worst-case stall current (not just the normal running current), giving protective circuits (fuse, PTC, current limiter) the chance to step in.

**Full explanation:**

This question is exactly a combination of several concepts you learned in this section and previous ones: Back-EMF and stall current calculation (Question 19), relay contact specs (Question 2), and circuit protection principles from Part 5.

Practical design rule: if your pump mechanically jams for any reason (say, sediment or freezing in cold weather), this is exactly the moment when current reaches the I_stall value (which can be 5 to 10 times normal) and may continue for a fairly long time (until someone notices or protection kicks in).

Whatever component directly switches or passes this current — the relay contact (if you're controlling with a relay), or the MOSFET itself (if you're controlling directly with a MOSFET) — must, per its datasheet, be able to withstand this worst-case current without damage, not just the normal running current you measured under typical conditions.

In addition, a separate protection circuit (fuse, PTC, or an active current limiter, full recall from Part 5) must also be in the path, so that if this Stall condition continues for a long time (meaning it's genuinely a persistent mechanical fault, not a transient moment), it cuts off the current before it damages the driver or wiring.

**🎯 Interview tip:** This is exactly the kind of "end-to-end design" question asked in higher-level interviews — a strong answer should show that worst-case current calculation, choosing a switching component matched to it, and an independent protection layer are all designed together and coordinated, not separately.

---
