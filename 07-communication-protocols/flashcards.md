# Interview Flashcards — 🔧 Part 7 | Communication Protocols and Buses

A set of 20 questions and detailed answers on communication protocols and buses, for hardware/embedded technical interview prep.

---

## Question 1: What is a differential signal, and why is it resistant to common-mode noise?

**Short answer:** In a differential signal, information is sent over two wires — one carries the actual signal, the other its exact inverse. Instead of measuring each wire relative to GND, the receiver only measures the difference between these two wires; noise that lands equally on both wires gets automatically cancelled out in this very "taking the difference" process.

**Full explanation:**

In a single-ended signal — like UART, I2C, standard SPI — voltage on one wire is measured relative to a shared GND. The problem: if noise (say, from a nearby motor) lands on that same wire, or even on the GND itself, the receiver can't tell whether that voltage change is real signal or noise — both look exactly the same.

Analogy: imagine two people standing next to each other in a loud, crowded market, speaking at the same time — one says "yes," the other simultaneously says its exact opposite, "no." A third listener, instead of trying to hear just one person (whose voice gets mixed with the market noise), measures the difference between what these two people say. Because the market noise affects both of them almost equally (they're standing right next to each other), when the listener computes their difference, that shared noise cancels out of the calculation, leaving only the real signal.

This is exactly the physical logic of a differential signal — the shared foundation of CAN, RS-485, USB, and Ethernet.

---

## Question 2: How does a twisted pair work? Why are the wires twisted together?

**Short answer:** Twisting two wires together has two useful effects: external noise gets distributed more evenly across both wires (since both are almost always at nearly the same close distance from the noise source), and the magnetic field that these two wires themselves generate (with opposing currents) largely cancels itself out.

**Full explanation:**

The first effect helps the quality of differential signal reception: the more evenly (symmetrically) external noise lands on both wires, the better the "taking the difference" process removes it — and twisting the two wires together guarantees exactly this symmetry, because both wires are always at nearly the same distance from any external noise source.

The second effect is the reverse: the twisted pair itself can also be a source of interference for others, because current goes out on one wire and returns on the other. When you twist these two wires together, the magnetic fields resulting from these two opposing currents (each in the opposite direction of the other) nearly cancel each other out at short distances — meaning they both pick up less noise from outside, and radiate less noise (less EMI) to the outside.

This is exactly the physics you see in Ethernet network cable (four twisted pairs), CAN cable, and USB cable.

---

## Question 3: Why is UART an "asynchronous" protocol? How does this feature relate directly to the importance of crystal accuracy (ppm)?

**Short answer:** Because UART has no shared clock wire at all between transmitter and receiver; each side has its own internal clock and must simply guess, based on an agreed-upon rate (Baud Rate), "when exactly it should read the next bit." If the two sides' clocks differ even slightly, this error accumulates over the course of a multi-bit frame — exactly the same crystal ppm problem we saw in Part 2.

**Full explanation:**

Unlike SPI, which has a shared clock wire (SCK), UART has no wire at all to precisely synchronize the two sides. Each side simply counts its own internal clock (which comes from a crystal), based on the agreed Baud Rate, to guess where the middle of each bit is.

If the receiver's and transmitter's clocks differ even by a few percent, this small difference accumulates over the course of an 8–10 bit frame — to the point where the receiver might read the last bit at the wrong point. This is exactly why crystal frequency tolerance (ppm, Part 2) matters for serial communication.

At higher Baud Rates (say, 115200 versus 9600), this sensitivity to clock error becomes even greater, because the time window per bit is shorter — meaning that same amount of ppm error becomes a bigger share of a smaller time window.

---

## Question 4: Explain the exact structure of a UART frame — what exactly is the role of each section?

**Short answer:** Each byte within a frame is sent in this order: a Start Bit (a sudden drop from HIGH to LOW that announces the start of the frame), 8 data bits, an optional Parity Bit (simple error detection), and a Stop Bit (a return to HIGH indicating the end of the frame).

**Full explanation:**

**Start Bit:** the line, which is usually HIGH at rest, suddenly goes LOW — this sudden drop tells the receiver "a new byte is coming, start counting your clock from this exact moment."

**Data bits:** usually 8 (sometimes 5 to 9, depending on the settings).

**Parity Bit (optional):** a simple error-detection bit — it counts the number of 1 bits in the data and sends an extra bit so the total becomes even (Even) or odd (Odd); if the receiver doesn't find this balance after receiving, it knows a bit got corrupted somewhere — a very basic error-detection method, not error correction (you only find out an error happened, you can't fix it yourself).

