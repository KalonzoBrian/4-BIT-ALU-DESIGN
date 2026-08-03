# 4-BIT-ALU-DESIGN

A concise, modular design of a 4-bit Arithmetic Logic Unit (ALU) that supports common arithmetic and bitwise operations and formats its 4-bit result for display on a single hexadecimal 7-segment display.

This repository contains the theoretical design, logic formulation, and detailed Logisim-Evolution gate-level implementation steps for Phase 1 and Phase 2 of the assignment.

---

## Table of contents

- Overview
- Features & Supported Operations
- Inputs and Control Signals
- ALU Architecture
  - Arithmetic block
  - Logic block
  - Shift / Routing
  - Output selection (Multiplexer)
- Phase 2: Gate-Level Simulation (Logisim-Evolution)
  - Arithmetic Unit
  - Logic Unit
  - Multiplexer Routing Unit
  - 7-Segment Decoder
  - Top-Level Integration
- Hexadecimal → 7-Segment Decoder
  - Truth table summary
  - Minimal boolean equations (overview)
- Testing & Simulation
- Implementation & Files (suggested)
- Screenshots
- Notes
- Future extensions
- License & Author

---

## Overview

This project describes a 4-bit ALU that accepts two 4-bit operands A and B and a 3-bit selection code S2 S1 S0 to produce a 4-bit output Y. The output Y is also decoded into the signals (a..g) required by a single 7-segment display so that 0–F (hex) values can be displayed.

The design is modular so each block (arithmetic, logic, shift/routing, multiplexer, decoder) can be implemented and tested independently.

---

## Features & Supported Operations

The ALU supports eight operations (selected by the 3-bit selector S):

