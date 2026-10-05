# JKU2: gf180mcu-jku-atbs-adc (ws-run1)

| | |
|---|---|
| Repository | https://github.com/iic-jku/gf180mcu-jku-atbs-adc @ 2fae3a6 (main, last commit 2026-10-04) |
| Slot | 1x0p5 (DIE_AREA 3932 um x 2531 um, core 3048 um x 1647 um) |
| Top cell | chip_top |
| Licence in repo | `LICENSE` is the Solderpad Hardware License v2.1 text with the notice "SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1", copyright 2026 Simon Dorrer, JKU (audit: SHL-2.1; GitHub: NOASSERTION). The 10 files with SPDX headers still say Apache-2.0. |
| README | OK (753 words), mostly template build instructions plus layout render and die micrograph |

## What the repo actually contains

The digital core of an ATBS ADC (adaptive threshold-based sampling, a level-crossing style converter for biosignals such as ECG), taken from Simon Dorrer's master's thesis "Threshold-Based Sampling ASIC with FOSS tools" (stated in the VHDL headers, dated October 2024). The chip is digital only. The analog parts of the ADC are off-chip: two comparators (upper and lower threshold), two 8-bit parallel DACs with write strobes (plus PWM alternatives), an input select switch (MAX4564 named in the source), a biosignal amplifier with leads-off detection, and a switched-capacitor clock consumer. The core tracks the input between two DAC thresholds, time-stamps each crossing, adapts the threshold step (ATBS mode), stores spikes in a memory and sends them out over UART.

Structure:

- `macros/tbs_core_board/`: the single design macro. VHDL source of record in `vhdl/` (`rtl/tbs_core_ea.vhd`, `rtl/tbs_core_board.vhd` and sub-entities: `adaptive_threshold_control`, `analog_trigger`, `dac_control`, `debouncer`, `memory2uart`, `pwm_modulator`, `sc_noc_generator`, `spike_detector`, `spike_encoder`, `spike_memory`, `sync_chain`, `time_measurement`, `uart`), converted to one Verilog file `verilog/rtl/tbs_core_board.v` by `vhdl2verilog.sh`, which is what LibreLane synthesises. Also FPGA project trees for Quartus (DE10-Standard) and Vivado (S7-Mini), ModelSim testbenches, and MATLAB scripts.
- Macro flow (`macros/tbs_core_board/librelane/config.yaml`): LibreLane Classic, die 2700 um x 1300 um, 25% target density, `STD_CELL_LIBRARY: gf180mcu_fd_sc_mcu9t5v0` with the 3.3 V Liberty corners (tt_025C_3v30, ss_n40C_3v00, ss_125C_3v00, ff_n40C_3v60, ff_125C_3v60), clock 125 ns (8 MHz), routing limited to Metal4, one ECO antenna diode inserted. 17823 cells, 297763 um2 cell area. Signoff: no setup or hold violations in any corner (worst setup slack about 47 ns), antenna, LVS and DRC passed.
- Top (`src/chip_top.sv`, `src/chip_core.sv`, `librelane/config.yaml`): LibreLane Chip flow, pads `gf180mcu_fd_io__bi_24t` (33 outputs), `gf180mcu_fd_io__in_c` (12 inputs and reset), `gf180mcu_fd_io__in_s` (clock), `gf180mcu_ws_io__dvdd/dvss` (12 VDD, 13 VSS); 72 pads in total. Top reports show antenna, LVS and DRC passed; top STA reports no reg-to-reg paths (all logic is inside the macro).
- `ip/`: wafer.space ID and logo, plus JKU, IICQC, "credits" and "names" artwork cells.
- `cocotb/chip_top_tb.py` (621 lines): five tests (TBS with full and "virtual" DAC resolution, ATBS, UART debugging) with a committed waveform dump.
- `gds/chip_top.zip` and `release/v.1.0.0/gds/chip_top.gds.chip_top.precheck.20251217_161339.output.zip`: final GDS and precheck output. The precheck output decompresses to SHA-256 `abdbf57879b209b6ab20ab4dd5c1638e4b1ed90ed915d500a704ca3591a7ef4b`, exactly the hash in `project.json`, so the manufactured layout is in the repo.
- `release/v.1.0.0/img/`: layout renders and Keyence/Andonstar die micrographs.
- `macros/tbs_core_board/measurements/generate_ecg/`: MATLAB generator and data files for ECG test stimuli (arbitrary waveform generator input). `macros/tbs_core_board/reconstruction/`: MATLAB signal reconstruction and SNDR scripts with recorded data named "PoC" (8-bit ATBS and 5-bit TBS, 10 Hz sine, over UART). These predate the silicon (committed December 2025) and come from the FPGA proof of concept, not from this chip.

No silicon measurement results are in the repo; the README shows only a die photo.