**Stop Bit:** the line goes HIGH again, signaling the end of the frame and readiness for the next one.

---

## Question 5: What's the difference between Hardware Flow Control (RTS/CTS) and Software Flow Control (XON/XOFF) in UART?

**Short answer:** Hardware Flow Control uses two extra wires (RTS/CTS) to signal "ready/not ready" — fast and reliable, but needs two extra GPIO pins. Software Flow Control uses special bytes (XON/XOFF) within the same normal data stream — no extra wire, but slower and prone to interference with binary data.

**Full explanation:**

These two methods solve a shared problem: when the transmitter is faster than the receiver can process the bytes (the receiver's buffer is filling up).

**Hardware Flow Control (RTS/CTS):** the receiver uses the CTS (Clear To Send) pin to tell the transmitter "you're allowed to send now or not," and the transmitter signals readiness with RTS (Request To Send). Fast and reliable, but it costs two extra GPIO pins per connection.

**Software Flow Control (XON/XOFF):** no extra wire — when the receiver's buffer fills up, it sends a special byte (XOFF) within the same normal data stream to tell the transmitter to pause temporarily; when space frees up again, it sends the XON byte. Simpler wiring, but slower, and it can conflict with binary data that happens to contain the same XON/XOFF byte pattern.

---

## Question 6: Why is SPI usually faster than I2C? Give at least three separate reasons.

**Short answer:** Three main reasons: (1) SPI has a dedicated data wire for each direction (MOSI/MISO), unlike I2C, which has a shared wire where only one direction is possible at any given moment. (2) SPI has no addressing overhead in each byte, while I2C has to send the device address every time. (3) SPI has no extra ACK/NACK bit after each byte.

**Full explanation:**

**First reason — genuine full-duplex:** because MOSI and MISO are two completely separate wires, data can flow in both directions simultaneously. I2C has only one data wire (SDA), so only one direction is possible at any given moment (half-duplex).

**Second reason — no repeated addressing overhead:** in I2C, the master must send the target device's address at the start of every transaction. In SPI, device selection is done with a separate physical pin (CS), not by sending an address byte over the same data line.

**Third reason — no ACK/NACK:** I2C waits for an acknowledgment bit (ACK) from the receiver after every byte; SPI has no such protocol overhead.

The combined result of these three differences: SPI can usually reach tens of megahertz, while I2C is usually limited to a few hundred kilohertz up to a bit more.

---

## Question 7: What exactly do the four SPI clock modes (the CPOL/CPHA combination) determine? Why does a mode mismatch between master and slave produce "seemingly random" data, rather than an obvious error?

**Short answer:** CPOL determines whether the clock is HIGH or LOW at rest; CPHA determines whether data is read on the clock's first edge or its second edge. If the master and slave have different modes, both think they're working correctly, but they read data at exactly the wrong moments — the result is completely meaningless, seemingly random bytes, not a specific error message.

**Full explanation:**

**CPOL (Clock Polarity):** is the clock HIGH when nothing is being transferred (CPOL=1), or LOW (CPOL=0)?

**CPHA (Clock Phase):** is data read on the clock's first edge (CPHA=0), or its second edge (CPHA=1)?

Combining these two produces 4 modes (0,0 / 0,1 / 1,0 / 1,1). No protocol tells either side "if you chose the wrong mode, I'll give you an error" — because SPI has no explicit error-detection mechanism at all (unlike ACK in I2C). Both sides are completely "confident" they're reading the correct edges, but because their definition of "which edge is correct" differs, each ends up reading the data exactly one edge earlier or later than the correct moment — the result is bytes with no recognizable pattern at all.

---

## Question 8: Why do you need Tri-State (High-Z) when several slaves are on a shared SPI bus? How does this differ from the similar mechanism in I2C?

**Short answer:** Because SPI uses Push-Pull output (not Open-Drain like I2C), if several slaves simultaneously try to actively drive the shared MISO line, exactly the same Bus Contention (Part 6) occurs — two Push-Pull outputs each giving a different value means a genuine short circuit. Solution: only the device whose CS is active drives MISO; the rest keep their outputs in Tri-State.

**Full explanation:**

Recall from Part 6: a Push-Pull output always actively gives either HIGH or LOW, never "releases." If two or more SPI slaves simultaneously try to give different values on that same shared MISO wire, this is exactly the same Bus Contention risk we saw in Part 6 — it can lead to physically burning out a pin.

SPI's solution differs from I2C's: because I2C was designed from the start as Open-Drain, it doesn't have this problem at all (only pulling down is possible, no one actively drives HIGH). But because SPI is Push-Pull, it must be explicitly managed: at any given moment, only the device whose CS pin has been activated by the master is allowed to drive the MISO line; the rest of the devices must keep their outputs in the Tri-State (High-Z, meaning effectively disconnected from the line) state — exactly the concept we saw fully in Part 6.

---

## Question 9: What problem does Repeated Start in I2C solve that a full Stop followed by a new Start can't?

**Short answer:** When the master wants to perform a "write, then immediately read" operation on the same device (like "write the register address, then read its value"), Repeated Start performs these two operations without ever fully releasing the bus between them — and this guarantees that no other master grabs the bus in between these two operations.

**Full explanation:**

Suppose the master wants to perform a two-stage transaction on the same device: first write the address of an internal register (say, "I want to read the temperature register"), then immediately read that register's value.

If the master sends a full Stop and then a completely new Start, the bus is declared "free" in the gap between these two — meaning there's a real risk that another master (in a multi-master system) grabs exactly this gap and takes over the bus for its own work, disrupting our "write-read" transaction.

Repeated Start is exactly like a normal Start, but without the bus ever actually being declared "free" — this guarantees that the two operations (writing the register address + reading its value) happen as a single, uninterrupted atomic transaction.

---

## Question 10: Exactly how does I2C multi-master arbitration work without a separate central arbiter? Explain the complete physical mechanism.

**Short answer:** While each master is sending its bits, it also reads the bus itself and compares it with what it sent. If a master sent bit 1 (released the line) but, upon reading the bus, sees the line is actually 0 (because another master sent 0 at that same moment), it realizes it has "lost" and immediately backs off — this is exactly the same Wired-AND mechanism from Part 6.

**Full explanation:**

Suppose two masters on the same I2C bus start sending at the same time. Recall from Part 6: on an Open-Drain line, if even one device pulls it down, the entire line goes down (Wired-AND) — I2C uses exactly this property to resolve multi-master conflicts, without needing a separate central arbiter.

While each master is sending its bits, it also reads the actual value on the bus and compares it with what it itself sent. If a master sent bit 1 (meaning it released the line, since in Open-Drain releasing means hoping for HIGH) but, upon reading the bus, sees the line is actually 0 (because another master sent 0 at that same moment and pulled it down), this master realizes it has "lost" — it immediately backs off and lets the master that sent the higher-priority bit (0) continue.

This is a completely decentralized, automatic arbitration mechanism that results directly from Open-Drain/Wired-AND physics — with no central arbiter or extra coordination message at all.

---

## Question 11: Why is I2C usually limited to short distances (on a single board), while CAN and RS-485 can go long distances (meters to kilometers)?

**Short answer:** Because I2C is single-ended and relies on a pull-up resistor plus the line's parasitic capacitance, which gets worse with bus length (a larger RC time constant, a slower signal edge); CAN and RS-485 are differential, which are inherently more noise-resistant and designed for long distances.

**Full explanation:**

Recall from Part 6: a larger pull-up combined with the line's parasitic capacitance creates an unwanted RC filter that slows down the signal edge. On a long I2C bus, the parasitic capacitance of the whole wire (which increases with bus length) makes exactly this effect worse — which is why I2C is usually limited to short distances (on a single board, or at most a few tens of centimeters with a suitable pull-up).

CAN and RS-485 were designed from the start for long distance: differential signaling (Question 1) is inherently more resistant to noise that accumulates on a wire over long distances, and these two protocols are designed with the physical path (a specific cable, proper termination) that preserves exactly this resistance over long distances — RS-485 up to about 1200 meters (at low speed).

For longer distances, you should always go with RS-485 or CAN, not try to "stretch" I2C/SPI beyond the distance they were originally designed for.

---

## Question 12: How can 1-Wire even "steal" its power from that same single data wire (Parasitic Power)?

**Short answer:** The device has a small internal capacitor that charges up when the line is HIGH at rest; that same energy stored in this capacitor lets the device keep working during the moments the line goes LOW for data transfer (when the device can no longer draw energy directly from the line).

**Full explanation:**

1-Wire (whose most famous application is the DS18B20) works with only a single data wire (plus GND) — sometimes even without a separate power wire.

The trick is this: the device has a small internal capacitor (recall capacitor physics from Part 2) that charges up whenever the data line is HIGH at rest. When the microcontroller temporarily pulls the line LOW to transfer data, the device can no longer draw energy directly from the line — but that same energy stored in its internal capacitor lets it keep working during these brief moments.

This is a beautiful example of creatively using a basic concept (energy storage in a capacitor) to solve a completely different problem (reducing the number of wires needed).

---

## Question 13: What exactly do "Dominant" and "Recessive" bits mean in CAN? How does this concept enable priority-based arbitration without any arbiter at all?

**Short answer:** In the recessive state, both the CANH and CANL wires return to a shared middle voltage (near-zero difference). In the dominant state, the transmitter actively pulls CANH high and CANL low. If even one device sends a dominant bit, the entire bus becomes dominant — exactly the same Wired-AND logic we saw in I2C, but this time with differential voltage instead of a simple digital level.

**Full explanation:**

Instead of normal HIGH/LOW, CAN uses these two concepts, and this naming difference has a deep physical reason: in the recessive state, no device actively drives the line — the termination resistors (120Ω) return both wires to a shared middle voltage. In the dominant state, the transmitter actively pulls CANH high and CANL low, creating a clear difference.

Every CAN message has a numeric identifier; a smaller identifier has higher priority. When several devices start sending at the same time, exactly like the I2C mechanism (Question 10), each one reads the bus while it's sending; the moment a device sees that the recessive bit it sent has actually become dominant on the bus (meaning another device sent a dominant bit), it silently backs off.

Result: the message with the highest priority (lowest identifier) always wins without delay and without any real collision — an extremely elegant arbitration system that results entirely from differential + dominant/recessive physics, with no separate central arbiter at all.

---

## Question 14: Why does CAN Bus need 120Ω termination resistors at both ends of the bus, while I2C and SPI don't need this resistor at all?

**Short answer:** Because CAN usually operates over fairly long cables and at fairly high speeds — conditions where the wire effectively becomes a transmission line that reflects the signal if not terminated with a specific impedance. I2C/SPI operate over very short distances, where potential reflections die out on their own before they actually reach the next bit.

**Full explanation:**

When the signal's speed (its edge transition) is "fast enough" relative to the physical length of the wire, the wire is no longer a simple, property-free conductor — it becomes a transmission line whose behavior resembles high-frequency (RF) circuits. If the end of this line isn't terminated with an impedance matching the wire's own characteristic impedance, part of the signal's energy reflects and travels back — this reflection interferes with subsequent signals and corrupts bits.

CAN and RS-485 operate over long distances (meters to kilometers) and often at fairly high speeds — exactly the conditions where "the wire behaves as a transmission line." A 120Ω termination resistor (which matches the characteristic impedance of standard CAN cable) absorbs this energy instead of reflecting it.

I2C and SPI usually operate over a distance of a few centimeters on a single board, where the signal's round-trip time is so short that any potential reflections have already died out on their own before they actually reach the next bit — which is why you don't see this problem in practice, not because the physics doesn't exist.

---

## Question 15: What role do the DE (Driver Enable) and RE (Receiver Enable) pins play in an RS-485 transceiver (like the MAX485)? What exactly happens if managing them is forgotten?

**Short answer:** Most RS-485 implementations are half-duplex — only one device can transmit at any given moment. The software must explicitly enable DE before transmitting (transmitter mode), and immediately disable it again right after transmission finishes (returning to receiver mode). Forgetting to do this means the device stays permanently in transmitter mode and never hears the other side's response.

**Full explanation:**

Unlike SPI, which is full-duplex (it can transmit and receive simultaneously), most RS-485 implementations, over that same shared pair of wires, allow only one direction of transfer at any given moment (half-duplex). The RS-485 transceiver chip has these two extra control pins exactly for this reason: DE and RE.

The software must follow a precise sequence: before sending data, enable DE (the chip enters "transmitter" mode and can drive the line); immediately after transmission fully finishes (not earlier, or part of the data gets cut off), disable DE again so the chip returns to "receiver" mode and can hear the other side's response.

If this is forgotten, the device gets permanently stuck in "transmitter" mode and never hears anything from the bus — or even worse, if two devices simultaneously keep DE enabled, exactly that same Bus Contention (Part 6) occurs.

**🎯 Interview tip:** This question specifically tests a common practical bug that many newcomers first run into with RS-485 — showing awareness of this detail demonstrates hands-on experience, not just theoretical knowledge.

---

## Question 16: What's the main difference between RS-232 and RS-485? Why do you see physical RS-232 less often today?

**Short answer:** RS-232 is single-ended and point-to-point (only one transmitter, one receiver), with high voltage levels (±12V) for better noise margin at moderate distance. RS-485 is differential and multi-drop (several devices on the same shared pair of wires), for much longer distances (up to 1200 meters). You see RS-232 less often today because USB has replaced it.

**Full explanation:**

RS-232: the older version of point-to-point serial communication — the same UART logic, but with much higher voltage levels (traditionally usually ±12V or ±15V, instead of TTL logic's 3.3V/5V) so it has a better noise margin over fairly longer cables (recall Part 6). Physical RS-232 ports (the well-known DB9 connector) have been almost entirely removed from modern computers, replaced by USB.

A practical point that still applies: if you want to connect a microcontroller (with a 3.3V-level UART) to an old industrial device with a genuine RS-232 port, you need a level-conversion IC (like the well-known MAX232 family) that converts the ±12V levels to 3.3V/5V TTL — another example of Level Shifting (Part 6), just between two completely different standards.

RS-485: it's differential (like CAN) but only specifies the physical layer, not a complete protocol with framing and arbitration (you design the layer above it yourself, or use Modbus). It's multi-drop — several devices (sometimes up to 32 or more) can sit on that same shared pair of wires, for much longer distances (up to around 1200 meters).

---

## Question 17: What exactly is the Enumeration process in USB, and why is this process exactly what makes "Plug and Play" possible?

**Short answer:** When you plug in a USB device, before any actual data transfer, the device sends a small "descriptor" of itself (device type, manufacturer, what capabilities it has) to the host. The host reads this and picks the appropriate driver/behavior on its own — this is exactly why the user doesn't need to manually install a driver or manually configure anything.

**Full explanation:**

This automatic process is exactly what makes the "just plug it in and it works" experience possible — the host knows nothing in advance about that specific device, but the device itself "introduces" itself at the moment of connection.

This introduction protocol happens over that same D+/D- differential pair (Question 1), which is later also used for actual data transfer. In addition to D+/D- and GND, there's also a fourth wire (VBUS) that usually delivers 5V from the host to the device — the same thing many ESP32 development boards use for power (recall the USB Power Delivery discussion from Part 4).

Common embedded use: flashing firmware, a virtual serial port (Virtual COM Port, which actually just emulates that same UART over USB), and powering the board.

---

## Question 18: What's the difference between MII and RMII in the interface between the MAC and PHY in Ethernet? Why is RMII more common in the embedded world?

**Short answer:** MII is the older, more complete interface but needs about 16 pins; RMII is a reduced version that does the same job over about 7–9 pins, at a higher frequency. RMII is more common because embedded microcontrollers have limited GPIO pins, and saving on pin count is exactly the same pin budgeting concept from Part 6.

**Full explanation:**

In an Ethernet implementation, two parts are logically separate: the MAC (Media Access Control) — the protocol logic (framing, addressing), which is usually inside the microcontroller itself; and the PHY (Physical Layer) — a separate chip that actually sends/receives the electrical signal over the copper cable. These two are connected to each other by a standard interface.

MII (Media Independent Interface) is the older, more complete interface, but it needs a large number of pins (about 16). RMII (Reduced MII) is a reduced version that does the same job over fewer pins (about 7–9), at a higher frequency.

For embedded microcontrollers with limited GPIO pins (recall the pin budget from Part 6), this pin savings is very valuable — a few fewer pins for the Ethernet interface means a few more pins for the project's actual peripherals. That's why RMII is far more common in the embedded world than MII.

---

## Question 19: Why is SWD more popular than JTAG in modern microcontrollers (especially ARM Cortex-based ones)?

**Short answer:** Because SWD provides the same debug and programming capability with only 2 wires (SWDIO and SWCLK), while JTAG needs 4 to 5 wires. In modern embedded chips whose physical package often has limited pins, saving 2 to 3 pins for debugging means more pins for the project's actual peripherals.

**Full explanation:**

JTAG was originally designed for "Boundary Scan" (testing the connections of a chip's pins on a PCB after assembly, without a physical probe on every pin), but today it's used mostly for debugging and programming. It usually needs 4 to 5 wires: TDI, TDO, TCK, TMS, and sometimes TRST — and it's a general industry standard (not specific to any particular architecture).

SWD (Serial Wire Debug) is ARM's own proprietary standard that provides the same debug capability with only 2 wires (SWDIO and SWCLK), but it doesn't have Boundary Scan capability.

The reason SWD is popular in modern embedded chips is exactly the same pin-budgeting logic we saw in Part 6: these chips' physical packages often have limited pins; saving 2 to 3 pins for debugging means more pins for the project's actual peripherals — a completely practical trade-off, not just an arbitrary technical choice.

---

## Question 20: Protocol-selection scenario: for an industrial project with long distance, an electrically noisy environment, and a need to prioritize a critical message (like a safety alarm), which would you choose among UART/I2C/SPI/CAN/RS-485, and why?

**Short answer:** CAN is the best choice — because it's both differential and resistant to industrial noise, it covers long distance, and most importantly, its priority-based arbitration mechanism (dominant/recessive bits) inherently provides exactly that "critical message prioritization," without needing any extra software logic.

**Full explanation:**

Putting the requirements together: long distance and a noisy environment immediately rule out UART, I2C, and SPI (these three are single-ended and/or designed for short distance).

Between the two remaining options (CAN and RS-485), the deciding factor is exactly the need for "critical message prioritization": RS-485 only specifies the physical layer and has no internal arbitration or prioritization mechanism at all — if you want message prioritization, you have to implement it yourself in the layer above (software).

CAN, however, provides this feature for free and guaranteed, at its own physical and protocol level: every message has a numeric identifier, and the smaller identifier always wins without delay and without any real collision (Question 13). This means a critical safety alarm message, even if the bus happens to be busy at that exact moment, is guaranteed to get through immediately and without delay — exactly what the scenario requires. That's why CAN, not just physically but architecturally at the protocol level, is the best match for this specific scenario.

**🎯 Interview tip:** This kind of "design scenario" question is one of the most common higher-level interview formats — a strong answer should show that you made the decision based on systematically comparing requirements against each protocol's actual features, not just memorizing "CAN is good for industrial."

---
