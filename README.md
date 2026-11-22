# 64-Vehicle Smart Parking Lot Control System (FSM & Memory Architecture)

A modular combinational and sequential digital logic system designed to manage a 64-vehicle parking lot, developed as part of the **EE142 Digital Logic Design** course at Boğaziçi University. Built in **Logisim**, the architecture integrates a 10-state Finite State Machine (FSM), a 64×1-bit RAM hardware search engine, and a zero-delay ROM-based Binary-to-BCD display driver.

---

## System Overview

* **Inputs:**
  * `Ce`: Vehicle entry request
  * `Cx`: Vehicle exit request
  * `Reset`: Asynchronous daily service counter reset
  * `8-bit BCD Address`: Target parking space address for exiting vehicles
* **Outputs:**
  * `G`: Entry/exit gate control signal
  * `F`: Capacity full flag (active at 64 vehicles)
  * `7-Segment Displays`: Real-time current occupancy and cumulative daily service counts

---

## Modular Hardware Architecture

The system is synthesized into **five primary sub-systems** adhering to modularity and minimal-component design principles:

1. **Finite State Machine (Control Unit):**
   * Implements a **10-state FSM** using a **State Decoder architecture** (4-bit state register + 4-to-16 decoder yielding active-high state lines `Y0–Y9`) to eliminate massive 9-variable Karnaugh maps.
   * Control signals are generated directly via OR gates (e.g., `WE = Y5 + Y9`, `G = Y1 + Y2 + Y6 + Y7`, `Sen = Y4`).
2. **Memory & Hardware Search Unit:**
   * Models the 64-space capacity using a **64×1-bit RAM** block.
   * Uses a 6-bit counter to sequentially scan for the first vacant spot (`RAM data = 0`), locking the counter immediately upon detection (`Vf = 1`).
   * Employs a 2-to-1 MUX with hardware address locking (`Sen + RAMWE`) to route search addresses during entry and external BCD addresses during exit.
3. **Timer Module:**
   * A 4-bit counter paired with a comparator keeps the gate open for exactly **5 clock cycles** (`Tdone = 1`) and auto-resets via `NOT(Ten)` upon exiting the wait state.
4. **Occupancy Counter & ROM-Based Display:**
   * A 7-bit Up/Down counter tracks current occupancy (enabled by `Cup + Cdown`, direction controlled by `Cup`) and triggers the Full Flag (`F = 1`) at `0x40` (64).
   * Uses a **ROM Look-up Table (0x00–0x40)** for zero-delay Binary-to-BCD conversion to drive the 7-segment displays.
5. **Daily Service Counter:**
   * Three cascaded 4-bit BCD counters (max value `0x9`) track total daily entries with an independent asynchronous reset.

*(Add your exported Logisim schematics below)*
![Main Circuit Schematic](<img width="1273" height="733" alt="image" src="https://github.com/user-attachments/assets/c2350218-2e82-46f8-af56-e7837a02242b" />
)

---

## Testing, Verification & Root Cause Analysis

Module-level manual tests and full-system clock-driven simulations were conducted in Logisim:
* **Verified Stable Modules (100%):** Vehicle exit operations (RAM deletion), 5-cycle Timer synchronization, Current Occupancy counter, ROM-based BCD display, and Daily Service counter.
* **Known Issue & Root Cause Analysis (Entry Sequence):** During consecutive vehicle entries, a hardware addressing anomaly was identified where the system skips Space 2, writes to Space 3, and loops between Spaces 1 and 3. Hardware debugging traced this to:
  1. *Datapath Desynchronization:* Control signal fluctuation during the `SEARCH` to `UPDATE` transition causes the MUX selector to lose the latched address.
  2. *Counter Stagnation & False-Positive State:* The search counter fails to advance past occupied slots while the FSM still increments the occupancy display (`Cup`).

---

## Repository Structure

```text
├── circuits/     # Logisim (.circ) files
├── report/       # Technical report (PDF & LaTeX source)
├── notes/        # State tables and handwritten design notes
└── images/       # Exported circuit schematics
