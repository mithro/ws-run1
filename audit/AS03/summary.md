# AS03: WS-Multi (ws-run1)

| | |
|---|---|
| Repository | https://github.com/AvalonSemiconductors/ws-submission-2025 @ ef1e121 (main, last commit 2026-05-20; "Final, fabbed version" is ef13a2e, 2025-12-18; later commits are docs and one spice file) |
| Slot | 1x1 (DIE_AREA 3932 x 5122 um) |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 (LICENSE file) |
| README | THIN (41 words; real docs are a Sphinx site, sources in docs/source/) |

## What the repo actually contains

Multi-project die. A 5-bit `design_sel` input (pull-ups, can be bonded) selects which sub-design owns the 42 bidir pads; the rest are held in reset. 12 analog pads. Each sub-design is a LibreLane macro in `macros/<name>/`, placed by `librelane/config.yaml`. Built on the wafer.space template with the stock 5 V library `gf180mcu_fd_sc_mcu7t5v0`, `gf180mcu_fd_io` pads, 5 V supply, RT_MAX_LAYER Metal5, corners tt_025C_5v00 / ss_125C_4v50 / ff_n40C_5v50.

Sub-designs (ID = `design_sel`, from `docs/source/multiplexer.rst`):

- 00001 VGA demo, TinyTapeout animated logo (rejunity submodule), in MiscStuff.
- 00010 VGA demo, wafer.space screensaver (TinyTapeout submodule), in MiscStuff.
- 00011 Hellorld UART + Diceroll + LED blinker (GFMPW-1 replicas), in MiscStuff.
- 00100 HP Nanoprocessor replica; 00110 Simple 8-bit CPU; 00101 24-bit-colour VGA test (all MiscStuff/DACs).
- 01000/01001 RV32IMA (rv32ima_zicsr_smnmi) with 6510 or 6502 pinout, macro 65RV32 (uses the custom DFFs).
- 10000 SID 2 FM synth (3 voices x 2 operators), macro FMSynth.
- 10100 "Secret Message" (documented as "Nothing"), macro SecretMessage.
- 11000 NTSC/PAL test signal generator, macro NTSCTest (DAC composite out; custom DFFs).
- 11001 Retro DRAM controller; 11010 Retro GPIO chip (16 GPIO, 3 timers, 3x 12-bit DAC, PRNG, UART, SPI).
- 11011 SID replica (two SIDs, 8x clock, on-chip filters, POT inputs, two DACs for analog audio), macro SID.
- 11100/11101 AS65C10/AS65C02 6502 core (claims up to 33 MHz), macro AS65x.
- 11110 C64 PLA replica with selectable delays (CASDEL[1:0], DEL), macro C64PLA.
- 11111 all pads high-Z. Support macros: Multiplexer, InputRepeater.
- Art only: avali_logo, aef2, tholin, lunar_art_1/2, plus ws_ip id and logo.

Analog / mixed-signal (`macros/Analog/`):

- `R2R_DAC/` `r2r_dac_buffered`: 12-bit R-2R DAC with big op-amp buffer (`opamp_chonky`). Extracted spice uses `nfet_05v0`, `pfet_05v0` (W=100u L=0.6u output fingers; 14u/10u input stage) and `ppolyf_u_1k_6p0` resistors (r_width 1u, length 2/4/10/23.4u). Delivered as GDS/LEF, a stub .lib (pin caps 0) and a blackbox Verilog with an ifdef SIM model. xschem benches `r2r_dac_bench.sch` and `opamp_bench.sch` (.control tran). 11 instances dac0..dac10. `opamp_slim` is unused.
- `AnalogPad/` `gf180mcu_fd_io__asig_5p0_fixed`: custom IO cell (GDS/LEF/mag/lib), a modified PDK `asig_5p0`; 12 instances analog_0..11. Commit message "Custom analog pad structure"; what is "fixed" is not determinable without the original cell.
- `gf180_ntsc/`: xschem opamp (`pfet_06v0`/`nfet_06v0`) and sine generator (`cap_mim_2f0fF`, `ppolyf_u_1k_6p0`); not referenced by any flow config (inferred: not taped out).
- `R2R_DAC/test_counter`: digital 12-bit counter macro for DAC testing (spice updated 2026-05-20, after tape-out).

DAC consumers: GPIO (3), SID (2), NTSC luma, VGA R/G/B, and spares straight to pads.

