# 404IUTCOMP

**A 32-bit RISC-V Harvard-Architecture Pipelined Processor, built from scratch in Logisim Evolution.**

> Five pipeline stages. A hand-built forwarding unit. A variable-length instruction fetch mechanism bolted on for extra credit. This is that project.

---

## 📖 Overview

**404IUTCOMP** is a fully custom 32-bit RISC-V processor designed and simulated in **Logisim Evolution**, built as a university computer architecture project. It implements a **5-stage Harvard pipeline** (separate instruction and data memories) and goes beyond the baseline requirements with a hardware **data forwarding unit** to resolve pipeline hazards without paying the cost of constant stalling.

As a bonus/extra-credit extension, the design also supports **mixed 16-bit and 32-bit instruction fetching**, inspired by ARM's Thumb instruction-width scheme — meaning the fetch stage can dynamically detect and align variable-length instructions rather than assuming a fixed 4-byte stride.

This was, without question, the most demanding project of the coursework — spanning datapath design, hazard analysis, timing verification, and a fair amount of GUI-based circuit surgery in Logisim.

---

## 🏗️ Architecture

### Pipeline Stages

```
   IF    →    ID    →    EX    →    MEM    →    WB
(Fetch)   (Decode)  (Execute)  (Memory)   (Writeback)
```

- **Harvard architecture** — independent instruction and data memories, allowing simultaneous instruction fetch and data access with no structural memory conflict.
- **32-bit RISC-V ISA** — standard R/I/S/B/U/J instruction formats.
- Fully pipelined register file with dedicated read/write ports timed to the pipeline's clock discipline.

### Data Forwarding Unit

The core architectural contribution of this project is a **hardware hazard-resolution unit** that eliminates unnecessary pipeline stalls for RAW (Read-After-Write) hazards:

- **`EX/MEM` and `MEM/WB` forwarding paths**, each cascaded through dedicated pipeline registers (`EXMEM_Forward_Pipe`) carrying operand values forward with correct stage-aligned timing.
- **`Forwarding_Unit`** — a comparator block that checks destination/source register matches across pipeline stages and generates forwarding-select control signals.
- **Four cascaded 2-to-1 multiplexers** inserted into the ALU's operand A/B paths, allowing forwarded values to bypass the register file entirely when a hazard is detected.
- **Branch condition forwarding** — the `BranchUnit` also consumes forwarded operands (not raw register file output), ensuring branch decisions are evaluated on the most current values rather than stale ones — critical for correctness in tight loops with back-to-back dependent branches.

**Timing convention used throughout the design:**
| Destination stage | Delay from ID |
|---|---|
| MEM-stage timing  | 2 register delays |
| WB-stage timing   | 3 register delays |

This convention is applied consistently across every forwarded signal path in the circuit.

### Bonus: Variable-Length (16/32-bit) Instruction Fetch

Inspired by ARM Thumb's mixed-width instruction encoding, this extension modifies the fetch stage (`ThumbInstructor.circ`) to support both 2-byte and 4-byte instructions in the same instruction stream:

- Dynamic PC increment (`+2` vs `+4`) based on detected instruction width, rather than a fixed fetch stride.
- A `fetchhalf`-gated stall/bubble mechanism that prevents a partially-fetched 32-bit instruction from incorrectly propagating into the ID stage — architecturally, this stall logic is structurally analogous to a classic load-use hazard detector.
- Variable return-address computation for `JAL`/`JALR` (offset of 2 or 4, depending on the actual width of the calling instruction) instead of a hardcoded link offset.

---

## ✅ Project Status

**Completed and presented.** The processor — including the base pipeline, the full data forwarding implementation, and the bonus 16/32-bit fetch extension — was successfully demonstrated and defended in a live examiner presentation, with the design and its behavior fully verified and approved.

### What's implemented
- [x] 5-stage Harvard pipeline (IF → ID → EX → MEM → WB)
- [x] Full RISC-V 32-bit datapath and control
- [x] EX/MEM and MEM/WB data forwarding
- [x] Forwarded operands routed correctly into the ALU **and** the branch comparator
- [x] Bonus: mixed 16/32-bit instruction fetch with width-aware PC updates and stall logic

### Known limitations / future work
- **Load-use hazards** are not yet resolved via stalling — a dedicated Hazard Detection Unit with bubble insertion is the natural next phase.
- **Control-hazard flush/squash logic** is not implemented; mispredicted or resolved branches don't currently squash in-flight instructions in the earlier stages.

---

## 🛠️ Tools & Environment

- **[Logisim Evolution](https://github.com/logisim-evolution/logisim-evolution)** `v4.1.0` — schematic capture and simulation
- **RISC-V 32-bit ISA** — target instruction set
- Verified with hand-written **branch-free assembly test programs** specifically constructed to exercise both EX/MEM and MEM/WB forwarding paths independently

---


## 🎓 Context

Built as a computer architecture course project — implementation, debugging, and a full live technical presentation, including anticipated examiner Q&A on pipeline hazards, forwarding correctness, and the Harvard memory model.
