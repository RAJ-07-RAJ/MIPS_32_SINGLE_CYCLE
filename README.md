
# 32-Bit Single-Cycle MIPS Processor (RTL Implementation)

## 📌 Project Overview

This project implements a **32-bit Single-Cycle MIPS Processor** in Verilog HDL.

The design follows a classical single-cycle architecture inspired by the implementation described in
Computer Organization and Design.

All instructions complete in **one clock cycle**, including:

* Instruction Fetch
* Decode
* Execute
* Memory Access
* Write Back

The design is modular, structurally connected, and written with clear signal visibility for debugging and verification.

---

# 🏗 Architecture Overview

### Architecture Type

* **Single-Cycle**
* **Harvard Architecture**

  * Separate Instruction Memory
  * Separate Data Memory

### Datapath Width

* 32-bit

### Register File

* 32 registers
* Each 32-bit wide
* Register `$0` is hardwired to zero

---

# 🧠 Supported ISA

This processor supports a subset of the MIPS ISA:

## R-Type Instructions

| Instruction | Operation      |
| ----------- | -------------- |
| add         | rd = rs + rt   |
| sub         | rd = rs − rt   |
| and         | rd = rs & rt   |
| or          | rd = rs | rt   |
| slt         | rd = (rs < rt) |

---

## I-Type Instructions

| Instruction | Operation            |
| ----------- | -------------------- |
| addi        | rt = rs + imm        |
| lw          | rt = MEM[rs + imm]   |
| sw          | MEM[rs + imm] = rt   |
| beq         | if (rs == rt) branch |

---

## J-Type Instructions

| Instruction | Operation              |
| ----------- | ---------------------- |
| j           | Jump to target address |

---
Good. That’s a valid structural improvement.

You’re right:

* The **control signal meaning table** must appear **before**
* The instruction-wise control signal table
* And the BEQ explanation section should be removed (since ALUOp meaning already defines it)

Below is the corrected and cleanly structured section of your README.

You can directly replace the Control Unit section with this.

---

# 🧩 Control Unit Design

The Control Unit performs opcode-based decoding and generates high-level control signals that drive the datapath.

For R-type instructions, final ALU behavior is determined using the `funct` field via the ALU Control block.

---

# 🎛 Control Signal Definitions

| Signal     | Description                                   |
| ---------- | --------------------------------------------- |
| reg_dst    | Selects destination register (rd vs rt)       |
| alu_src    | Selects ALU operand B (register vs immediate) |
| mem_to_reg | Selects write-back data (ALU vs memory)       |
| reg_write  | Enables register file write                   |
| mem_read   | Enables data memory read                      |
| mem_write  | Enables data memory write                     |
| branch     | Enables branch decision logic                 |
| jump       | Enables jump logic                            |
| alu_op     | High-level ALU operation code                 |

---

# 🎛 Control Signal Table (Per Instruction)

| Instruction | reg_dst | alu_src | mem_to_reg | reg_write | mem_read | mem_write | branch | jump | alu_op |
| ----------- | ------- | ------- | ---------- | --------- | -------- | --------- | ------ | ---- | ------ |
| R-type      | 1       | 0       | 0          | 1         | 0        | 0         | 0      | 0    | 10     |
| addi        | 0       | 1       | 0          | 1         | 0        | 0         | 0      | 0    | 00     |
| lw          | 0       | 1       | 1          | 1         | 1        | 0         | 0      | 0    | 00     |
| sw          | X       | 1       | X          | 0         | 0        | 1         | 0      | 0    | 00     |
| beq         | X       | 0       | X          | 0         | 0        | 0         | 1      | 0    | 01     |
| j           | X       | X       | X          | 0         | 0        | 0         | 0      | 1    | XX     |

---

# 🧮 ALUOp Encoding

The `alu_op` signal is a 2-bit high-level control sent to the ALU Control block.

| alu_op | Meaning                           |
| ------ | --------------------------------- |
| 00     | Force ADD                         |
| 01     | Force SUB                         |
| 10     | Decode using funct field (R-type) |

This separation simplifies control logic and avoids unnecessary opcode + funct decoding for non-R-type instructions.

---

# 💾 Memory Architecture

## Addressing Model

The processor uses a **byte-addressable memory system**.

* Each address refers to 1 byte (8 bits)
* Memory accesses are **word-aligned (32-bit)**
* Lower two address bits `[1:0]` are ignored

Word index calculation:

```
word_index = address[31:2]
```

Unaligned accesses are not supported.

---

# 📦 Data Memory

### Size

* 16 KB total
* 16,384 bytes
* 4,096 words (32-bit each)

### Implementation

```
reg [31:0] memory [0:4095];
```

### Address Mapping

```
memory[address[13:2]]
```

Why `[13:2]`?

* 16 KB → 2¹⁴ byte address space
* Remove lower 2 bits (word alignment)
* 12-bit word index → 2¹² = 4096 entries

### Behavior

* Write: Synchronous (posedge clk)
* Read: Combinational

---

# 📘 Instruction Memory

### Size

* 4 KB total
* 4,096 bytes
* 1,024 instructions (32-bit each)

### Implementation

```
reg [31:0] memory [0:1023];
```

### Address Mapping

```
instruction = memory[address[11:2]];
```

Why `[11:2]`?

* 4 KB → 2¹² byte space
* Remove lower 2 bits
* 10-bit index → 2¹⁰ = 1024 instructions

### Behavior

* Combinational read
* Initialized to NOP

---

# 🔄 Program Counter (PC)

* 32-bit register
* Increments by 4 every cycle
* Supports:

  * Sequential execution
  * Branch
  * Jump

---

# ⚙ Datapath Highlights

* Single-cycle execution
* No pipelining
* No hazard handling required
* Branch resolved in same cycle
* Zero flag used for conditional branching

---

# ⚠ Design Assumptions & Limitations

* Only word-aligned memory access supported
* No byte or half-word load/store
* No pipeline implementation
* Memory read modeled as combinational (note: FPGA BRAM is synchronous in reality)
* No exception handling

---

# 🧪 Verification Strategy

* Modular testbench validation
* Signal-level observation through top-level exposure
* PC, instruction, ALU result, and memory data are externally observable

---



