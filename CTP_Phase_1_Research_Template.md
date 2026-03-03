# CTP Phase 1 — Research: `<CORE_NAME>`

**Core:** `<CORE_NAME>`
**Version / tag:** `<e.g. cv32e40p_v1.8.3>`
**Parameter config being certified:** `<e.g. COREV_PULP=0, FPU=0, NUM_MHPMCOUNTERS=1>`
**Repository:** `<path or URL>`
**Manual:** `<path to docs/source/>`

> Fill every field before writing any CTP files. Mark each value as:
> - VERIFIED — confirmed from RTL, testbench constraints, or simulation
> - FROM MANUAL — taken from official documentation (docs/source/*.rst)
> - FROM EXAMPLE — adopted from cv32e20 or other working CTP
> - ASSUMED — reasonable default, not explicitly documented
> - DERIVED — calculated from other researched fields
> - UNKNOWN — still need to find this

> [!important] One config per certification run
> This research file describes **exactly one parameter configuration**.
> ACT4 reads the UDB yaml and runs only the test suites for what is declared. Extensions not declared → tests not run.
> A different parameter config = a separate CTP, separate yaml, separate certification run.

## Quick Navigation

| Section                          | Topic                     | CTP files fed                                                                      |
| -------------------------------- | ------------------------- | ---------------------------------------------------------------------------------- |
| [1](#1-isa-profile)              | ISA Profile               | `<core>.yaml`                                                                      |
| [2](#2-memory-map)               | Memory Map                | `link.ld`, `rvmodel_macros.h`, `sail.json`, `rvtest_config.h`, `rvtest_config.svh` |
| [3](#3-implementation-ids)       | Implementation IDs        | `sail.json`                                                                        |
| [4](#4-mtvec-warl)               | mtvec WARL                | `sail.json`, `<core>.yaml`                                                         |
| [5](#5-trap--exception-behavior) | Trap & Exception Behavior | `<core>.yaml`                                                                      |
| [6](#6-mtval-reporting)          | mtval Reporting           | `<core>.yaml`                                                                      |
| [7](#7-misaligned-loadstore)     | Misaligned Load/Store     | `<core>.yaml`, `sail.json`                                                         |
| [8](#8-hpm-counters)             | HPM Counters              | `<core>.yaml`                                                                      |
| [9](#9-pmp)                      | PMP                       | `<core>.yaml`, `rvtest_config.h`, `rvtest_config.svh`                              |
| [10](#10-fpu-if-applicable)      | FPU                       | `<core>.yaml`                                                                      |
| [11](#11-interrupts)             | Interrupts                | `rvmodel_macros.h`                                                                 |
| [12](#12-misa)                   | MISA                      | `<core>.yaml`, `sail.json`                                                         |
| [12A](#12a-yaml-metadata-fields) | YAML Metadata             | `<core>.yaml`                                                                      |
| [12B](#12b-csr-reference-table)  | CSR Reference             | `<core>.yaml`                                                                      |
| [12C](#12c-tool-configuration)   | Tool Configuration        | `test_config.yaml`                                                                 |
| [13](#13-open-questions)         | Open Questions            | —                                                                                  |
| [14](#14-sources-consulted)      | Sources Consulted         | —                                                                                  |
| [15](#15-readiness-checklist)    | Readiness Checklist       | —                                                                                  |

---

## 1. ISA Profile

> **CTP file this feeds → `<core>.yaml`**
> Every row in the extension table becomes one entry in `implemented_extensions`.
> The table below lists all extensions that ACT4 can test or that UDB requires for correctness.
> Delete rows for extensions the core does not implement. Add notes for extensions NOT in this config (e.g. param-gated).

**Base ISA:** `[ ] RV32I  [ ] RV32E  [ ] RV64I`
**XLEN:** `__`
**Physical address width:** `__`
**Endianness:** `[ ] little  [ ] big`

### Implemented Extensions

> **Legend:**
> - **ACT4 testplan** = has a `testplans/<name>.csv` → ACT4 generates tests for it
> - **UDB only** = defined in UDB schema but no dedicated ACT4 testplan (still declare for correctness)
> - **Combo testplan** = ACT4 auto-generates cross-extension tests when both extensions are declared (e.g. `ZcbZbb.csv` runs when both Zcb and Zbb are declared); no need to list these as separate extensions

#### Base Integer

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| I | | | hardwired on all cores | `I.csv` | | |
| M | | | | `M.csv` | | |
| Zmmul | | | multiply-only subset of M — use M OR Zmmul, not both | `Zmmul.csv` | | |

#### Compressed

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| C | | | ACT4 has no `C.csv` — tests via Zca; still declare if core implements full C | UDB only | | |
| Zca | | | compressed base; on RV32 without F, same instructions as C | `Zca.csv` | | |
| Zcb | | | additional compressed ops | `Zcb.csv` + combos: `ZcbZbb.csv`, `ZcbZba.csv`, `ZcbM.csv` | | |
| Zcf | | | compressed FP loads/stores — RV32 + F only | `Zcf.csv` | | |
| Zcd | | | compressed double loads/stores — requires D | `Zcd.csv` | | |

#### CSR & Fence

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| Zicsr | | | CSR read/write/set/clear instructions | `Zicsr.csv` | | |
| Zifencei | | | instruction-fetch fence | `Zifencei.csv` | | |

#### Counters & Timers

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| Zicntr | | | base counters: cycle, time, instret — must declare if core has these | `Zicntr.csv` | | |
| Zihpm | | | hardware performance monitor counters (mhpmcounter3–31) | `Zihpm.csv` | | |

#### Floating Point

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| F | | | single-precision FP | `F.csv` | | |
| D | | | double-precision FP — requires F | `D.csv` | | |
| Zfinx | | | FP in integer registers — mutually exclusive with F | UDB only | | |
| Zdinx | | | double FP in integer registers — requires Zfinx | UDB only | | |
| Zhinx | | | half FP in integer registers | UDB only | | |
| Zhinxmin | | | half FP min in integer registers | UDB only | | |
| Zfh | | | half-precision FP — requires F | `Zfh.csv` + combo: `ZfhD.csv` | | |
| Zfhmin | | | half-precision FP minimum — requires F | `Zfhmin.csv` + combo: `ZfhminD.csv` | | |
| Zfa | | | additional FP instructions | combos only: `ZfaF.csv`, `ZfaD.csv`, `ZfaZfh.csv`, `ZfaZfhD.csv` | | |
| Zfbfmin | | | BFloat16 conversion minimum | `Zfbfmin.csv` | | |

#### Atomics

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| A | | | full atomics — modern split: Zaamo + Zalrsc | UDB only (tests via Zaamo + Zalrsc) | | |
| Zaamo | | | atomic memory operations (AMO*) | `Zaamo.csv` | | |
| Zalrsc | | | load-reserved / store-conditional | `Zalrsc.csv` | | |
| Zacas | | | compare-and-swap | `Zacas.csv` + combo: `ZacasZabha.csv` | | |
| Zabha | | | byte/halfword atomics | `Zabha.csv` | | |

#### Bit Manipulation

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| B | | | full bit manipulation umbrella (= Zba+Zbb+Zbs) | UDB only | | |
| Zba | | | address generation (sh1add, sh2add, sh3add) | `Zba.csv` | | |
| Zbb | | | basic bit manipulation (clz, ctz, popcount, etc.) | `Zbb.csv` | | |
| Zbs | | | single-bit operations (bset, bclr, binv, bext) | `Zbs.csv` | | |
| Zbc | | | carry-less multiply | `Zbc.csv` | | |

#### Scalar Crypto

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| Zbkb | | | crypto bit manipulation | `Zbkb.csv` | | |
| Zbkc | | | crypto carry-less multiply | `Zbkc.csv` | | |
| Zbkx | | | crypto crossbar permutations | `Zbkx.csv` | | |
| Zknd | | | AES decryption (NIST) | `Zknd.csv` | | |
| Zkne | | | AES encryption (NIST) | `Zkne.csv` | | |
| Zknh | | | SHA-256 hash (NIST) | `Zknh.csv` | | |
| Zksh | | | SM3 hash (ShangMi) | `Zksh.csv` | | |
| Zksed | | | SM4 block cipher (ShangMi) | `Zksed.csv` | | |

#### Conditional & Hints

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| Zicond | | | integer conditional operations (czero.eqz, czero.nez) | `Zicond.csv` | | |
| Zihintpause | | | pause hint instruction | `Zihintpause.csv` | | |

#### Vector (Phase 2+)

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| V | | | full vector — implies Zve64d | `Vx.csv`, `Vf.csv`, `Vls.csv` | | |
| Zve32x | | | vector 32-bit integer | UDB only | | |
| Zve32f | | | vector 32-bit integer + FP | UDB only | | |
| Zve64x | | | vector 64-bit integer | UDB only | | |
| Zve64f | | | vector 64-bit integer + FP | UDB only | | |
| Zve64d | | | vector 64-bit integer + double FP | UDB only | | |
| Zvfhmin | | | vector half-precision FP minimum | `Zvfhmin.csv` | | |
| Zvfbfmin | | | vector BFloat16 minimum | `Zvfbfmin.csv` | | |
| Zvfbfwma | | | vector BFloat16 widening multiply-add | `Zvfbfwma.csv` | | |

#### Privileged / Machine-Mode

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| Sm | | | machine-mode priv spec version — must match core's priv spec (check UDB `ext/Sm.yaml`) | UDB only (tested via trap/CSR tests) | | |
| Smhpm | | | machine HPM — version must match Sm (check UDB `ext/Smhpm.yaml`) | UDB only (tested via Zihpm tests) | | |

#### Memory Ordering & Cache (uncommon for simple cores)

| Extension | Version | Enabled? | Hardwired or param? | ACT4 testplan? | Source | Status |
|-----------|---------|----------|---------------------|----------------|--------|--------|
| Ztso | | | total store ordering (x86-like memory model) | UDB only | | |
| Zicbom | | | cache block management operations | UDB only | | |
| Zicbop | | | cache block prefetch operations | UDB only | | |
| Zicboz | | | cache block zero operations | UDB only | | |
| Zawrs | | | wait-on-reservation-set | UDB only | | |

> **Rules checklist:**
> - [ ] Only extensions ON in this parameter config are declared
> - [ ] Xcv / custom extensions NOT declared (not certifiable — Sail doesn't know them)
> - [ ] Entire category sections deleted if they don't apply to core
> - [ ] Combo testplans (e.g. `ZcbZbb.csv`) NOT listed as separate rows (they run automatically)
> - [ ] Misalignment testplans (`Misalign.csv`, etc.) NOT listed here (they test section 7 config, not extensions)
> - [ ] UDB-only extensions declared in yaml for schema correctness
> - [ ] Version numbers checked against UDB `spec/std/isa/ext/<name>.yaml`

---

## 2. Memory Map

> **CTP files this feeds → `link.ld`, `rvmodel_macros.h`, `sail.json`, `rvtest_config.h`, `rvtest_config.svh`**
> Memory map values do NOT go into the yaml. They scatter across five CTP files.

### ACT4 convention addresses (constant — reuse for every core)

| Purpose | Address | CTP file | Note |
|---------|---------|----------|------|
| [[Notes/RISC-V/CTP Addresses/Exit Register\|Exit register]] | `0x20000000` | `rvmodel_macros.h` | Pass = write `123456789`; Fail = write `1` |
| [[Notes/RISC-V/CTP Addresses/Virtual Stdout\|Virtual stdout]] | `0x10000000` | `rvmodel_macros.h` | Byte-by-byte write for IO |
| [[Notes/RISC-V/CTP Addresses/CLINT\|CLINT base]] | `0x02000000` | `rvmodel_macros.h`, `sail.json`, `rvtest_config.svh` | SiFive convention; mtime at +0xBFF8, mtimecmp at +0x4000 |
| CLINT size | `0xC0000` (786432) | `sail.json` | Must cover mtime (+0xBFF8) and mtimecmp (+0x4000) offsets |

### Core-dependent addresses (research each time)

| Purpose                                                                | Address                      | CTP file                                    | Note                                                                        |
| ---------------------------------------------------------------------- | ---------------------------- | ------------------------------------------- | --------------------------------------------------------------------------- |
| [[Notes/RISC-V/CTP Addresses/Boot Address\|Boot address]]              | `0x________`                 | `link.ld`                                   | Check alignment constraints in integration chapter                          |
| [[Notes/RISC-V/CTP Addresses/Interrupt Vector Table\|Vector table]]    | `0x________`                 | `link.ld`                                   | Where mtvec points; must not overlap boot                                   |
| [[Notes/RISC-V/CTP Addresses/mtvec\|mtvec base]]                       | `0x________`                 | `rvmodel_macros.h`, `sail.json`             | Check WARL behavior in CSR chapter                                          |
| [[Notes/RISC-V/CTP Addresses/RAM Range\|RAM range]]                    | `0x________` to `0x________` | `link.ld`, `sail.json`, `rvtest_config.svh` | Sail region must cover all linked addresses                                 |
| [[Notes/RISC-V/CTP Addresses/hart_id\|hart_id]]                        | `0x________`                 | `sail.json`, `<core>.yaml`                  | Always 0 for single-core                                                    |
| [[Notes/RISC-V/CTP Addresses/Debug Module Addresses\|Debug halt]]      | `0x________`                 | `link.ld`                                   | From core's debug module spec                                               |
| [[Notes/RISC-V/CTP Addresses/Debug Module Addresses\|Debug exception]] | `0x________`                 | `link.ld`                                   | From core's debug module spec                                               |
| [[Notes/RISC-V/CTP Addresses/Access Fault Address\|Access fault addr]] | `0x________`                 | `rvtest_config.h`, `rvtest_config.svh`      | Must be outside all valid memory regions                                    |
| DTB address                                                            | `0x________`                 | `sail.json`                                 | Where Sail writes device tree blob; cv32e20 uses `0x1000`                   |
| LARGEST_PROGRAM                                                        | `0x________`                 | `rvtest_config.svh`                         | Max program size for PMP region calculation; cv32e20 uses `0x40000` (256KB) |

### Sail memory region attributes

> These go into `sail.json` → `memory.regions[].attributes` for each memory region.
> Most cores have a single RAM region. Cores with separate ROM/peripheral regions need multiple entries.
> For most cores, use cv32e20 defaults unless documented otherwise.

- [ ] `cacheable`: `[ ] true  [ ] false`
- [ ] `coherent`: `[ ] true  [ ] false`
- [ ] `executable`: `[ ] true  [ ] false` — must be true for region containing code
- [ ] `readable`: `[ ] true  [ ] false`
- [ ] `writable`: `[ ] true  [ ] false`
- [ ] `read_idempotent`: `[ ] true  [ ] false` — reading same address twice gives same result
- [ ] `write_idempotent`: `[ ] true  [ ] false` — writing same value twice has no extra effect
- [ ] `misaligned_fault`: `[ ] NoFault  [ ] Fault` — whether misaligned access to this region faults
- [ ] `reservability`: `[ ] RsrvNone  [ ] RsrvNonEventual  [ ] RsrvEventual` — LR/SC reservation support
- [ ] `supports_cbo_zero`: `[ ] true  [ ] false` — cache block zero support
- [ ] `include_in_device_tree`: `[ ] true  [ ] false`

> cv32e20 defaults for a single RAM region: all true except `supports_cbo_zero=false`, `misaligned_fault=NoFault`, `reservability=RsrvNone`.

### Sail platform config

> These go into `sail.json` → `platform.*`. Some overlap with other sections but must also be set here.

| Parameter | Value | Note |
|-----------|-------|------|
| `cache_block_size_exp` | `__` | Power of 2; must be 6 if Zic64b is declared |
| `reservation_set_size_exp` | `__` | Power of 2; min 2 (RV32) or 3 (RV64) for Zalrsc; max 6 (Za64rs) or 7 (Za128rs) |
| `clock_frequency` | `__` | In Hz; cv32e20 uses `1000000000` (1 GHz) |
| `instructions_per_tick` | `__` | How many instructions between timer ticks; cv32e20 uses `2` |
| `wfi_is_nop` | `[ ] true  [ ] false` | Whether WFI acts as NOP |

### Where each address goes (consistency check)

| CTP file            | What goes in it                                                                                                                                                 |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `link.ld`           | Boot addr, RAM range, debug addresses, `.tohost` section                                                                                                        |
| `rvmodel_macros.h`  | Exit register, stdout, CLINT (mtime/mtimecmp), access fault, boot macro, interrupt set/clear macros                                                             |
| `sail.json`         | RAM region (base+size+attributes), CLINT (base+size), DTB address, platform IDs, platform config (cache/reservation/clock/wfi), mtvec config, extension enables |
| `rvtest_config.h`   | Access fault address, PMP grain, PMP count                                                                                                                      |
| `rvtest_config.svh` | XLEN, RAM base, LARGEST_PROGRAM, CLINT base, access fault address, PMP config                                                                                   |

### Simulation exit mechanism

Exit: write to `0x20000000`
Pass: magic value `123456789`
Fail: any other write (typically `1`)

---

## 3. Implementation IDs

> These are the **reset values** of 4 read-only identity CSRs. Sail must return the same value as real hardware when a test reads these. They go into `sail.json` → `platform.*`.
> Record the **full reset value** per CSR — do not decompose into subfields. See atomic notes for each CSR.

| CSR | Reset value | Source | Status |
|-----|-------------|--------|--------|
| [[Notes/RISC-V/CTP Addresses/mvendorid\|`mvendorid`]] (0xF11) | `0x` | CSR chapter — look for "Reset Value" under mvendorid | |
| [[Notes/RISC-V/CTP Addresses/marchid\|`marchid`]] (0xF12) | `0x` | CSR chapter — look for "Reset Value" under marchid | |
| [[Notes/RISC-V/CTP Addresses/mimpid\|`mimpid`]] (0xF13) | `0x` | core_versions.rst or equivalent — value depends on parameter config | |
| [[Notes/RISC-V/CTP Addresses/mhartid\|`mhartid`]] (0xF14) | `0x` | Testbench config — hart_id_i signal value; always 0 for single-core | |

> **mvendorid note:** JEDEC-encoded. Read as a single full value — don't split into bank/offset for sail.json. See [[Notes/RISC-V/CTP Addresses/mvendorid]].
> **mimpid note:** Config-dependent — check core_versions chapter for your specific parameter set. See [[Notes/RISC-V/CTP Addresses/mimpid]].
> **Coverage:** These 4 CSRs cover all RISC-V machine-mode implementation identity. `mconfigptr` (0xF15) exists in newer priv specs but is not used by ACT4 and not present in simple cores.

---

## 4. mtvec WARL

> See [[Notes/RISC-V/CTP Addresses/mtvec]] for background.
> Source: CSR chapter → `mtvec` section. Goes into `sail.json` → `base.*` and `<core>.yaml` WARL definition.

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| [[Notes/RISC-V/CTP Addresses/MTVEC_MODES\|`MTVEC_MODES`]] | `[ ] [0]  [ ] [1]  [ ] [0,1]` | CSR chapter — which MODE values are described as legal | |
| [[Notes/RISC-V/CTP Addresses/MTVEC_ACCESS\|`MTVEC_ACCESS`]] | `[ ] rw  [ ] ro` | CSR chapter — is BASE field WARL or hardwired | |
| [[Notes/RISC-V/CTP Addresses/MTVEC_ILLEGAL_WRITE_BEHAVIOR\|`MTVEC_ILLEGAL_WRITE_BEHAVIOR`]] | `[ ] retain  [ ] nearest_legal  [ ] other: ___` | CSR chapter — what happens on illegal write | |
| [[Notes/RISC-V/CTP Addresses/MTVEC_BASE_ALIGNMENT_VECTORED\|`MTVEC_BASE_ALIGNMENT_VECTORED`]] | `__` bytes | CSR chapter — how many lower BASE bits are hardwired 0 in vectored mode | |
| [[Notes/RISC-V/CTP Addresses/MTVEC_BASE_ALIGNMENT_DIRECT\|`MTVEC_BASE_ALIGNMENT_DIRECT`]] | `__` bytes | CSR chapter — how many lower BASE bits are hardwired 0 in direct mode | |

---

## 5. Trap & Exception Behavior

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| `TRAP_ON_ECALL_FROM_M` | `[ ] true  [ ] false` | | |
| `TRAP_ON_EBREAK` | `[ ] true  [ ] false` | | |
| `TRAP_ON_UNIMPLEMENTED_INSTRUCTION` | `[ ] true  [ ] false` | | |
| `TRAP_ON_RESERVED_INSTRUCTION` | `[ ] true  [ ] false` | | |
| `TRAP_ON_UNIMPLEMENTED_CSR` | `[ ] true  [ ] false` | | |
| `TRAP_ON_ILLEGAL_WLRL` | `[ ] true  [ ] false` | | |
| `PRECISE_SYNCHRONOUS_EXCEPTIONS` | `[ ] true  [ ] false` | | |

### Reserved behavior (sail.json)

> How Sail should handle implementation-defined reserved cases. These go into `sail.json` → `base.reserved_behavior`.

| Parameter | Value | Note |
|-----------|-------|------|
| `amocas_odd_register` | `[ ] AMOCAS_Fatal  [ ] AMOCAS_Illegal` | Only if Zacas is declared |
| `fcsr_rm` | `[ ] Fcsr_RM_Fatal  [ ] Fcsr_RM_Illegal` | Dynamic rounding mode with reserved FRM value |
| `pmpcfg_write_only` | `[ ] PMP_Fatal  [ ] PMP_ClearPermissions` | PMP entry with R=0, W=1 |
| `xenvcfg_cbie` | `[ ] Xenvcfg_Fatal  [ ] Xenvcfg_ClearPermissions` | Reserved CBIE value 0b10 |
| `rv32zdinx_odd_register` | `[ ] Zdinx_Fatal  [ ] Zdinx_Illegal` | Only if Zdinx is declared |

---

## 6. mtval Reporting

| Exception | mtval contains | Source | Status |
|-----------|----------------|--------|--------|
| Breakpoint | `[ ] VA  [ ] 0  [ ] other` | | |
| Load misaligned | `[ ] VA  [ ] 0` | | |
| Store/AMO misaligned | `[ ] VA  [ ] 0` | | |
| Instruction misaligned | `[ ] VA  [ ] 0` | | |
| Load access fault | `[ ] VA  [ ] 0` | | |
| Store/AMO access fault | `[ ] VA  [ ] 0` | | |
| Instruction access fault | `[ ] VA  [ ] 0` | | |
| Illegal instruction | `[ ] encoding  [ ] 0` | | |
| `MTVAL_WIDTH` | `__` bits | | |

---

## 7. Misaligned Load/Store

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| `MISALIGNED_LDST` | `[ ] true (HW handles)  [ ] false (traps)` | | |
| `MISALIGNED_LDST_EXCEPTION_PRIORITY` | `[ ] low  [ ] high` | | |
| `MISALIGNED_MAX_ATOMICITY_GRANULE_SIZE` | `__` bytes | | |
| `MISALIGNED_SPLIT_STRATEGY` | `[ ] custom  [ ] other: ___` | | |

### Sail misaligned config (sail.json)

> These go into `sail.json` → `memory.misaligned`. Must be consistent with the MISALIGNED_LDST values above.

| Parameter | Value | Note |
|-----------|-------|------|
| `supported` | `[ ] true  [ ] false` | Must match MISALIGNED_LDST above |
| `byte_by_byte` | `[ ] true  [ ] false` | If supported: does HW split into byte-by-byte accesses? |
| `order_decreasing` | `[ ] true  [ ] false` | If supported: does HW access bytes in decreasing address order? |
| `allowed_within_exp` | `__` | Power of 2; max size boundary a misaligned access can straddle (0 = no restriction) |

---

## 8. HPM Counters

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| `NUM_MHPMCOUNTERS` (RTL param) | `__` (0–29) | | |
| `HPM_EVENTS` | `[__]` | | |

**HPM_COUNTER_EN** (32 entries, T=counter exists, F=not):

`[__, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __]`

> Index 0=mcycle, 1=unused, 2=minstret, 3–31=mhpmcounter3–31

**COUNTINHIBIT_EN** (32 entries):

`[__, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __, __]`

---

## 9. PMP

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| Number of PMP entries | `__` (0, 16, or 64) | | |
| PMP grain size (G) | `__` | | |
| PMP usable count | `__` (entries above this are read-only-zero) | | |
| TOR supported | `[ ] true  [ ] false` | | |
| NA4 supported | `[ ] true  [ ] false` | | |
| NAPOT supported | `[ ] true  [ ] false` | | |

---

## 10. FPU (if applicable)

> Remove this entire section if FPU=0 and ZFINX=0 in your config. Keep it as N/A reference if a future FPU=1 config is planned.

| Parameter | Value | Status |
|-----------|-------|--------|
| FPU enabled? | `[ ] FPU=0  [ ] FPU=1` | |
| Zfinx? | `[ ] ZFINX=0  [ ] ZFINX=1` | |
| F extension version | `__` | |
| FPU_ADDMUL_LAT | `__` (timing only, no ACT4 impact) | |
| FPU_OTHERS_LAT | `__` (timing only, no ACT4 impact) | |

---

## 11. Interrupts

> See [[Notes/RISC-V/CTP Addresses/CLINT]] for mtime/mtimecmp background.

### Machine-mode interrupts

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| mtime address | `0x0200BFF8` (CLINT base + 0xBFF8) | | |
| mtimecmp address | `0x02004000` (CLINT base + 0x4000) | | |
| How to assert MEI | | | |
| How to clear MEI | | | |
| How to assert MSW | | | |
| How to clear MSW | | | |
| Fast interrupts? | `[ ] yes  [ ] no` | | |

### Supervisor-mode interrupts (Phase 2+ — only if S-mode is implemented)

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| How to assert SEI | | | |
| How to clear SEI | | | |
| How to assert SSW | | | |
| How to clear SSW | | | |

---

## 12. MISA

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| `MISA_CSR_IMPLEMENTED` | `[ ] true  [ ] false` | | |
| `MUTABLE_MISA_M` | `[ ] true  [ ] false` | | |
| `MUTABLE_MISA_C` | `[ ] true  [ ] false` | | |
| `MUTABLE_MISA_F` | `[ ] true  [ ] false` | | |

### Sail base config (sail.json → base)

> These are derived from MISA research above. Fill after determining MISA writability.

| Parameter | Value | Note |
|-----------|-------|------|
| `writable_misa` | `[ ] true  [ ] false` | True if any MISA bit is runtime-writable |
| `writable_fiom` | `[ ] true  [ ] false` | FP I/O mode writability in mstatus; typically false if no FPU, true if FPU present |

---

## 12A. YAML Metadata Fields

> **CTP file this feeds → `<core>.yaml`**
> Schema-required header fields for the UDB configuration file.
> Source: `riscv-unified-db/spec/schemas/config_schema.json`

- [ ] `$schema`: `config_schema.json#` (constant for all cores)
- [ ] `kind`: `"architecture configuration"` (constant for all cores)
- [ ] `type`: `"fully configured"` (constant for all cores)
- [ ] `name`: `<core_name>` (e.g., `"cv32e40p"`)
- [ ] `description`: AsciiDoc-formatted description of this specific config

---

## 12B. CSR Reference Table

> **CTP file this feeds → `<core>.yaml`**
> Unified table of all CSRs the core implements. Each row feeds the yaml `csrs:` block.
> Fill address and access mode from the CSR chapter. WARL details from Sections 4, 8, 12.
> Add/remove rows based on what the core actually implements.

| CSR Name | Address | Access | Reset Value | WARL Notes | Feeds Section | Source | Status |
|----------|---------|--------|-------------|------------|---------------|--------|--------|
| mvendorid | 0xF11 | RO | | | 3 | | |
| marchid | 0xF12 | RO | | | 3 | | |
| mimpid | 0xF13 | RO | | | 3 | | |
| mhartid | 0xF14 | RO | | | 3 | | |
| mstatus | 0x300 | RW | | Which bits writable? | 5, 12 | | |
| misa | 0x301 | RW/RO | | See Section 12 | 12 | | |
| mtvec | 0x305 | RW | | MODE bits, BASE alignment | 4 | | |
| mcounteren | 0x306 | RW | | | 8 | | |
| mcountinhibit | 0x320 | RW | | Which bits writable? | 8 | | |
| mscratch | 0x340 | RW | | | | | |
| mepc | 0x341 | RW | | Low bits hardwired 0? | | | |
| mcause | 0x342 | RW | | | 5 | | |
| mtval | 0x343 | RW | | See Section 6 | 6 | | |
| mip | 0x344 | RW | | Which bits writable? | 11 | | |
| mie | 0x304 | RW | | Which bits writable? | 11 | | |
| mcycle | 0xB00 | RW | | | 8 | | |
| minstret | 0xB02 | RW | | | 8 | | |
| mhpmcounter3 | 0xB03 | RW | | | 8 | | |
| mhpmevent3 | 0x323 | RW | | | 8 | | |

> **Custom CSRs:** Add any core-specific CSRs below the standard ones.
> **Access key:** RO = read-only, RW = read-write, RW-H = read-write with hardware update

---

## 12C. Tool Configuration

> **CTP file this feeds → `test_config.yaml`**
> Paths to build tools and test scope settings. Environment-dependent, not core-dependent.

- [ ] `compiler_exe`: `__` (required — RISC-V GCC or LLVM path)
- [ ] `objdump_exe`: `__` (optional — for ELF analysis)
- [ ] `ref_model_exe`: `__` (required — Sail reference model executable)
- [ ] `include_priv_tests`: `[ ] true  [ ] false` (default: true; set false for M-mode-only cores)

---

## 13. Open Questions

- [ ] `___`

---

## 14. Sources Consulted

| Document / File | What it answered |
|-----------------|-----------------|
| *(add as you go)* | |

---

## 15. Readiness Checklist

### Memory Map & Addresses
- [ ] Boot address confirmed
- [ ] Trap handler address confirmed
- [ ] Exit address confirmed (`0x20000000` — ACT4 standard)
- [ ] CLINT address confirmed (`0x02000000` — ACT4 standard)
- [ ] Sail memory region attributes set (section 2)
- [ ] Sail platform config set (section 2)

### ISA & Extensions
- [ ] Extension list finalised
- [ ] MISA confirmed (section 12)
- [ ] YAML metadata fields set (section 12A)

### CSRs & IDs
- [ ] All implementation IDs confirmed (section 3)
- [ ] mtvec WARL modes confirmed (section 4)
- [ ] CSR reference table complete (section 12B)

### Core Behavior
- [ ] Sections 5, 6, 7 filled (traps, mtval, misaligned)
- [ ] HPM counter config confirmed (section 8)
- [ ] PMP count confirmed (section 9)

### Tools & Meta
- [ ] Tool paths confirmed (section 12C)
- [ ] No open questions blocking file creation

---

*Template — duplicate for each core: [[Notes/RISC-V/ACT4/ACT4 — CTP Workflow]]*
# 5. Trap & Exception Behavior
## 5. Trap & Exception Behavior

> **Source:** RISC-V Config Parameter List (ISA Parameters CSV)
> **UDB field names** → feeds `<core>.yaml` params section
> CSV reference: https://docs.google.com/spreadsheets/d/1tFGLbocTp1YNn11aN8JTmvxL_23e7x8SlBNJfOPRmAE/

**Machine Mode Exception Traps** (CSV lines 519-542):

| Parameter Name (UDB) | Description | Value | Source | Status |
|---|---|---|---|---|
| `trap_on_ecall_Mmode` | ECALL from M-mode causes trap | `[ ] true  [ ] false` | | |
| `trap_on_ebreak_Mmode` | EBREAK causes trap | `[ ] true  [ ] false` | | |
| `trap_on_illegal_inst_excp` | Illegal instruction exception | `[ ] true  [ ] false` | | |
| `trap_on_resrvd_inst` | Reserved/unimplemented instruction trap | `[ ] true  [ ] false` | | |
| `trap_on_unimp_inst` | Unimplemented custom instruction trap | `[ ] true  [ ] false` | | |
| `trap_unimp_csr` | Unimplemented CSR access behavior | `[ ] always_illegalinstruction  [ ] always_unpredictable  [ ] custom` | | |

**Reserved Behavior Config** (sail.json → base.reserved_behavior):

| Parameter Name | Value | Note |
|---|---|---|
| `amocas_odd_register` | `[ ] AMOCAS_Fatal  [ ] AMOCAS_Illegal` | Only if Zacas declared |
| `fcsr_rm` | `[ ] Fcsr_RM_Fatal  [ ] Fcsr_RM_Illegal` | Dynamic rounding mode with reserved FRM value |
| `pmpcfg_write_only` | `[ ] PMP_Fatal  [ ] PMP_ClearPermissions` | PMP entry with R=0, W=1 |
| `xenvcfg_cbie` | `[ ] Xenvcfg_Fatal  [ ] Xenvcfg_ClearPermissions` | Reserved CBIE value 0b10 |
| `rv32zdinx_odd_register` | `[ ] Zdinx_Fatal  [ ] Zdinx_Illegal` | Only if Zdinx declared |
# 6. mtval Reporting
## 6. mtval Reporting

> **Source:** RISC-V Config Parameter List — Exception Parameters (CSV lines 192-246)
> **Purpose:** Determines what exception information Sail expects to be written to mtval
> **Feeds:** `<core>.yaml` → params section
> **Reference:** https://docs.google.com/spreadsheets/d/1tFGLbocTp1YNn11aN8JTmvxL_23e7x8SlBNJfOPRmAE/edit?gid=0#gid=0 lines 192-246

| Parameter Name (UDB) | Exception Type | mtval Behavior | CSV Line | Status |
|---|---|---|---|---|
| `report_va_in_mtval_on_breakpoint` | Breakpoint (EBREAK) | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 192 | |
| `report_va_in_mtval_on_instruction_misaligned` | Instr misaligned | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 204 | |
| `report_va_in_mtval_on_instruction_access_fault` | Instr access fault | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 198 | |
| `report_va_in_mtval_on_load_misaligned` | Load misaligned | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 221 | |
| `report_va_in_mtval_on_load_access_fault` | Load access fault | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 216 | |
| `report_va_in_mtval_on_store_amo_misaligned` | Store/AMO misaligned | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 237 | |
| `report_va_in_mtval_on_store_amo_access_fault` | Store/AMO access fault | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 232 | |
| `report_encoding_in_mtval_on_illegal_instruction` | Illegal instruction | `[ ] write_va  [ ] always_write_zero  [ ] custom` | 160 | |
| `csr_mtval_width` | CSR field width | `__` bits (≥ PHYS_ADDR_WIDTH) | 36 | |

**Notes:**
- `write_va`: mtval is written with virtual address/PC of the faulting access
- `always_write_zero`: mtval is always set to 0 on exception
- `custom`: Implementation-specific behavior (leads to UNPREDICTABLE in tests)
- See CSV for detailed behavior descriptions for each exception type
# 7. Misaligned Load/Store
## 7. Misaligned Load/Store

> **Source:** RISC-V Config Parameter List — Misaligned Access Parameters (CSV lines 82-117)
> **Purpose:** Determines if/how the core handles misaligned load/store/AMO operations
> **Feeds:** `<core>.yaml` → params section; `sail.json` → memory.misaligned
> **Reference:** https://docs.google.com/spreadsheets/d/1tFGLbocTp1YNn11aN8JTmvxL_23e7x8SlBNJfOPRmAE/edit?gid=0#gid=0 lines 82–117

**Hardware Support Parameters** (UDB):

| Parameter Name (UDB) | Description | Value | CSV Line | Status |
|---|---|---|---|---|
| `misaligned_supported` | Unaligned memory accesses supported by HW | `[ ] true  [ ] false` | 117 | |
| `misaligned_exception_priority` | Priority vs. page faults / access faults | `[ ] low  [ ] high  [ ] PMA  [ ] custom` | 82 | |
| `misaligned_max_atomic_grain` | Max granule size for atomic misaligned access (bytes) | `[ ] 0  [ ] 1  [ ] 2  [ ] 4  [ ] 8  [ ] other: __` | 98 | |
| `misaligned_split_strategy` | How HW processes split accesses | `[ ] by_byte  [ ] custom` | 109 | |

**Sail misaligned config** (sail.json → memory.misaligned):

> These go into `sail.json` → `memory.misaligned`. Must be consistent with the parameters above.

| Parameter | Value | Note |
|---|---|---|
| `supported` | `[ ] true  [ ] false` | Must match `misaligned_supported` above |
| `byte_by_byte` | `[ ] true  [ ] false` | If supported: does HW split into byte-by-byte accesses? |
| `order_decreasing` | `[ ] true  [ ] false` | If supported: does HW access bytes in decreasing address order? |
| `allowed_within_exp` | `__` | Power of 2; max size boundary a misaligned access can straddle (0 = no restriction) |
# 8. HPM Counters
## 8. HPM Counters

> **Source:** RISC-V Config Parameter List — Zihpm Category (CSV lines 4–12)
> **Purpose:** Defines which hardware performance monitor counters are implemented and their inhibit behavior
> **Feeds:** `<core>.yaml` → params section
> **Reference:** https://docs.google.com/spreadsheets/d/1tFGLbocTp1YNn11aN8JTmvxL_23e7x8SlBNJfOPRmAE/edit?gid=0#gid=0 lines 4–12

**HPM Configuration Parameters** (UDB):

| Parameter Name (UDB) | Description | Value | CSV Line | Status |
|---|---|---|---|---|
| `csr_hpm_count_inhibit_enabled` | Which HPM counters support mcountinhibit disable | `[__, __, __, ...]` (32-entry boolean array) | 4 | |

**Notes on HPM_COUNTER_EN array:**
- Index 0 = `mcycle` (always implemented if M-mode exists)
- Index 1 = unused (always false)
- Index 2 = `minstret` (always implemented if M-mode exists)
- Index 3–31 = `mhpmcounter3` through `mhpmcounter31` (depends on `NUM_MHPMCOUNTERS`)
- Index _i_ = true means counter _i_ is implemented
- If `HPM_COUNTER_EN[i]` = false, then `COUNTINHIBIT_EN[i]` must also be false

**How to populate for CV32E40P (example):**
- Set `NUM_MHPMCOUNTERS` from RTL parameter (e.g., 1, 8, 16, 29)
- Set `HPM_COUNTER_EN[0]` = true (mcycle always exists)
- Set `HPM_COUNTER_EN[1]` = false (unused, always)
- Set `HPM_COUNTER_EN[2]` = true (minstret always exists)
- Set `HPM_COUNTER_EN[3]` through `[2+NUM_MHPMCOUNTERS]` = true if counters implemented
- Set remaining indices = false
- Set `COUNTINHIBIT_EN` entries to match or be false (cannot exceed `HPM_COUNTER_EN`)
# 3. Implementation IDs


### Extracting VENDOR_ID_BANK and VENDOR_ID_OFFSET from mvendorid

> **Important:** If your yaml schema requires `VENDOR_ID_BANK` and `VENDOR_ID_OFFSET` as separate fields, extract them from the mvendorid full value using bit operations (do NOT use mvendorid directly).

**Formula:**
```
VENDOR_ID_OFFSET = mvendorid & 0x7F        (bits 6:0)
VENDOR_ID_BANK   = mvendorid >> 7          (bits 31:7)
```

**Example:** If mvendorid = `0x00000602`:
- `VENDOR_ID_OFFSET` = `0x602 & 0x7F` = `0x02` (2 in decimal)
- `VENDOR_ID_BANK` = `0x602 >> 7` = `0x0C` (12 in decimal = 12 continuation codes)

**Note on JEDEC encoding:**
- `VENDOR_ID_OFFSET`: final byte of JEDEC manufacturer ID (parity bit discarded)
- `VENDOR_ID_BANK`: number of JEDEC continuation codes (JEDEC bank number = continuation codes + 1)
# 12. MISA
## 12. MISA

> **Source:** RISC-V Config Parameter List (ISA Parameters CSV)
> **UDB field names** → feeds `<core>.yaml` params section
> **Reference:** https://docs.google.com/spreadsheets/d/1tFGLbocTp1YNn11aN8JTmvxL_23e7x8SlBNJfOPRmAE/

| Parameter | Value | Source | Status |
|-----------|-------|--------|--------|
| `MISA_CSR_IMPLEMENTED` | `[ ] true  [ ] false` | CSV line 1217: does misa CSR exist (true) or is read-only-zero (false)? | |
| `MUTABLE_MISA_M` | `[ ] true  [ ] false` | CSV: can M-mode bit be written at runtime? (only if MISA_CSR_IMPLEMENTED=true) | |
| `MUTABLE_MISA_C` | `[ ] true  [ ] false` | CSV: can C extension bit be written at runtime? (only if MISA_CSR_IMPLEMENTED=true) | |
| `MUTABLE_MISA_F` | `[ ] true  [ ] false` | CSV: can F extension bit be written at runtime? (only if MISA_CSR_IMPLEMENTED=true) | |

### Notes on MUTABLE_MISA_* parameters
- **Only include MUTABLE_MISA_ext if that extension is declared in Section 1**
- If extension not declared → parameter not in config
- Example: CV32E40P has no User mode → omit `MUTABLE_MISA_U` entirely
- Mutability only makes sense if the CSR exists (`MISA_CSR_IMPLEMENTED=true`)

### Sail base config (sail.json → base)

> These are derived from MISA research above. Fill after determining MISA writability.

| Parameter | Value | Note |
|-----------|-------|------|
| `writable_misa` | `[ ] true  [ ] false` | True if any MISA bit is runtime-writable |
| `writable_fiom` | `[ ] true  [ ] false` | FP I/O mode writability in mstatus; typically false if no FPU, true if FPU present |