Selection Code (S2 S1 S0) → Operation (Y)
- 000 — Y = A + B (Addition)
- 001 — Y = A − B (Subtraction via two's complement)
- 010 — Y = A ⋅ B (Bitwise AND)
- 011 — Y = A + B (Bitwise OR)
- 100 — Y = A ⊕ B (Bitwise XOR)
- 101 — Y = A ≪ 1 (Logical shift-left of A by 1)
- 110 — Y = A (Bypass / routing)
- 111 — Y = B (Bypass / routing)

---

## Inputs and Control Signals

- A[3:0] — 4-bit input operand A (A3 = MSB, A0 = LSB)
- B[3:0] — 4-bit input operand B (B3 = MSB, B0 = LSB)
- S[2:0] — Operation selection (S2 is MSB of select)
- Cout / flags — optional carry/overflow flags can be added for arithmetic operations

Output:
- Y[3:0] — 4-bit ALU result
- seg[6:0] (a,b,c,d,e,f,g) — signals driving a common-cathode 7-segment display (active-high convention assumed)

---

## ALU Architecture

The ALU is split into three cooperating sections: Arithmetic, Logic, and Multiplexer routing.

### 1) Arithmetic block
- A single 4-bit ripple-carry adder is used for both addition and subtraction.
- Addition: feed A and B to the adder with Cin = 0 → Sum = A + B.
- Subtraction: invert B (bitwise NOT) and set Cin = 1 → Sum = A + (~B) + 1 = A − B (two's complement).
- Optional outputs: carry out (Cout) and overflow flag for signed results.

![Arithmetic Unit](screenshots/Arithmetic_Unit.png)
*Figure: Arithmetic_Unit — subtraction implemented via two's complement (invert B + Cin).* 

### 2) Logic block
- Bitwise operations are implemented by four parallel single-bit gates per operation:
  - AND: A ⋅ B (one 2-input AND per bit)
  - OR: A + B (one 2-input OR per bit)
  - XOR: A ⊕ B (one 2-input XOR per bit)
- Logical shift-left (A ≪ 1): implemented by routing:
  - Y0 = 0 (LSB hardwired to 0)
  - Y1 = A0
  - Y2 = A1
  - Y3 = A2
  - A3 is discarded (or can be routed to a flag for extended designs)

![Logic Unit](screenshots/Logic_Unit.png)
*Figure: Logic_Unit — parallel bitwise AND/OR/XOR and logical shift-left of A.*

### 3) Multiplexer routing block
- Four 8-to-1 multiplexers (one for each result bit Y3..Y0).
- S[2:0] is connected to the select lines of all multiplexers.
- The inputs to each MUX are the corresponding bits from the Arithmetic block, Logic block outputs, the shift result, and bypass outputs (A, B), arranged to match the selection table above.

![Routing Unit](screenshots/Routing_Unit.png)
*Figure: Routing_Unit — collects operation outputs and selects the final 4-bit ALU result based on S[2:0].*

---

## Phase 2: Gate-Level Simulation (Logisim-Evolution)

This section provides a step-by-step blueprint to implement the ALU structurally using Logisim-Evolution. Follow these steps to build and test the ALU physically in Logisim-Evolution.

### 1. The Arithmetic Unit

Create a new circuit named `Arithmetic_Unit`. This sub-circuit handles addition and subtraction.

- Place two 4-bit input pins labeled `A` and `B`.
- Place a 1-bit input pin labeled `Subtract`.
- Drop a standard 4-bit Adder component onto the canvas.
- Route input `A` directly into the first operand pin of the Adder.
- To achieve two's complement for subtraction (`A-B`), route input `B` into a 4-bit NOT gate (inverter) to create `\overline{B}`.
- Use a 4-bit 2-to-1 Multiplexer to select between `B` (if `Subtract = 0`) and `\overline{B}` (if `Subtract = 1`). Connect the output of this MUX to the second operand pin of the Adder.
- Wire the `Subtract` pin directly to the Carry-In (`C_in`) of the Adder. This adds the required `1` for two's complement.
- Route the 4-bit sum to an output pin.

(See screenshot for the implemented Arithmetic unit.)

![Arithmetic Unit](screenshots/Arithmetic_Unit.png)

### 2. The Logic Unit

Create a new circuit named `Logic_Unit` to compute bitwise operations.

- Place two 4-bit input pins labeled `A` and `B`.
- Use Logisim's built-in bitwise gates (AND, OR, XOR), ensuring their data bits attribute is set to 4.
- Wire `A` and `B` into the AND gate, the OR gate, and the XOR gate.
- For the Logical Shift Left operation (`A<<1`), use Logisim's "Splitter" component to break `A` into its individual bits. Use another Splitter to recombine the bits: hardwire a constant `0` to bit 0, route input bit 0 to output bit 1, input bit 1 to output bit 2, and input bit 2 to output bit 3.
- Place four 4-bit output pins for each of the logic results.

![Logic Unit](screenshots/Logic_Unit.png)

### 3. The Multiplexer Routing Unit

Create a new circuit named `Routing_Unit`. This routes all results to the final output based on the selection code.

- Place eight 4-bit input pins for all possible operations: Add, Subtract, AND, OR, XOR, Shift, Bypass A, and Bypass B.
- Place a 3-bit input pin for the selection code (`S_2 S_1 S_0`).
- Place a 4-bit, 8-to-1 Multiplexer on the canvas and connect the 3-bit selection code to the select pin of the MUX.
- Wire the eight operational inputs into the MUX data pins matching the mapping table defined in Phase 1 (e.g., pin `000` gets Addition, `010` gets AND).
- Route the MUX output to a single 4-bit output pin.

![Routing Unit](screenshots/Routing_Unit.png)

### 4. The 7-Segment Decoder

Create a new circuit named `Hex_Decoder`.

- Place a 4-bit input pin (the result from the ALU).
- Use a Splitter to break the 4-bit input into four individual 1-bit lines (`D_3, D_2, D_1, D_0`).
- Using Logisim's basic AND, OR, and NOT gates, build the combinational logic for segments `a` through `g` using the exact minimal Boolean equations derived via K-maps in Phase 1.
- Alternatively, use Logisim-Evolution's "Combinational Analysis" tool: input the truth table from Phase 1 and let it auto-generate the minimized structural gates.
- Place seven 1-bit output pins labeled `a` through `g`.

![Hex Decoder](screenshots/Hex_Decoder.png)
*Figure: Hex_Decoder — sum-of-products implementation for segments a..g (or generated via combinational analysis).* 

### 5. Top-Level Integration

Open the `main` circuit canvas to combine everything into a single top-level circuit.

- Drag and drop the `Arithmetic_Unit`, `Logic_Unit`, `Routing_Unit`, and `Hex_Decoder` blocks onto the main canvas.
- Create master input pins: 4-bit `A`, 4-bit `B`, and 3-bit Operation Select.
- Route `A` and `B` to the inputs of both the Arithmetic and Logic blocks.
- Split the 3-bit Operation Select line and route its Least Significant Bit (`S_0`) to the `Subtract` pin of the Arithmetic block (since subtraction is code `001`).
- Route the outputs of the Arithmetic and Logic blocks into their respective inputs on the `Routing_Unit`.
- Route the master 3-bit Operation Select to the select pin of the `Routing_Unit`.
- Connect the output of the `Routing_Unit` to the input of the `Hex_Decoder`.
- Place a 7-Segment Display component on the canvas and wire the seven output pins of the `Hex_Decoder` to the corresponding pins on the display component.
- Use Logisim's "Poke" tool to toggle the inputs and demonstrate that the correct hex digit illuminates for each operation.

![Top Level](screenshots/Top_Level.png)

---

## Hexadecimal → 7-Segment Decoder

The ALU result Y[3:0] (denoted D3..D0) must be converted to 7 signals (a..g) to display hex digits 0–9 and A–F.

- Display conventions used:
  - Active-high segments (common-cathode).
  - Hex 'B' and 'D' are represented as lowercase 'b' and 'd' to avoid ambiguity (i.e., conventional 7-seg patterns for lowercase forms).
- The full truth table maps D3..D0 → segments (a..g); implementers can use either:
  - The provided truth table and derive logic gates.
  - A small ROM (16x7) / LUT for compact hardware.
  - Minimised boolean equations (below) or simple logic synthesis.

The Truth Table (expanded in design notes) maps decimal 0–15 to segment patterns. Use the project doc/phase1_design.pdf for the full table and K-map derivations.

---

## Minimal Boolean Equations (summary)

For implementers who want gate-level equations, Karnaugh-map reduced SOP expressions are available for each segment (a..g). These were derived from the 16-entry truth table; each expression can be implemented directly with AND/OR/NOT gates or synthesized by an HDL tool.

(Full equations are included in the Phase 1 theoretical design notes.)

---

## Testing & Simulation

Recommended steps to verify the design:

1. Unit tests:
   - Verify the 4-bit ripple-carry adder with carry-in = 0 (addition) and the two's complement subtraction mode (invert B, Cin = 1).
   - Verify each logic operation (AND, OR, XOR) for all 16 input combinations (A and B each 0–15).
   - Verify shift-left behavior across A values.

2. Integration tests:
   - For each S[2:0] selection, exercise all A/B combinations and confirm Y matches the expected result.
   - Validate 7-segment outputs by mapping Y to seg[6:0] and checking against the truth table.

3. Simulation environment:
   - Create a simple HDL testbench (Verilog/VHDL/SystemVerilog) that enumerates all combinations or targeted subsets.
   - Use a simulator (Icarus Verilog, ModelSim, Questa, GHDL) to run tests and inspect signals.

4. Optional: Build a small testbench to generate waveform outputs (GTKWave) and visually confirm segment states for specific hex values.

---

## Implementation & Files (suggested repo layout)

- README.md                — this documentation
- doc/phase1_design.pdf    — theoretical design, full truth table, K-maps, and equations
- rtl/alu.v                — Verilog implementation of the ALU (arithmetic, logic, mux)
- rtl/hex7seg.v            — Verilog for Hex → 7-segment decoder (ROM or gate logic)
- tb/alu_tb.v              — testbench for ALU + decoder
- sim/                     — simulation scripts and expected logs/waveforms
- fpga/                    — synthesis constraints and top-level board files (optional)
- screenshots/             — screenshots for Logisim circuits (Arithmetic_Unit.png, Logic_Unit.png, Routing_Unit.png, Hex_Decoder.png, Top_Level.png)


---

## Screenshots

Included screenshot references (place images in `screenshots/` so links render on GitHub):

- screenshots/Arithmetic_Unit.png — screenshot of the Arithmetic_Unit circuit in Logisim-Evolution
- screenshots/Logic_Unit.png — screenshot of the Logic_Unit circuit in Logisim-Evolution
- screenshots/Routing_Unit.png — screenshot of the Routing_Unit (MUX routing) circuit
- screenshots/Hex_Decoder.png — screenshot of the Hex_Decoder implementation (logic gates / combinational analysis output)
- screenshots/Top_Level.png — screenshot of the top-level integrated circuit showing inputs, ALU blocks, MUX, decoder, and 7-segment display

---

## Notes

- Deadline note: The original assignment deadline was midnight on Saturday, July 25th. Today is July 30th; if this is for submission, check with your instructor about late credit policies. This README includes both Phase 1 theory and Phase 2 gate-level implementation guidance for Logisim-Evolution.

- If any screenshot filenames differ from the ones listed above, tell me their exact names and I'll update the README links accordingly.

---

## Future extensions / enhancements

- Extend arithmetic to 4-bit signed detection and overflow reporting.
- Add carry/borrow, zero, sign, and parity flags.
- Implement rotate-left/right operations and arithmetic shifts.
- Implement multi-bit shifts (by variable amount) using barrel shifters.
- Expand to an 8-bit or 16-bit ALU by instantiating additional modules and handling carries.
- Synthesize to an FPGA board and connect a physical 7-seg display with proper current-limiting resistors.

---

## License

Add a license file to the repository (e.g., MIT) if you intend to make this project open-source. If you prefer a different license, include LICENSE.md with the appropriate text.

---

## Author

Project: 4-BIT-ALU-DESIGN  
Author / Owner: KalonzoBrian

---

If you'd like, I can also:
- Commit the referenced screenshots into `screenshots/` if you upload them here, or I can create placeholder images for you to replace.
- Generate a starter Verilog implementation and testbench and commit them under `rtl/` and `tb/`.
- Produce the full truth table and explicit minimized boolean expressions in a PDF or markdown file ready to commit.
