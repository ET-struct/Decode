# MIPS Decode Stage
### Lab — MIPS Instruction Decode (ID) Stage

#### Design Scope and Assumptions
*   **In scope:** Implementation of the Instruction Decode (ID) stage of a MIPS pipeline, which includes the Control Unit, Register File, Sign Extension, and the ID/EX Latch.
*   **Out of scope:** Other MIPS pipeline stages (IF, EX, MEM, WB) and ALU/Branch computation logic. 
*   **Assumptions:** The design operates on a 32-bit architecture and relies on a positive edge-triggered clock and a synchronous high reset signal to initialize internal registers.
*   **Limitations:** The register file (`REG[0:31]`) is pre-populated with a limited set of dummy data (10 hardcoded hex values starting at index 0) rather than being dynamically initialized from memory. 

#### Requirements
| Requirement | Description | Verification Method |
| ------ | ------ | ------ |
| REQ-01 | The Control unit shall correctly output `wb`, `mem`, and `ex` signals based on the input 6-bit opcode (R-type, LW, SW, BEQ).[cite: 5] | Testbench simulation (`decodeTB.v`) and waveform inspection. |
| REQ-02 | The Register File shall output 32-bit data for registers `rs` and `rt` and write data to `rd` when `regwrite` is high.[cite: 3] | Testbench simulation (`decodeTB.v`) and waveform inspection. |
| REQ-03 | The Sign Extend module shall successfully extend a 16-bit immediate to a 32-bit value.[cite: 4] | Testbench simulation (`decodeTB.v`) and waveform inspection. |
| REQ-04 | The ID/EX Latch shall correctly pass the register outputs, control signals, and instruction bits to the next stage on the clock edge, or clear them on reset. | Testbench simulation (`decodeTB.v`) and waveform inspection. |

#### Architecture
##### Timing or Behavioral Summary
The Instruction Decode stage receives a 32-bit instruction and the incremented PC from the IF stage[cite: 6]. The top 6 bits (opcode) are decoded by the Control Unit into execution, memory, and write-back signals (`ex`, `mem`, `wb`). Simultaneously, the Register File uses the `rs` and `rt` fields to output read data (`readdat1`, `readdat2`) while the lower 16 bits of the instruction are sign-extended to 32 bits. All of this data, alongside the relevant destination register bits, is passed into the `idExLatch` and captured on the positive clock edge to be fed into the Execute (EX) stage.
['decode.png']()

#### Implementation
##### Source Code
| File | Description |
| ------ | ------ |
| [`decode.v`](decode.v) | Final ID stage module instantiating and connecting the Register File, Control, Sign Extend, and ID/EX Latch. |
| [`regfile.v`](regfile.v) | 32x32-bit register file module pre-loaded with dummy values and supporting synchronous reset. |
| [`control.v`](control.v) | Control module decoding MIPS opcodes (RTYPE, LW, SW, BEQ) into pipeline control signals. |
| [`signExtend.v`](signExtend.v) | Combinational logic to extend a 16-bit immediate to 32 bits. |
| [`idExLatch.v`](idExLatch.v) | Pipeline latch capturing signals on the clock edge for the Execute stage. |
| [`decodeTB.v`](decodeTB.v) | Verification testbench driving clock, reset, and instruction inputs to test decoding and register writes. |

##### Implementation Notes
*  **Synchronous Reset:** A synchronous reset (`rst`) is explicitly included in `regfile.v`, `control.v`, and `idExLatch.v` to zero out their values and clear uninitialized states in the simulator.
*  **Control Opcode Defaults:** The `control.v` module implements a `default` case for unrecognized opcodes that prints `"Opcode not recognized."` to the console and safely zeros out the control signals.

#### Verification
##### Verification Strategy
The design is verified using an integrated testbench (`decodeTB.v`) that first initializes a system reset, then progressively feeds the module a sequence of MIPS instruction formats: R-Format (ADD), Branch (BEQ), Load Word (LW), and Store Word (SW). It concludes by driving `wb_reg_write_tb` high to simulate the Write Back stage updating the Register File.

#### Reproduction Instructions
##### Tools and Versions
*   **Simulator / synthesis tool:** Xilinx Vivado (GUI) or Icarus Verilog
*   **Language / standard:** Verilog

##### Build and Run
```bash
To compile and run using a standard Verilog simulator (e.g., Icarus Verilog):
iverilog -o decode_sim decodeTB.v decode.v regfile.v signExtend.v control.v idExLatch.v
vvp decode_sim
```
Expected Output:
```text
# The testbench generates a waveform. Expected key signal transitions:
  0 ns : rst_tb = 1 (Registers cleared)
  2 ns : rst_tb = 0, Instruction R-Format (ADD) provided to ID stage.
  4 ns : Instruction BEQ decoded.
  6 ns : Instruction LW decoded.
  8 ns : Instruction SW decoded.
 10 ns : wb_reg_write_tb = 1 (Simulates writing to REG[2]).
 12 ns : Simulation finishes via $finish command.
```
## Repository Organizaiton
.
```text
├── README.md
├── images/
│   └── decode.png
├── control.v
├── decode.v
├── decodeTB.v
├── idExLatch.v
├── regfile.v
└── signExtend.v
```
## Known Issues and Future Improvements
- **Technical Debt (Limited Instruction Support):** The Control module (`control.v`) currently only decodes a very limited subset of MIPS opcodes: R-Format, Load Word (LW), Store Word (SW), Branch on Equal (BEQ), and NOP[cite: 13]. Any other instruction falls into the `default` case and zeroes out the control signals while printing an "Opcode not recognized" warning[cite: 13].
- **Technical Debt (Manual Write-Back):** Because this stage is isolated from the rest of the pipeline, the Write Back (WB) phase must be manually simulated in the testbench (`decodeTB.v`) by manually driving the `wb_reg_write_tb`, `wb_write_reg_location_tb`, and `mem_wb_write_data_tb` signals at specific time intervals to verify register writes[cite: 7].
- **Future Improvement (Expand Instruction Set):** Update the `control.v` module to support a wider array of MIPS instructions, such as J-type instructions (like `j` or `jal`) and other I-type arithmetic instructions (like `addi`), which will be necessary for executing complete programs[cite: 13].
- **Future Improvement (Pipeline Integration):** Connect the top-level `decode.v` module's inputs (like `if_id_instr` and `if_id_npc`) to the outputs of the previously completed Instruction Fetch (IF) stage, and prepare to route the `idExLatch` outputs to the upcoming Execute (EX) stage[cite: 4].