### Licence detail

Same arrangement as JKU1. Root `LICENSE` is SHL-2.1 (Solderpad wrapper over Apache-2.0, which the recipient may treat as plain Apache-2.0), appendix "Copyright 2026 Simon Dorrer, Johannes Kepler University, Department for Integrated Circuits, SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1". The repo was Apache-2.0 until commit 0f8a391 (2026-05-16), after tapeout. SPDX headers in the top RTL, scripts and docs still read `Apache-2.0`; the VHDL sources carry only an author/thesis header with no licence statement.

## Authors and affiliations

| Person / organisation | Role | Affiliation | Source |
|---|---|---|---|
| Simon Dorrer (GitHub `simi1505`) | Everything: HDL (master's thesis), floorplan, pin plan, hardening (152 of 158 commits) | JKU Linz, IICQC | `AUTHORS.md`, VHDL headers, `gh api users/simi1505` |
| Jonathan Hager | 5 commits (simulation waveform dumps) | JMU Würzburg per the JKU1 repo; not stated here | git log |
| Jakob Schaumberger (GitHub `JakobSchaumberger`) | 1 commit in October 2026 fixing the pin mapping of `dac_pwm_lower_o` | JKU per the JKU1 `AUTHORS.md`; not stated here | git log, PR #1 |
| IICQC, JKU Linz (`iic-jku`) | Hosting organisation, sponsor | Johannes Kepler University Linz | `gh api orgs/iic-jku`, README acknowledgements |

## Value for GF180MCU characterisation and PDK development

### Verdict: Low

One synthesised digital macro on stock cells with stock IO; the ADC's analog blocks are not on the chip. The only process-relevant point is that it uses the 9-track 5 V library signed off against the 3.3 V Liberty corners.

### Test structures and measurable circuits

Nothing purpose-built.

| Structure | Exercises | Access | Where |
|---|---|---|---|
| `tbs_core_board` macro run at 3.3 V | Function and maximum clock frequency of `gf180mcu_fd_sc_mcu9t5v0` at 3.0 to 3.6 V against the `*_3v30`/`*_3v00`/`*_3v60` Liberty data (STA: 125 ns period met with about 47 ns worst slack, so roughly 13 MHz predicted at the slowest corner; inferred from the slack figure) | `clk_PAD`, UART and DAC output pads | `macros/tbs_core_board/reports/hold_setup_timing.rpt` |
| `bi_24t` PWM and DAC bus outputs | IO cell drive and edge rate | Bidir pads | `src/chip_core.sv` |

Simulation results in the repo: STA reports, cocotb RTL/gate-level tests and ModelSim testbenches. No SPICE-level data.

### Candidate PDK additions

- None in cells or devices.
- Flow notes: the macro config supplies its own `LIB` map so the 5 V 9T cells are timed at 3.3 V corners alongside `gf180mcu_fd_io` 3v30 and `gf180mcu_ws_io` libs; it also disables KLayout DRC at macro level and adds one ECO diode via `Odb.InsertECODiodes`. Useful as an example of a 3.3 V digital signoff setup.

### What would be needed to use it

A packaged or chip-on-board die plus an external analog front end: two comparators, two 8-bit parallel DACs (or RC filters on the PWM outputs), input switch and amplifier, an 8 MHz clock and a UART host. No PCB design, pinout table or test plan for the ASIC is in the repo (only FPGA pin documents under `verilog/quartus` and `verilog/vivado`, and `docs/pin_planning.txt` with pad counts). The MATLAB reconstruction scripts could be reused to evaluate ADC performance once a board exists.

## Notes and caveats

- Silicon bug, fixed in the repo after tapeout: at tag `tapeout-submitted-2025-12-18`, `src/chip_core.sv` assigns `dac_pwm_upper_o` to both `bidir_out[26]` and `bidir_out[2]`, so `dac_pwm_lower_o` is not connected to any pad on the manufactured chip. Commit de47f82 (2026-10-03, "Fix pin mapping for dac_pwm_lower_o") changes `bidir_out[2]` to `dac_pwm_lower_o`. HEAD RTL therefore differs from silicon; use the tag for the as-built netlist. The parallel DAC outputs are unaffected.
- `gds/chip_top.zip` (hash `6b864279…`) is the pre-precheck GDS and does not match `project.json`; the precheck output in `release/v.1.0.0/gds/` does.
- The platform description says "digital core of an ATBS ADC"; that is accurate, and it means the chip is not an ADC on its own.
- The macro's hardened views (`macros/tbs_core_board/final/`) are git-ignored and absent; a 76 MB `out.def` is committed instead.
- `src/chip_core.sv` still has the template header "© 2025 XXX Authors".
- The measurement and reconstruction data being FPGA-derived is inferred from file names ("PoC") and commit dates, not stated explicitly.
