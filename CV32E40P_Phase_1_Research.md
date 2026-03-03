# CTP Phase 1 — Research: `cv32e40p`

**Core:** `cv32e40p`
**Version / tag:** `cv32e40p_v1.8.3`
**Parameter config being certified:** `COREV_PULP=0, FPU=0, NUM_MHPMCOUNTERS=1`
**Repository:** `/home/marin/Projects/RISC-V/risc-v-certification/Core_repos/cv32e40p_v1.8.3/`
**Manual:** `docs/source/` in the above repo

> Fill every field before writing any CTP files. Mark each value as:
> - VERIFIED — confirmed from RTL, testbench constraints, or simulation
> - FROM MANUAL — taken from official documentation (docs/source/*.rst)
> - FROM EXAMPLE — adopted from cv32e20 or other working CTP
> - ASSUMED — reasonable default, not explicitly documented
> - DERIVED — calculated from other researched fields
> - UNKNOWN — still need to find this

> [!important] One config per certification run
> This research file describes **exactly one parameter configuration**: `COREV_PULP=0, FPU=0, NUM_MHPMCOUNTERS=1`.
> ACT4 reads the UDB yaml and runs only the test suites for what is declared. There is no concept of testing all configs simultaneously.
>
> **Consequence for this file:**
> - Extensions marked `NOT in this config` (F, Zfinx) are documented here for completeness but will **not** appear in `implemented_extensions` in the yaml → ACT4 generates no FPU tests
> - `NUM_MHPMCOUNTERS=1` goes into the yaml `params` block — ACT4 uses this to know only counter index 3 is valid
> - A future `FPU=1` certification would be a **separate CTP**, separate yaml, separate run
> - Config choice (`FPU=0, NUM_MHPMCOUNTERS=1`) derived from core-v-verif defaults — confirm scope with Mike → [[Notes/RISC-V/CV32E40P/CTP/Mentor Questions — CV32E40P CTP|Q5]]

## Quick Navigation

| Section                              | Topic                     | CTP files fed                                                                      |
| ------------------------------------ | ------------------------- | ---------------------------------------------------------------------------------- |
| [1](#1-isa-profile)                  | ISA Profile               | `cv32e40p.yaml`                                                                    |
| [2](#2-memory-map)                   | Memory Map                | `link.ld`, `rvmodel_macros.h`, `sail.json`, `rvtest_config.h`, `rvtest_config.svh` |
| [3](#3-implementation-ids)           | Implementation IDs        | `sail.json`                                                                        |
| [4](#4-mtvec-warl)                   | mtvec WARL                | `sail.json`, `cv32e40p.yaml`                                                       |
| [5](#5-trap--exception-behavior)     | Trap & Exception Behavior | `cv32e40p.yaml`                                                                    |
| [6](#6-mtval-reporting)              | mtval Reporting           | `cv32e40p.yaml`                                                                    |
| [7](#7-misaligned-loadstore)         | Misaligned Load/Store     | `cv32e40p.yaml`, `sail.json`                                                       |
| [8](#8-hpm-counters)                 | HPM Counters              | `cv32e40p.yaml`                                                                    |
| [9](#9-sm-parameters)                | Sm Parameters             | `cv32e40p.yaml`                                                                    |
| [10](#10-pmp)                        | PMP                       | `cv32e40p.yaml`, `rvtest_config.h`, `rvtest_config.svh`                            |
| [11](#11-fpu)                        | FPU                       | `cv32e40p.yaml`                                                                    |
| [12](#12-interrupts)                 | Interrupts                | `rvmodel_macros.h`                                                                 |
| [13](#13-misa)                       | MISA                      | `cv32e40p.yaml`, `sail.json`                                                       |
| [13A](#13a-yaml-metadata-fields)     | YAML Metadata             | `cv32e40p.yaml`                                                                    |
| [13B](#13b-csr-reference-table)      | CSR Reference             | `cv32e40p.yaml`                                                                    |
| [13C](#13c-tool-configuration)       | Tool Configuration        | `test_config.yaml`                                                                 |
| [14](#14-open-questions--next-steps) | Open Questions            | —                                                                                  |
| [15](#15-sources-consulted)          | Sources Consulted         | —                                                                                  |
| [16](#16-readiness-checklist)        | Readiness Checklist       | —                                                                                  |
| [14](#14-sources-consulted)          | Sources Consulted         | —                                                                                  |
---

## 1. ISA Profile

> **CTP file this feeds → `cv32e40p.yaml`**
> Every row in the extension table becomes one entry in `implemented_extensions`. The base ISA, XLEN, and endianness also go into the yaml `params` block. This is the only section that maps directly and entirely to the UDB yaml.

**Base ISA:** `[x] RV32I  [ ] RV32E  [ ] RV64I`
**XLEN:** 32
**Physical address width:** 32
**Endianness:** `[x] little  [ ] big`

### Implemented Extensions

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| I | 2.1 | always | hardwired | `I.csv` | intro.rst line 72 | FROM MANUAL |
| M | 2.0 | always | hardwired | `M.csv` | intro.rst line 73 | FROM MANUAL |
| C | 2.0 | always | hardwired | UDB only | intro.rst line 74 | FROM MANUAL |
| Zca | 1.0.0 | always | hardwired — same instructions as C on RV32 without F | `Zca.csv` | intro.rst + cv32e20 + testplans/ | FROM MANUAL |
| Zicsr | 2.0 | always | hardwired | `Zicsr.csv` | intro.rst line 75 | FROM MANUAL |
| Zifencei | 2.0 | always | hardwired | `Zifencei.csv` | intro.rst line 76 | FROM MANUAL |
| Zicntr | 2.0 | always | hardwired — must declare if core has cycle/time/instret | `Zicntr.csv` | intro.rst line 77; testplans/Zicntr.csv | FROM MANUAL |
| F | 2.2 | **NOT in this config** | param: FPU=1 (we use FPU=0) | N/A — not declared | intro.rst line 78 | FROM MANUAL |
| Zfinx | 1.0 | **NOT in this config** | param: ZFINX=1 (we use 0) | N/A — not declared | intro.rst line 79 | FROM MANUAL |
| Sm | 1.11.0 | always | hardwired | UDB only | intro.rst line 133; UDB ext/Sm.yaml | FROM MANUAL |
| Smhpm | 1.11.0 | always | hardwired — defines mcountinhibit CSR | UDB only | UDB ext/Smhpm.yaml | FROM MANUAL |

> **Notes:**
> - **A extension:** NOT implemented — CV32E40P does not support RV32A atomics (intro.rst line 183)
> - **D extension:** NOT implemented — not listed in ISA
> - **Zicntr:** ACT4 has `testplans/Zicntr.csv` with actual tests (csrrs/csrrc against cycle, time, instret counters). It must be declared. cv32e20 omitting it was a CTP gap.
> - **Zca vs C:** ACT4 has `Zca.csv` but no `C.csv`. Q4 asks whether to declare both or just Zca.
> - **Sm/Smhpm 1.11.0:** From UDB schema — must match core's Priv Spec (1.11)
> - See [[Notes/RISC-V/CV32E40P/CTP/Mentor Questions — CV32E40P CTP]] — Q1/Q2/Q3 resolved. Q4 (Zca+C) and Q5 (config scope) open.

> **Rules checklist:**
> - [x] Only extensions ON in this parameter config are declared
> - [x] Xcv / custom extensions NOT declared (not certifiable — Sail doesn't know them)
> - [x] Entire category sections deleted (Atomics, Bit Manipulation, Scalar Crypto, etc.)
> - [x] Combo testplans NOT listed as separate rows
> - [x] UDB-only extensions declared (C, Sm, Smhpm)
> - [x] Version numbers checked against UDB `spec/std/isa/ext/<name>.yaml`
> - [x] **Zicntr added (2026-03-03)** — required for cycle/instret counter tests

### ISA Profile Status (2026-03-03)
- ✅ All 9 extensions declared in yaml `implemented_extensions`
- ✅ Zicntr added after Zifencei (controls Zicntr CSR access)
- ✅ YAML syntax validated
- Next: Create remaining 7 CTP files (test_config.yaml, link.ld, sail.json, etc.)

---

## 2. Memory Map

> **CTP files this feeds → `link.ld`, `rvmodel_macros.h`, `sail.json`, `rvtest_config.h`, `rvtest_config.svh`**
> Unlike section 1, memory map values do NOT go into the yaml. They scatter across five CTP files.

### Source of address values: core-v-verif testbench

> **core-v-verif** = OpenHW Group's official **hardware verification testbench** for CV32E40P
> - Location: `/home/marin/Projects/RISC-V/core-v-verif-cc/core-v-verif/cv32e40p/`
> - Key file: `env/uvme/uvme_cv32e40p_cfg.sv` — UVM configuration with constraint blocks defining **default boot addresses**
> - These are **not arbitrary defaults** — they represent proven values used to verify the actual CV32E40P RTL hardware
> - For ACT4 certification, we reuse these same addresses because they are known-good for the core
> - **Status:** All boot addresses from core-v-verif constraints are marked **VERIFIED** (verified by OpenHW Group's testbench)

### ACT4 convention addresses (constant — reuse for CV32E40P)

| Purpose | Address | CTP file | Note | Source | Status |
|---------|---------|----------|------|--------|--------|
| Exit register | `0x20000000` | `rvmodel_macros.h` | Pass = `123456789`; Fail = `1` | [[Notes/RISC-V/CTP Addresses/Exit Register\|Exit Register note]]; bsp/syscalls.c | VERIFIED |
| Virtual stdout | `0x10000000` | `rvmodel_macros.h` | Byte-by-byte write for IO | [[Notes/RISC-V/CTP Addresses/Virtual Stdout\|Virtual Stdout note]]; bsp/syscalls.c | VERIFIED |
| CLINT base | `0x02000000` | `rvmodel_macros.h`, `sail.json`, `rvtest_config.svh` | SiFive convention | [[Notes/RISC-V/CTP Addresses/CLINT\|CLINT note]]; cv32e20 sail.json line 129 | VERIFIED |
| CLINT size | `0xC0000` | `sail.json` | 786432 bytes; covers mtime + mtimecmp | [[Notes/RISC-V/CTP Addresses/CLINT Size\|CLINT Size note]]; cv32e20 sail.json line 130 | VERIFIED |

### Core-dependent addresses (from **core-v-verif official testbench** verification defaults)

> **CRITICAL SOURCE:** These values come from **`core-v-verif/cv32e40p/env/uvme/uvme_cv32e40p_cfg.sv`** — the official OpenHW Group verification testbench for CV32E40P.
> This is NOT arbitrary; it represents **proven working values used to verify the actual CV32E40P hardware RTL**.
> Constraint block: `default_cv32e40p_boot_cons` (lines 307–316)

| Purpose                  | Address                 | CTP file                                    | Source                                                                                                                 | Verification Status                              |
| ------------------------ | ----------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| **Boot address**         | `0x00000080`            | `link.ld`                                   | core-v-verif: `uvme_cv32e40p_cfg.sv:312` — constraint: `boot_addr == 'h0000_0080` (UNLESS plusarg overrides)           | **VERIFIED** (core-v-verif testbench constraint) |
| **mtvec base**           | `0x00000000`            | `rvmodel_macros.h`, `sail.json`             | core-v-verif: `uvme_cv32e40p_cfg.sv:313` — constraint: `mtvec_addr == 'h0000_0000` (vectored mode setup in bsp/crt0.S) | **VERIFIED** (core-v-verif testbench constraint) |
| **Debug halt addr**      | `0x1A110800`            | `link.ld`                                   | core-v-verif: `uvme_cv32e40p_cfg.sv:314` — constraint: `dm_halt_addr == 'h1A11_0800`                                   | **VERIFIED** (core-v-verif testbench constraint) |
| **Debug exception addr** | `0x1A111600`            | `link.ld`                                   | core-v-verif: `uvme_cv32e40p_cfg.sv:315` — constraint: `dm_exception_addr == 'h1A11_1600`                              | **VERIFIED** (core-v-verif testbench constraint) |
| **hart_id**              | `0x00000000`            | `sail.json`, `cv32e40p.yaml`                | core-v-verif: `uvme_cv32e40p_cfg.sv:308` — constraint: `mhartid == 'h0000_0000` (single-core)                          | **VERIFIED** (core-v-verif testbench constraint) |
| **Vector table**         | `0x00000000`            | `link.ld`                                   | core-v-verif BSP: `bsp/vectors.S` — placed at RAM start (same as mtvec for vectored mode)                              | **VERIFIED** (core-v-verif BSP code)             |
| **RAM range**            | `0x00000000–0x003FFFFF` | `link.ld`, `sail.json`, `rvtest_config.svh` | core-v-verif BSP: `bsp/link.ld` — 4 MB MEMORY region declaration                                                       | **VERIFIED** (core-v-verif BSP linker script)    |
| **Access fault address** | `0x00000000`            | `rvtest_config.h`, `rvtest_config.svh`      | Must be **outside** all valid RAM (0x0–0x3FFFFF). 0x0 works because boot is at 0x80, not 0x0                           | **VERIFIED** (logical deduction from RAM map)    |
| **DTB address**          | `0x00001000`            | `sail.json`                                 | cv32e20 sail.json line 87 (candidate); not core-v-verif specific                                                       | **FROM MANUAL** (adopted from cv32e20)           |
| **LARGEST_PROGRAM**      | `0x00040000`            | `rvtest_config.svh`                         | cv32e20 example (256 KB); not in cv32e40p manual or core-v-verif                                                       | **FROM MANUAL** (adopted from cv32e20)           |

### Sail memory region attributes

> **Source determination:** See [[Notes/RISC-V/CTP Addresses/How to Determine Memory Region Attributes]] and [[Notes/RISC-V/CTP Addresses/Sail Configuration Discovery Workflow]]
>
> Region: `0x00000000–0x003FFFFF` (4 MB RAM), single region

| Attribute                  | Value    | Determined by                                                 | Chapter/Source                                                                                                                       | Status      |
| -------------------------- | -------- | ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ----------- |
| **executable**             | true     | Section 1: I extension present (fetch capability)             | integration.rst (no restrictions on instruction fetch)                                                                               | FROM MANUAL |
| **readable**               | true     | Section 1: I+M extensions (load capability)                   | integration.rst (load unit documented)                                                                                               | FROM MANUAL |
| **writable**               | true     | Section 1: I+M extensions (store capability)                  | integration.rst (store unit documented)                                                                                              | FROM MANUAL |
| **cacheable**              | true     | No cache module in core → true (standard convention)          | intro.rst (no cache listed)                                                                                                          | FROM MANUAL |
| **coherent**               | true     | Single-core system → always coherent                          | integration.rst (hart_id=0 single-hart)                                                                                              | FROM MANUAL |
| **read_idempotent**        | true     | RAM property: reading does not change data                    | [[Notes/RISC-V/CTP Addresses/Sail Memory Region Attributes\|RAM definition]]                                                         | FROM MANUAL |
| **write_idempotent**       | true     | RAM property: writing same value twice = writing once         | [[Notes/RISC-V/CTP Addresses/Sail Memory Region Attributes\|RAM definition]]                                                         | FROM MANUAL |
| **misaligned_fault**       | NoFault  | Section 7: CV32E40P's LSU never raises misaligned exceptions  | load_store_unit.rst                                                                                                                  | FROM MANUAL |
| **reservability**          | RsrvNone | Section 1: No Zaamo/Zalrsc extension → no reservation support | Section 1 extension list (no A extension); [[Notes/RISC-V/CTP Addresses/Sail Memory Region Attributes\|rule: no atomics = RsrvNone]] | VERIFIED    |
| **supports_cbo_zero**      | false    | Section 1: No Zicboz extension declared                       | Section 1 extension list (no Zicboz); [[Notes/RISC-V/CTP Addresses/Sail Memory Region Attributes\|rule: no Zicboz = false]]          | VERIFIED    |
| **include_in_device_tree** | true     | Standard: testbench may read DTB to discover memory           | cv32e20 sail.json line 112 (true for RAM)                                                                                            | FROM MANUAL |

### Sail platform config

> **Source determination:** See [[Notes/RISC-V/CTP Addresses/Sail Platform Config]] and [[Notes/RISC-V/CTP Addresses/Sail Configuration Discovery Workflow]]
>
> All values from cv32e20 example (sail.json lines 121–134) with inline Sail schema comments.

| Parameter                    | Value      | Constraint from Sail schema                                       | Source                                                                                                       | Status      |
| ---------------------------- | ---------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ----------- |
| **cache_block_size_exp**     | 6          | Power of 2; exactly 6 if Zic64b declared                          | cv32e20 sail.json line 121–122 (comment: "specified as power of 2")                                          | FROM MANUAL |
| **reservation_set_size_exp** | 3          | Min 2 (RV32) / 3 (RV64); max 6 (Za64rs) / 7 (Za128rs); always ≤12 | cv32e20 sail.json lines 123–126 (explicit constraint comment)                                                | FROM MANUAL |
| **clock_frequency**          | 1000000000 | Arbitrary; doesn't affect test correctness                        | cv32e20 sail.json line 132; [[Notes/RISC-V/CTP Addresses/Sail Platform Config\|timing only, no ACT4 impact]] | FROM MANUAL |
| **instructions_per_tick**    | 2          | Affects timing only; use core datasheet or conservative estimate  | cv32e20 sail.json line 133 (conservative for multi-cycle ops)                                                | FROM MANUAL |
| **wfi_is_nop**               | true       | Tests don't use WFI; simplifies simulation                        | cv32e20 sail.json line 134; [[Notes/RISC-V/CTP Addresses/Sail Platform Config\|tests never block on WFI]]    | FROM MANUAL |

### Simulation exit mechanism

Exit: write to `0x20000000`
Pass: magic value `123456789`
Fail: any other write (typically `1`)

---

## 3. Implementation IDs

> These are the **reset values** of 4 read-only CSRs. When a test reads e.g. `mvendorid`, Sail must return the same value as the real hardware. These go into `sail.json` → `platform.*`.
> One full value per CSR — sail.json does not decompose into subfields.

| CSR                                                           | Reset value  | Source                                                  | Status      |
| ------------------------------------------------------------- | ------------ | ------------------------------------------------------- | ----------- |
| [[Notes/RISC-V/CTP Addresses/mvendorid\|`mvendorid`]] (0xF11) | `0x00000602` | control_status_registers.rst — Reset Value: 0x0000_0602 | FROM MANUAL |
| [[Notes/RISC-V/CTP Addresses/marchid\|`marchid`]] (0xF12)     | `0x00000004` | control_status_registers.rst — Reset Value: 0x0000_0004 | FROM MANUAL |
| [[Notes/RISC-V/CTP Addresses/mimpid\|`mimpid`]] (0xF13)       | `0x00000000` | core_versions.rst — mimpid=0 when COREV_PULP=0, FPU=0   | FROM MANUAL |
| [[Notes/RISC-V/CTP Addresses/mhartid\|`mhartid`]] (0xF14)     | `0x00000000` | uvme_cv32e40p_cfg.sv:308 (single-core, hart_id_i=0)     | VERIFIED    |

> **mvendorid JEDEC encoding:** bits 31:7 = `0xC` (12 continuation codes = bank 13), bits 6:0 = `0x2` (manufacturer byte). Full value = `0x602`. See [[Notes/RISC-V/CTP Addresses/mvendorid]].
> **mimpid is config-dependent:** 0 only when COREV_PULP=0, FPU=0, ZFINX=0, COREV_CLUSTER=0. See [[Notes/RISC-V/CTP Addresses/mimpid]] and [[Notes/RISC-V/CV32E40P/CV32E40P - Version Differences (v1.0.0 vs v1.8.3)]].

---

## 4. mtvec WARL

> **Source:** CV32E40P control_status_registers.rst lines 549–589
> **Key finding:** Manual confirms "Both direct mode and vectored mode are supported" (line 589)
> Reset value: {mtvec_addr_i[31:8], 6'b0, **2'b01**} → defaults to vectored mode

| Parameter                     | Value                                           | Source                                                                | Status       |
| ----------------------------- | ----------------------------------------------- | --------------------------------------------------------------------- | ------------ |
| MTVEC_MODES                   | `[ ] [0]  [ ] [1]  [x] [0,1]`                   | control_status_registers.rst lines 579–581                            | **VERIFIED** |
| MTVEC_ACCESS                  | `[x] rw  [ ] ro`                                | control_status_registers.rst line 577 (MODE[0] is RW)                 | **VERIFIED** |
| MTVEC_ILLEGAL_WRITE_BEHAVIOR  | `[x] retain  [ ] nearest_legal  [ ] other: ___` | (not documented; assumed standard)                                    | **ASSUMED**  |
| MTVEC_BASE_ALIGNMENT_VECTORED | `256` bytes                                     | control_status_registers.rst line 567 ("always aligned to 256 bytes") | **VERIFIED** |
| MTVEC_BASE_ALIGNMENT_DIRECT   | `256` bytes                                     | Same alignment applies to both modes (line 567)                       | **VERIFIED** |

### Notes on MTVEC configuration
- Both MODE[0]=0 (direct) and MODE[0]=1 (vectored) are supported → declare both
- BASE is always 256-byte aligned (bits [7:2] hardwired 0)
- MODE[1] is hardwired 0 (RO)
- Reset value has MODE[0]=1 (vectored), but can be changed to direct mode at runtime

---

## 5. Trap & Exception Behavior

> **Source:** RISC-V Priv Spec § 3.3, 4.3; CV32E40P exceptions_interrupts.rst lines 102–130
> **Concepts:** [[Notes/RISC-V/Concepts/Trap Behavior — Concepts|Trap Behavior Parameters]], [[Notes/RISC-V/Concepts/Exception Codes — Concept|Exception Codes]]

| Parameter | Value | CV32E40P | Source | Status |
|-----------|-------|----------|--------|--------|
| TRAP_ON_ECALL_FROM_M | `[x] true` | **true** (cannot disable) | exc_intr.rst:122 | **VERIFIED** |
| TRAP_ON_EBREAK | `[x] true` | **true** (always) | exc_intr.rst:117 | **VERIFIED** |
| TRAP_ON_UNIMPLEMENTED_INSTRUCTION | `[x] true` | **true** (cannot disable) | exc_intr.rst:122–125 | **VERIFIED** |
| TRAP_ON_RESERVED_INSTRUCTION | `[x] true` | **true** (assumed) | Implied in illegal instr | **ASSUMED** |
| TRAP_ON_UNIMPLEMENTED_CSR | `[x] false` | **false** (silent return 0) | Standard RISC-V behavior | **ASSUMED** |
| TRAP_ON_ILLEGAL_WLRL | `[x] false` | **false** (silently mask) | RISC-V spec § 2.2 | **ASSUMED** |
| PRECISE_SYNCHRONOUS_EXCEPTIONS | `[x] true` | **true** | Priv Spec § 3.3; likely standard impl | UNKNOWN |

### Exception Sources (CV32E40P Implementation)

Three exception types documented in exceptions_interrupts.rst lines 102–125:

| Code | Name | Behavior | Manual Reference |
|------|------|----------|------------------|
| 2 | Illegal Instruction | Cannot be disabled; always active | Line 122 |
| 3 | Breakpoint (EBREAK) | Always triggers | Line 117 |
| 11 | ECALL from M-Mode | Cannot be disabled; always active | Line 122 |

**Key Citation (line 122):**
> "The illegal instruction exception and M-Mode ECALL instruction exceptions cannot be disabled and are always active."

### Notes on TRAP_ON_* Parameters

- **ECALL_FROM_M, EBREAK, UNIMPLEMENTED_INSTRUCTION:** Verified as non-disableable from manual; status VERIFIED
- **RESERVED_INSTRUCTION:** Implied by illegal instruction handling (undefined instructions trap)
- **UNIMPLEMENTED_CSR:** yaml=false (standard: undefined CSRs return 0, no trap unless implementation forces it)
- **ILLEGAL_WLRL:** yaml=false (standard: illegal WLRL writes silently masked per Priv Spec § 2.2)

### Reserved behavior (sail.json)

| Parameter | Value | Note |
|-----------|-------|------|
| amocas_odd_register | N/A | Not applicable (no Zacas) |
| fcsr_rm | N/A | Not applicable (FPU=0) |
| pmpcfg_write_only | N/A | Not applicable (PMP count=0) |
| xenvcfg_cbie | N/A | Not applicable (no S-mode) |
| rv32zdinx_odd_register | N/A | Not applicable (no Zdinx) |
## 6. mtval Reporting

> **Source:** CV32E40P control_status_registers.rst lines 665–684 (incomplete); RISC-V Priv Spec § 3.1.13
> **Issue:** Manual documentation incomplete — just says "Writes are ignored; reads return 0"
> **Yaml values:** All REPORT_VA_IN_MTVAL_* = true; MTVAL_WIDTH = 32

| Exception | mtval (yaml) | Manual docs | Status |
|-----------|--------------|-------------|--------|
| Breakpoint | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Load misaligned | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Store/AMO misaligned | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Instruction misaligned | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Load access fault | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Store/AMO access fault | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Instruction access fault | `[x] VA` | (not documented) | **ASSUMED** from yaml |
| Illegal instruction | `[x] encoding` | (not documented) | **ASSUMED** from yaml |
| **MTVAL_WIDTH** | **32 bits** | control_status_registers.rst line 673 (RO, 31:0) | **VERIFIED** |

### Notes on mtval documentation gap
- CV32E40P manual doesn't detail what gets written to mtval per exception type
- Manual just says CSR is RO (read-only) and "reads return 0" — unclear what this means in context of trap handling
- Yaml declares all exceptions report VA (virtual address), which is standard RISC-V behavior
- **Verification needed:** Check RISC-V Priv Spec § 3.1.13 for standard mtval behavior per exception type
- **MTVAL_WIDTH = 32:** Matches XLEN (32-bit RISC-V), confirmed from CSR definition

---

## 7. Misaligned Load/Store

> **Source:** CV32E40P load_store_unit.rst lines 56–70
> **Key finding:** "The LSU **never raises** address-misaligned exceptions" (line 59)

| Parameter                                 | Value                   | Manual source                                                                                        | Status           |
| ----------------------------------------- | ----------------------- | ---------------------------------------------------------------------------------------------------- | ---------------- |
| **MISALIGNED_LDST**                       | `[x] true (HW handles)` | "load/store is performed as two bus transactions in case that the data item crosses a word boundary" | **VERIFIED**     |
| **MISALIGNED_LDST_EXCEPTION_PRIORITY**    | `[x] low`               | "The LSU never raises address-misaligned exceptions" → no exception to prioritize; splits instead    | **VERIFIED**     |
| **MISALIGNED_MAX_ATOMICITY_GRANULE_SIZE** | `4096` bytes            | Manual mentions "word boundary (4 bytes)" for splits, but 4096 is unverified in manual               | **QUESTIONABLE** |
| **MISALIGNED_SPLIT_STRATEGY**             | `[x] custom`            | "Transfer corresponding to lowest address is performed first" (custom order)                         | **VERIFIED**     |

### Sail misaligned config (sail.json)

| Parameter          | Value          | Determined from | Status  |
| ------------------ | -------------- | --------------- | ------- |
| supported          | `[x] true`     | MISALIGNED_LDST=true → supported | **VERIFIED** |
| byte_by_byte       | `[x] true`     | Splits at word boundary = byte-level granularity | **VERIFIED** |
| order_decreasing   | `[ ] false`    | Manual: "lowest address performed first" = ascending order | **VERIFIED** |
| allowed_within_exp | `2` (4-byte word) | Manual: splits when crossing **word boundary** | **VERIFIED** |

### Critical Note: MISALIGNED_MAX_ATOMICITY_GRANULE_SIZE Discrepancy

**Issue:** Yaml value is 4096 bytes, but manual only documents 4-byte (word) boundary splits.

**What manual says:**
- "for non-word-aligned address... performed as two bus transactions"
- "word boundary" is the split point (= 4 bytes)
- Atomicity granule should be the max size without split = 4 bytes

**What yaml claims:** 4096 bytes (4 KB page-like size)

**Possible interpretations:**
1. 4096 is unverified guess (yaml comment admits "unable to verify from documentation")
2. Atomicity granule means something else (max addressable range? max cache line?)
3. Confusion between word boundary (4 bytes) and page/cache granule

**Impact on tests:** If granule is actually 4 bytes, tests expecting atomic 4KB accesses will fail

**TODO:**
- [ ] Ask Mike: what does MISALIGNED_MAX_ATOMICITY_GRANULE_SIZE actually mean?
- [ ] Clarify if 4096 is correct or should be 4

---

## 8. HPM Counters

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| NUM_MHPMCOUNTERS (RTL param) | 1 (certified config) | integration.rst | FROM MANUAL |
| HPM_EVENTS | `[0]` | perf_counters.rst | UNKNOWN |

**HPM_COUNTER_EN** (32 entries, T=counter exists, F=not):

`[F, F, F, T, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F]`

> Index 0=mcycle, 1=unused, 2=minstret, 3=mhpmcounter3 (only one enabled); rest hardwired 0 when NUM_MHPMCOUNTERS=1

**COUNTINHIBIT_EN** (32 entries):

`[T, F, T, T, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F, F]`

> Typical: bit 0 (mcycle), 2 (minstret), 3+ inhibitable

---

## 9. Sm Parameters

> **Source:** RISC-V Unified Database parameter definitions (Sm extension)
> **Concepts:** [[Notes/RISC-V/Concepts/Sm Parameters — Atomic Notes|Sm Extension Parameters]]

| Parameter | Value | Source | Verification | Status |
|-----------|-------|--------|---|--------|
| PRECISE_SYNCHRONOUS_EXCEPTIONS | `[x] true  [ ] false` | Sm extension requirement | Manual doesn't explicitly document exception timing precision | **ASSUMED** |
| MARCHID_IMPLEMENTED | `[x] true  [ ] false` | control_status_registers.rst — marchid=0x4 implemented | core_versions.rst confirms marchid exists and is read | **VERIFIED** |
| MIMPID_IMPLEMENTED | `[x] true  [ ] false` | control_status_registers.rst — mimpid=0x0 implemented | core_versions.rst confirms mimpid value depends on config | **VERIFIED** |
| PMA_GRANULARITY | `3` (8-byte minimum) | UDB parameter required by Sm | CV32E40P has no MMU/TLB → PMA not used | **QUESTIONABLE** |

### Notes on Sm Parameters

**PRECISE_SYNCHRONOUS_EXCEPTIONS:**
- Controls whether synchronous exceptions occur at precise instruction boundaries
- true = exceptions occur exactly at instruction boundary (standard)
- false = exception may occur at unpredictable state (non-standard, reduces test reliability)
- CV32E40P likely implements precise exceptions (RISC-V standard), but manual doesn't explicitly state this
- TODO: verify from exceptions_interrupts.rst or ask Mike

**MARCHID_IMPLEMENTED & MIMPID_IMPLEMENTED:**
- Both are true because CV32E40P implements both CSRs (marchid=0x4, mimpid config-dependent)
- UDB requires these parameters to be declared when Sm extension is present
- Related parameters: ARCH_ID_VALUE, IMP_ID_VALUE (which hold the actual CSR values)

**PMA_GRANULARITY Critical Issue:**
- PMA (Physical Memory Attributes) is used for systems with virtual memory (MMU)
- PMA_GRANULARITY = log2(minimum addressable PMA region) = 3 → 8-byte regions
- **CV32E40P has NO MMU, NO TLB, NO virtual memory** — PMA is not applicable
- Standard guidance: "for systems with an MMU, should not be smaller than 12" (4 KB page size)
- **Question:** Why is this parameter in the yaml at all if CV32E40P doesn't use PMA?
  - Possible reason: UDB schema requires it for all Sm implementations
  - Possible reason: Placeholder value (reasonable minimum of 3, but unused)
  - **Impact:** Should not affect ACT4 testing (no virtual memory tests)

### TODO
- [ ] Verify PRECISE_SYNCHRONOUS_EXCEPTIONS from exceptions_interrupts.rst
- [ ] Ask Mike: Is PMA_GRANULARITY needed for M-mode only core without MMU?

---

## 10. PMP

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| Number of PMP entries | 0 | cv32e40p_if_stage.sv:176 | VERIFIED |
| PMP usable count | 0 | | VERIFIED |
| PMP grain size (G) | N/A | | VERIFIED |
| TOR supported | N/A | | VERIFIED |
| NA4 supported | N/A | | VERIFIED |
| NAPOT supported | N/A | | VERIFIED |

> **Confirmed:** `// PMP is not supported in CV32E40P` — RTL comment in cv32e40p_if_stage.sv:176

---

## 11. FPU

> FPU=0 and ZFINX=0 in this config. Section kept as N/A reference since FPU=1 is a potential future config.

| Parameter | Value | Status |
|-----------|-------|--------|
| FPU enabled? | FPU=0 (baseline config) | VERIFIED |
| Zfinx? | ZFINX=0 | VERIFIED |
| F extension version | N/A (FPU=0) | VERIFIED |
| FPU_ADDMUL_LAT | N/A (FPU=0) | VERIFIED |
| FPU_OTHERS_LAT | N/A (FPU=0) | VERIFIED |

---

## 12. Interrupts

### Machine-mode interrupts

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| mtime address | `0x0200BFF8` (CLINT base + 0xBFF8) | CLINT spec | VERIFIED |
| mtimecmp address | `0x02004000` (CLINT base + 0x4000) | CLINT spec | VERIFIED |
| How to assert MEI | | exceptions_interrupts.rst | UNKNOWN |
| How to clear MEI | | exceptions_interrupts.rst | UNKNOWN |
| How to assert MSW | | exceptions_interrupts.rst | UNKNOWN |
| How to clear MSW | | exceptions_interrupts.rst | UNKNOWN |
| Fast interrupts (PULP)? | No (COREV_PULP=0) | integration.rst | VERIFIED |

### Supervisor-mode interrupts

> Not applicable — CV32E40P is machine-mode only (Phase 0/1)

---

## 13. MISA

| Parameter            | Value                 | Source                       | Status   |
| -------------------- | --------------------- | ---------------------------- | -------- |
| MISA_CSR_IMPLEMENTED | `[ ] true  [ ] false` | control_status_registers.rst | UNKNOWN  |
| MUTABLE_MISA_M       | `[ ] true  [ ] false` | control_status_registers.rst | UNKNOWN  |
| MUTABLE_MISA_C       | `[ ] true  [ ] false` | control_status_registers.rst | UNKNOWN  |
| MUTABLE_MISA_F       | N/A (FPU=0)           |                              | VERIFIED |

### Sail base config (sail.json → base)

| Parameter | Value | Note | Status |
|-----------|-------|------|--------|
| `writable_misa` | `[ ] true  [ ] false` | True if any MISA bit is runtime-writable | UNKNOWN |
| `writable_fiom` | `[ ] true  [ ] false` | FP I/O mode; FPU=0 so likely false | UNKNOWN |

---

## 13A. YAML Metadata Fields

> **CTP file this feeds → `cv32e40p.yaml`**
> Schema-required header fields per `config_schema.json`.

- [x] `$schema`: `config_schema.json#` (constant for all cores)
- [x] `kind`: `"architecture configuration"` (constant for all cores)
- [x] `type`: `"fully configured"` (constant for all cores)
- [x] `name`: `"cv32e40p"`
- [ ] `description`: AsciiDoc description of this config — UNKNOWN

---

## 13B. CSR Reference Table

> **CTP file this feeds → `cv32e40p.yaml`**
> Unified table of all CSRs CV32E40P implements. Each row feeds the yaml `csrs:` block.
> WARL details from Sections 4, 8, 12.

| CSR Name | Address | Access | Reset Value | WARL Notes | Feeds Section | Source | Status |
|----------|---------|--------|-------------|------------|---------------|--------|--------|
| mvendorid | 0xF11 | RO | `0x00000602` | N/A (read-only) | 3 | control_status_registers.rst | FROM MANUAL |
| marchid | 0xF12 | RO | `0x00000004` | N/A (read-only) | 3 | control_status_registers.rst | FROM MANUAL |
| mimpid | 0xF13 | RO | `0x00000000` | N/A (read-only) | 3 | core_versions.rst | FROM MANUAL |
| mhartid | 0xF14 | RO | `0x00000000` | N/A (read-only) | 3 | uvme_cv32e40p_cfg.sv | VERIFIED |
| mstatus | 0x300 | RW | | Which bits writable? | 5, 12 | control_status_registers.rst | UNKNOWN |
| misa | 0x301 | | | See Section 12 | 12 | control_status_registers.rst | UNKNOWN |
| mtvec | 0x305 | RW | `0x00000000` | MODE bits, BASE alignment | 4 | control_status_registers.rst | UNKNOWN |
| mcounteren | 0x306 | | | | 8 | control_status_registers.rst | UNKNOWN |
| mcountinhibit | 0x320 | RW | | Which bits writable? | 8 | control_status_registers.rst | UNKNOWN |
| mscratch | 0x340 | RW | | | | control_status_registers.rst | UNKNOWN |
| mepc | 0x341 | RW | | Low bits hardwired 0? | | control_status_registers.rst | UNKNOWN |
| mcause | 0x342 | RW | | | 5 | control_status_registers.rst | UNKNOWN |
| mtval | 0x343 | RW | | See Section 6 | 6 | control_status_registers.rst | UNKNOWN |
| mip | 0x344 | RW | | Which bits writable? | 11 | control_status_registers.rst | UNKNOWN |
| mie | 0x304 | RW | | Which bits writable? | 11 | control_status_registers.rst | UNKNOWN |
| mcycle | 0xB00 | RW | | | 8 | control_status_registers.rst | UNKNOWN |
| minstret | 0xB02 | RW | | | 8 | control_status_registers.rst | UNKNOWN |
| mhpmcounter3 | 0xB03 | RW | | Only counter (NUM_MHPMCOUNTERS=1) | 8 | control_status_registers.rst | UNKNOWN |
| mhpmevent3 | 0x323 | RW | | Event selector for counter 3 | 8 | control_status_registers.rst | UNKNOWN |

> **Custom CSRs:** CV32E40P has no custom CSRs in COREV_PULP=0 config.
> **Note:** mcounteren may not exist (M-mode only, no U-mode). Verify from CSR chapter.

---

## 13C. Tool Configuration

> **CTP file this feeds → `test_config.yaml`**

- [x] `compiler_exe`: `riscv64-unknown-elf-gcc` — ASSUMED (standard toolchain)
- [x] `objdump_exe`: `riscv64-unknown-elf-objdump` — ASSUMED
- [x] `ref_model_exe`: `sail_riscv_sim` — ASSUMED (v0.10 required)
- [x] `include_priv_tests`: `false` — VERIFIED (M-mode only, no S/U-mode)

---

## 14. Open Questions & Next Steps

### Confirm with Mike
- [ ] **Q4** — Zca + C in yaml: declare both or just Zca? (ACT4 has Zca.csv but no C.csv) → [[Notes/RISC-V/CV32E40P/CTP/Mentor Questions — CV32E40P CTP|Mentor Questions]]
- [ ] **Q5** — Is `COREV_PULP=0, FPU=0, NUM_MHPMCOUNTERS=1` the correct deliverable? Is FPU=1 also in scope? → [[Notes/RISC-V/CV32E40P/CTP/Mentor Questions — CV32E40P CTP|Mentor Questions]]

### Still to research (read chapters, then fill)

| Section | Chapter to read | Key things to find |
|---------|----------------|-------------------|
| 2 (Sail) | `load_store_unit.rst` | misaligned_fault: does HW handle misaligned or trap? |
| 3 (IDs) | `control_status_registers.rst`, `core_versions.rst` | mvendorid, marchid, mimpid values |
| 4 (mtvec) | `control_status_registers.rst` | mtvec modes, RW, alignment, WARL behavior |
| 5 (traps) | `exceptions_interrupts.rst` | ecall, ebreak, illegal insn, unimpl CSR behavior |
| 6 (mtval) | `exceptions_interrupts.rst` | What goes in mtval per exception type |
| 7 (misalign) | `load_store_unit.rst` | HW handled or trap? Priority? |
| 8 (HPM) | `perf_counters.rst` | HPM events, mcountinhibit implementation |
| 11 (IRQ) | `exceptions_interrupts.rst` | MEI/MSI/MTI assertion/clear mechanisms |
| 12 (MISA) | `control_status_registers.rst` | misa implemented, which bits writable |

### Address to confirm
- [ ] DTB address — using `0x1000` from cv32e20; confirm appropriate for CV32E40P
- [ ] misaligned_fault — read `load_store_unit.rst` Section 7 to determine NoFault vs Fault

---

## 15. Sources Consulted

| Document / File | What it answered |
|-----------------|-----------------|
| `intro.rst` | ISA extensions, Priv Spec 1.11, no A/D confirmed |
| `integration.rst` | boot_addr_i, mtvec_addr_i, parameters, NUM_MHPMCOUNTERS, COREV_PULP |
| `control_status_registers.rst` | CSR map, mtvec, ID registers (not fully read) |
| `exceptions_interrupts.rst` | (not yet read) |
| `load_store_unit.rst` | (not yet read) |
| `perf_counters.rst` | (not yet read) |
| `core_versions.rst` | (not yet read) |
| `cv32e40p_if_stage.sv` | PMP not supported (line 176) |
| `cv32e40p_cs_registers.sv` | (not yet read) |
| cv32e20 CTP example | Memory layout, platform config template, Sail schema comments |
| [[Notes/RISC-V/CTP Addresses/\|CTP Addresses atomic notes]] | Atomic documentation for all Section 2 concepts |
| [[Notes/RISC-V/CTP Addresses/Sail Configuration Discovery Workflow\|Sail discovery workflow]] | How to determine Sail settings |
| UDB `spec/std/isa/ext/Sm.yaml` | Sm 1.11.0 ratified |
| UDB `spec/std/isa/ext/Smhpm.yaml` | Smhpm 1.11.0 ratified; mcountinhibit |
| UDB `spec/std/isa/ext/Zicntr.yaml` | Zicntr v2.0.0 |
| `testplans/*.csv` | Confirmed ACT4 test files exist for declared extensions |
| `bsp/link.ld` | RAM layout, boot=0x80, debug addresses |
| `bsp/syscalls.c` | stdout=0x10000000, exit=0x20000000 |
| `bsp/crt0.S`, `bsp/vectors.S` | mtvec setup (vectored at 0x0), vector table at 0x0 |
| `uvme_cv32e40p_cfg.sv` | UVM defaults: boot=0x80, mtvec=0x0, hart_id=0x0, dm_halt=0x1A110800, dm_exception=0x1A111600 |

---

## 16. Readiness Checklist

### Memory Map & Addresses
- [x] Boot address confirmed (`0x80`)
- [x] Trap handler (mtvec) base confirmed (`0x0` vectored)
- [x] Exit address confirmed (`0x20000000` — ACT4 standard)
- [x] CLINT address confirmed (`0x02000000` — ACT4 standard)
- [x] CLINT size confirmed (`0xC0000`)
- [x] RAM range confirmed (`0x0–0x3FFFFF`)
- [x] Debug addresses confirmed (halt=`0x1A110800`, exception=`0x1A111600`)
- [x] hart_id confirmed (`0x0`)
- [x] Access fault address confirmed (`0x0`)
- [x] Sail memory attributes filled (with determination sources)
- [x] Sail platform config filled (with determination sources)

### ISA & Extensions
- [x] Extension list finalized (pending Q4 on Zca+C)
- [ ] MISA confirmed (section 12)
- [x] YAML metadata fields set (section 12A) — constants filled

### CSRs & IDs
- [x] Implementation IDs confirmed (section 3) — mvendorid=`0x602`, marchid=`0x4`, mimpid=`0x0`
- [ ] mtvec WARL modes confirmed (section 4)
- [ ] CSR reference table complete (section 12B) — structure added, values UNKNOWN

### Core Behavior
- [ ] Sections 5, 6, 7 filled (traps, mtval, misaligned)
- [ ] misaligned_fault confirmed from load_store_unit.rst
- [ ] HPM events confirmed (section 8)
- [x] PMP confirmed — not implemented

### Tools & Meta
- [x] Tool paths confirmed (section 12C)
- [ ] Q4 and Q5 answered

---