Avalon 5 V cell library: `macros/MiscStuff/gf180mcu_as_ex_mcu7t5v0` (submodule github.com/AvalonSemiconductors/gf180mcu_as_ex_mcu7t5v0, pinned 5d671a8). Cells: `dfxtp_2`, `dfxtn_2` (faster but larger DFFs per README), `dffm2_2`, `trans_1`/`trans_2` (transmission gates, blackboxed for STA). Liberty at 3 corners, LEF, CDL, Verilog. MiscStuff, NTSCTest and 65RV32 exclude stock `gf180mcu_fd_sc_mcu7t5v0__dff*`, so every flop in them maps to the Avalon DFFs; the other macros use stock cells. No RTL instantiates the trans cells.

Verification present: cocotb RTL and GL benches, AS65x FunctionalBench/IsaBench, SID/Verilator. Commits say "AS65x passing all tests", "Fix antenna violations inside macros". Not present: final GDS, DRC/LVS/STA reports, silicon data. The top flow nulls KLayout.DRC and Magic.DRC and their checkers (DRC presumably run separately; inferred).

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Tholin (GitHub 89Mods, "Luca H.") | all sub-designs, analog, cell library, docs (15 commits) | Avalon Semiconductors (Germany) | SPDX "2025 Tholin" in docs; GitHub profile (Germany, tholin.dev); AvalonSemiconductors org page |
| Leo Moser | wafer.space template, flow (18 commits) | not stated (wafer.space inferred from template origin) | git log |
| TrevorPeyton, Tim Ansell | one template commit each | not stated | git log |
| rejunity (Renaldas Zioma); TinyTapeout | VGA demos (submodules) | GitHub profile company "Research"; TinyTapeout org | gh api, submodule URLs |

## Value for GF180MCU characterisation and PDK development

### Verdict: Medium

Not built as a characterisation chip, but it has eleven 5 V op-amp-buffered R-2R DACs with direct analog pad access and the first silicon use of the Avalon ex-cell DFFs in three working macros. No test plan or silicon data, hence not High.

### Test structures and measurable circuits

| Structure | Exercises | Access | Where |
|---|---|---|---|
| 11x `r2r_dac_buffered` | `ppolyf_u_1k_6p0` matching (INL/DNL), `nfet_05v0`/`pfet_05v0` opamp gain, slew, 1 kohm drive, supply sensitivity | analog pads analog_0..11 via `asig_5p0_fixed`; code from GPIO-chip DAC registers, NTSC (luma[11:0] also on bidir[11:0]) or VGA demos | `macros/Analog/R2R_DAC`, `AnalogPad`, `src/chip_top.sv` 454-780 |
| Avalon `dfxtp_2`/`dfxtn_2` in RV32, NTSC, misc/VGA | max clock vs liberty (3 corners) | pass/fail versus clock on designs 01000/01001, 11000, VGA | `macros/MiscStuff/gf180mcu_as_ex_mcu7t5v0`, macro configs |
| C64 PLA delay settings | combinational propagation delay | CASDEL, DEL pads | `macros/C64PLA` |
| AS65C02 core | max clock (claimed 33 MHz) on stock 5 V cells | pads | `macros/AS65x` |
| IO pads | `gf180mcu_fd_io` bi_24t drive at 5 V | e.g. 28 UART TX copies in design 00011 | `docs/source/misc.rst` |

Simulation to compare: post-layout extracted spice for the DAC and bench schematics, no stored waveforms; liberty tables for the DFFs. No silicon data in the repo.

### Candidate PDK additions

- Custom IO cell `asig_5p0_fixed` and the `r2r_dac_buffered` macro (5 V, 12 bit).
- `gf180mcu_as_ex_mcu7t5v0` cells with lef/lib (3 corners)/cdl/verilog; its README says the trans cells cannot be characterised.
- Documented workarounds: a MAGIC_GDS_FLATGLOB list "to get rid of 3.3V devices (DUALGATE drawn into high-level cells)"; KLAYOUT_FILLER_OPTIONS Metal2_ignore_active because minimum Metal2 density could not be met; OpenROAD issue 8229; the cell library commit "Oops, forgot dualgate!".

### What would be needed to use it

Bare-die or bonded probing of the analog pads; 5 V supply; an 8-bit-bus controller for the GPIO chip DAC registers, or a VGA/NTSC clock source; scope or ADC. No test plan in the repo; DAC load/swing only in the docs.

## Notes and caveats

- No final GDS, reports or silicon data.
- The cell-library submodule is pinned older than the copy in WSLG (which adds a dynamic DFF).
- Whether any `trans_*` cell was placed is unverified.
- Commits after the fabbed version are docs and a spice file only.
