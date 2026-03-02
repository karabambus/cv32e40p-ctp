# CV32E40P ACT4 Certification — Phase 1 Research

**Core:** CV32E40P v1.8.3
**Config:** COREV_PULP=0, FPU=0, NUM_MHPMCOUNTERS=1
**Status:** Phase 1 Research Complete (55%)

## Completed Research

### 1. ISA Profile
- Base: RV32I, XLEN=32, little-endian
- Declared extensions: I, M, C, Zca, Zicsr, Zifencei, Zicntr, Sm, Smhpm
- NOT implemented: A (atomics), D (double FP), user mode
- Source: intro.rst (manual verification)

### 2. Memory Map (core-v-verif verified)
- Boot: 0x00000080 (constraint: `default_cv32e40p_boot_cons`)
- mtvec: 0x00000000 (vectored mode)
- RAM: 0x00000000–0x003FFFFF (4 MB)
- Debug halt: 0x1A110800
- Debug exception: 0x1A111600
- Source: core-v-verif testbench (`uvme_cv32e40p_cfg.sv`)

### 3. Implementation IDs
- mvendorid: 0x00000602 (JEDEC-encoded)
- marchid: 0x00000004
- mimpid: 0x00000000 (only when COREV_PULP=0, FPU=0)
- mhartid: 0x00000000 (single-core)

### 4. Sail Memory Region Config
- Single 4 MB RAM region, all attributes true except:
  - misaligned_fault: NoFault (LSU handles)
  - reservability: RsrvNone (no atomics)
  - supports_cbo_zero: false (no Zicboz)

### 5. Sail Platform Config
- cache_block_size_exp: 6
- clock_frequency: 1GHz
- instructions_per_tick: 2
- wfi_is_nop: true

### 6. HPM Counters
- NUM_MHPMCOUNTERS: 1
- HPM_COUNTER_EN: [F, F, F, T, F, ...] (only counter 3)
- COUNTINHIBIT_EN: [T, F, T, T, ...]

### 7. PMP
- NOT implemented (0 entries)
- Confirmed: RTL comment `cv32e40p_if_stage.sv:176`

### 8. Tool Configuration
- compiler_exe: riscv64-unknown-elf-gcc
- ref_model_exe: sail_riscv_sim (v0.10)
- include_priv_tests: false (M-mode only)

## Research In Progress

- Section 4: mtvec WARL modes, alignment
- Section 5: Trap & exception behavior
- Section 6: mtval reporting per exception
- Section 7: Misaligned load/store handling
- Section 12: MISA register mutability

## Blocked (Mentor Feedback Needed)

- Q4: Declare Zca + C, or just Zca?
- Q5: Is FPU=1 also in scope?

## References

- CV32E40P Manual: https://github.com/openhwgroup/cv32e40p/tree/cv32e40p_v1.8.3
- core-v-verif: https://github.com/openhwgroup/core-v-verif
- ACT4 Framework: https://github.com/riscv/riscv-arch-test/tree/act4
- RISC-V Specs: https://github.com/riscv/riscv-isa-manual
