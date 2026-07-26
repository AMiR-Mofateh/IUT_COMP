<div align="center">

# 404IUTCOMP

### A 32-bit RISC-V Harvard-Architecture Pipelined Processor
**Built from scratch in Logisim Evolution**

[![Made with Logisim Evolution](https://img.shields.io/badge/Made%20with-Logisim%20Evolution-blueviolet?style=for-the-badge)](https://github.com/logisim-evolution/logisim-evolution)
[![ISA](https://img.shields.io/badge/ISA-RISC--V%2032--bit-informational?style=for-the-badge)](https://riscv.org/)
[![Architecture](https://img.shields.io/badge/Architecture-Harvard%20%7C%205--Stage%20Pipeline-orange?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Completed%20%26%20Presented-success?style=for-the-badge)]()
[![Bonus](https://img.shields.io/badge/Bonus-16%2F32--bit%20Fetch-yellow?style=for-the-badge)]()

</div>

<p align="center">
<i>Five pipeline stages. A hand-built forwarding unit. A variable-length instruction fetch mechanism bolted on for extra credit.<br>This is that project.</i>
</p>

---

## 📖 Overview

**404IUTCOMP** is a fully custom 32-bit RISC-V processor designed and simulated in **Logisim Evolution**, built as a university computer architecture project. It implements a **5-stage Harvard pipeline** (separate instruction and data memories) and goes beyond the baseline requirements with a hardware **data forwarding unit** that resolves pipeline hazards without paying the cost of constant stalling.

As a bonus/extra-credit extension, the design also supports **mixed 16-bit and 32-bit instruction fetching**, inspired by ARM's Thumb instruction-width scheme — meaning the fetch stage can dynamically detect and align variable-length instructions instead of assuming a fixed 4-byte stride.

This was, without question, the most demanding project of the coursework — spanning datapath design, hazard analysis, timing verification, and a fair amount of GUI-based circuit surgery in Logisim.

---

## 🏗️ Architecture

### Pipeline Stages

<div align="center">

| `IF` | → | `ID` | → | `EX` | → | `MEM` | → | `WB` |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| Fetch | | Decode | | Execute | | Memory | | Writeback |

</div>

- 🏛️ **Harvard architecture** — independent instruction and data memories, enabling simultaneous instruction fetch and data access with zero structural memory conflict.
- 🧩 **32-bit RISC-V ISA** — standard R/I/S/B/U/J instruction formats.
- 🗂️ Fully pipelined register file with dedicated read/write ports timed to the pipeline's clock discipline.

---

### ⚡ Data Forwarding Unit

The core architectural contribution of this project: a **hardware hazard-resolution unit** that eliminates unnecessary pipeline stalls for RAW (Read-After-Write) hazards.

| Component | Role |
|---|---|
| `EXMEM_Forward_Pipe` | Cascaded pipeline registers carrying operand values forward with correct stage-aligned timing |
| `Forwarding_Unit` | Comparator block checking destination/source register matches across stages, generating forwarding-select signals |
| 4× 2-to-1 muxes | Inserted into the ALU's operand A/B paths — forwarded values bypass the register file when a hazard is detected |
| `BranchUnit` (forwarded) | Branch comparator rewired to consume **forwarded** operands instead of raw register file output |

> 🐛 **The bug that mattered most:** the `BranchUnit` was originally wired directly to raw register file outputs, bypassing forwarding entirely — causing branch conditions to evaluate on stale data (symptom: loops silently iterating twice as many times as expected). Rewiring `BranchUnit`'s inputs to the forwarding mux outputs fixed it. Small wire, big consequence.

**Timing convention used throughout the design:**

| Destination stage | Delay from ID |
|:---:|:---:|
| MEM-stage timing | 2 register delays |
| WB-stage timing | 3 register delays |

This convention is applied consistently across every forwarded signal path in the circuit.

---

### 🎁 Bonus: Variable-Length (16/32-bit) Instruction Fetch

Inspired by ARM Thumb's mixed-width instruction encoding, this extension modifies the fetch stage (`ThumbInstructor.circ`) to support both 2-byte and 4-byte instructions in the same instruction stream:

- 🔀 Dynamic PC increment (`+2` vs `+4`) based on detected instruction width, instead of a fixed fetch stride.
- ⏸️ A `fetchhalf`-gated stall/bubble mechanism preventing a partially-fetched 32-bit instruction from propagating into the ID stage — structurally analogous to a classic load-use hazard detector.
- 🔗 Variable return-address computation for `JAL`/`JALR` (offset of 2 or 4 depending on actual instruction width) instead of a hardcoded link offset.

---

## ✅ Project Status

<div align="center">

### 🏆 Completed and presented — successfully demonstrated and approved live.

</div>

The processor — including the base pipeline, the full data forwarding implementation, and the bonus 16/32-bit fetch extension — was successfully demonstrated and defended in a live examiner presentation, with the design and its behavior fully verified and approved.

**Implemented:**
- [x] 5-stage Harvard pipeline (IF → ID → EX → MEM → WB)
- [x] Full RISC-V 32-bit datapath and control
- [x] EX/MEM and MEM/WB data forwarding
- [x] Forwarded operands routed correctly into the ALU **and** the branch comparator
- [x] Bonus: mixed 16/32-bit instruction fetch with width-aware PC updates and stall logic

**Future work:**
- [ ] Load-use hazard detection with stall/bubble insertion
- [ ] Control-hazard flush/squash logic for mispredicted or resolved branches

---

## 🛠️ Tools & Environment

<div align="center">

| Tool | Purpose |
|---|---|
| ![Logisim](https://img.shields.io/badge/Logisim%20Evolution-v4.1.0-blueviolet) | Schematic capture & simulation |
| ![RISC-V](https://img.shields.io/badge/ISA-RISC--V%2032--bit-informational) | Target instruction set |

</div>

Verified with hand-written **branch-free assembly test programs**, specifically constructed to exercise both EX/MEM and MEM/WB forwarding paths independently.

---

## 📂 Repository Structure

```
IUT_COMP/
├── <pipeline .circ files>     # Base pipeline + forwarding unit
├── ThumbInstructor.circ       # Bonus 16/32-bit fetch extension
└── ...
```

## 🎓 Context

Built as a computer architecture course project — implementation, debugging, and a full live technical presentation, including anticipated examiner Q&A on pipeline hazards, forwarding correctness, and the Harvard memory model.

<div align="center">

---

*⭐ If you're studying pipelined processor design, feel free to explore the `.circ` files — the forwarding unit and the Thumb-style fetch extension are the two most instructive parts.*

</div>
