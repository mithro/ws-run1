# RBOY: RISCBoy-180 (ws-run1)

| | |
|---|---|
| Repository | https://github.com/Wren6991/RISCBoy-180 @ 8c2bfda (main, last commit 2026-07-14; tags v1.0.rc1, v1.0-rc2) |
| Slot | 1x1 (die 3932 x 5122 um) |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 (LICENSE file) |
| README | OK (140 words of own text; the long design diary is in `notes.md`) |

## What the repo actually contains

RB180, a single-chip games console made by Luke Wren in a two-week effort on the first wafer.space shuttle (free slot, README acknowledgement). It reuses his RISCBoy design (iCE40 original) and adds an audio processor.

- Two Hazard3 RISC-V cores ("CPU" and "APU"), identical RV32IM_Zba_Zbb_Zbkb_Zbs_Zca_Zcb_Zclsd_Zilsd with single-cycle multiplier and branch predictor (`doc/rb180_datasheet.adoc`; claims the first public tapeout of Zilsd/Zclsd). Hazard3 JTAG DTM is replaced with Luke's Two-Wire Debug (TWD, 2 pads DIO/DCK) with a virtual UART channel.
- RISCBoy PPU (tiled backgrounds, affine sprites, scanline/palette RAMs) with SPI/quad/"octal" 8080 LCD output (`hdl/dispctrl`), parallel async-SRAM controller for up to 256 kB external SRAM (38 pads), boot ROM from SPI flash, streaming-SPI peripheral, UART, GPIO, timers, an audio pipeline (16-bit 48 kSa/s to 1.5 MSa/s 4-bit PWM with a first-order sigma-delta and filters).
- Flow: LibreLane Chip flow, `STD_CELL_LIBRARY: gf180mcu_fd_sc_mcu9t5v0` (9-track 5 V), 5 V only supply, 25 MHz clk_sys target (datasheet: "5V single-supply operation at up to 25 MHz (post-PnR STA, slow corner)"). Many hand-tuned config knobs and SDC tricks documented in `librelane/config.yaml`, `librelane/chip_top.sdc` and `notes.md`.
- Macros: foundry `gf180mcu_fd_ip_sram__sram512x8m8wm1` x28 and `sram256x8m8wm1` x2 (release rc2 netlist), via `hdl/mem/sram_wrapper.v` (CPU 8 kB, APU 4 kB, PPU memories); `gf180mcu_name_plate`, ws ID and logo macros. IO: 53 `gf180mcu_fd_io__bi_t`, 3 `in_s`, ws dvdd x8 / dvss x10. About 250,000 instances total (rc2 `chip_top.pnl.v`).
- Scan cells without a scan chain: `librelane/gf180_scanflop_map.v` is a custom Yosys techmap that maps synchronous-set/clear/enable flops ("pseudo-DFFE") onto `sdffq/sdffrnq/sdffsnq` scan cells (SE as sync reset/enable select), because the stock flow lacks an enable/sync-reset flop mapping. Netlist: ~6,300 `sdff*` cells, ~300 plain `dffq`. This is not DFT (no scan chain); the SDC disables hold checks to the D or SI pins on the pseudo-DFFE flops (SCANMAP block in `notes.md`). A bug in this map (sync-set mapped as clear) was found in gate-level sim and fixed (commit "fuck fuck fuck fuck FUCK").
- Clocking structures: `hdl/clocks/clocks.v` with glitchless clock muxes (`hdl/cells/cell_clkmux_glitchless.v`) selecting padin_clk, x2/3 (`cell_clkdiv_3over2.v`), /2, or DCK for clk_sys and clk_audio independently; clock-root anchor buffers (`clkroot_anchor.v` instantiates `gf180mcu_fd_sc_mcu9t5v0__clkbuf_16`), false-path anchors, ICGs (`icgtn_4` x2 in the final netlist; clock gating was mostly removed with `NOCLOCKGATES`), DDR output cells, an async SRAM PHY with negedge flops. A planned programmable DCO ring oscillator is listed in the notes ("Maybe DCO ring osc") but not implemented (no oscillator in RTL or netlist).
- Chip ID: IDCODE 0x00280035 readable over TWD. Software: boot ROM, tests (`software/tests/{iram,eram,flash}`), cocotb gate-level and RTL sims.
- **Post-tapeout material (committed June-July 2026)**: `pcb/bringup/` is a KiCad bring-up board (RP2350 picoprobe-style debug probe tunnelling TWD, level shifters, 5 V/3.3 V/external supply by jumper, SRAM, SPI flash, LCD, socket for the standard wafer.space COB-to-socket PCB), with `SPEC.md` stating its purposes: smoke test, "sweep voltages and get coarse V/F sweeps", validate 3.3 V operation (supply 3.0 to 5.5 V). `doc/rb180_datasheet.adoc` is a datasheet with pinout. No silicon test results are in the repo (checked docs and notes; the bring-up board has been drawn but no measured data committed).
- Releases: `v1.0.rc1` (chip_top.gds 157 MB, pnl.v) and `v1.0-rc2` (chip_top.gds 122 MB, nl.v, pnl.v); no metrics, libs or reports are attached. Platform hash 2ac54634... matches neither release GDS (018f9b1f..., 2c3d7f0a...). The git history has a commit on 2025-12-18 (a98e9bd, ECO diodes on pad_DIO) after the 3 Dec deadline, so HEAD is not the submitted state.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Luke Wren | all RTL, flow tuning, PCB, docs (215 commits); author of Hazard3, RISCBoy, TwoWireDebug | not stated (GitHub: Cambridge, UK; no company) | git shortlog, copyright headers "Copyright (C) 2025 Luke Wren", `gh api users/Wren6991` |
| Leo Moser | template author, helped debug flow (82 commits) | notes.md calls him "Leo from wafer.space" | notes.md, AUTHORS.md |
| Tim Ansell, TrevorPeyton | 1 commit each | not stated | git shortlog |

## Value for GF180MCU characterisation and PDK development

### Verdict: Medium
A 250k-cell 9-track 5 V design with a documented intent to sweep supply voltage (3.0 to 5.5 V) and measure frequency, a timing-critical external-SRAM interface with documented STA, clock muxes/dividers that let an external clock be swept, and a custom scan-flop techmap. No purpose-built device or oscillator structures, so not High.

### Test structures and measurable circuits
| Structure | Exercises | Access | Where |
|---|---|---|---|
| Whole-chip V/F sweep | gf180mcu_fd_sc_mcu9t5v0 delay vs VDD (3.0 to 5.5 V); Hazard3 critical path (~1 ns slow-corner violation at 41.7 ns, later closed, per notes) | CLK pad (selectable x1, x2/3, x1/2) and TWD debug to run self-checks | `pcb/bringup/SPEC.md`, `hdl/clocks`, `librelane/chip_top.sdc` |
| 53 bi_t pads, SRAM bus (16 DQ, 17 A), 25 MHz | IO cell `gf180mcu_fd_io__bi_t` drive/timing, output delays to pad (STA shows 2.6 ns pad delay at 2.87 pF load) | SRAM_* pads | `hdl/cells/async_sram_phy_gf180mcu.v`, `notes.md` |
| 30 SRAM macros (28 x 512x8, 2 x 256x8) | foundry SRAM at 5 V and at reduced VDD; vendor model bugs noted | via CPU/APU/PPU, TWD | `hdl/mem/sram_wrapper.v` |
| Glitchless clock muxes + 3/2 divider | functional clock structures; padin_clk and DCK domain crossings | CLK pad, DCK pad | `hdl/cells/*.v`, `hdl/clocks/clocks.v` |
| Audio 1.5 MHz PWM out, 4-bit | IO toggling, noise | AUDIO pad | `hdl/apu/aout` |

Simulation/STA results are not in the repo (no metrics or libs); only netlists and layout in releases, plus STA excerpts in `notes.md`, so comparison needs a re-run of the flow.

### Candidate PDK additions
- Techmap `librelane/gf180_scanflop_map.v` (enable / sync-set / sync-reset flops onto scan cells), a candidate for the PDK/LibreLane flow as it improves density; also latch regfile experience (no active-low latch in the library; CTS gave each latch its own inverter).
- Documented PDK/flow problems (from `notes.md`): (1) Magic and KLayout DRC report many violations inside the foundry standard-cell library, which the author believes are false positives of the DRC decks, filed as wafer-space/gf180mcu-project-template issue 34, plus one DRC from a relative placement of `sdffrnq_4` and `nand3_4`; (2) the pad model `gf180mcu_fd_io__bi_*` does not model PU/PD (issue filed); (3) the SRAM simulation model needs an extra CSn edge before first access, does not initialise internal variables and cannot be used for RTL sims, no preload (to be filed on the GF180MCU repo); (4) sram macros carve holes in PDN (extra straps on edges), SRAM PDN connection issues; (5) CTS only runs when YAML clock flags are given, clock-gate inference clocks roots, resizer seldom upsizes; (6) OpenSTA lacks `set_ideal_network`, `-through` on `set_output_delay`; (7) low default file-handle limit breaks Magic stream-out XOR; (8) KLayout DRC takes ~3 h and ~20 GB; (9) DRT crash on MacOS and KLayout-vs-Magic XOR on MacOS (to file).

### What would be needed to use it
Packaged or COB-on-socket part, the bring-up board in `pcb/bringup` (design only; no evidence it was built), a 5 V supply that can be swept and a clock source; TWD probe firmware is not in this repo (hacked picoprobe is referenced). Measurement of Fmax would use the debug-run software tests in `software/tests`.

## Notes and caveats

- 18 submodules (Hazard3 and its test suites, RISCBoy, libfpga, TwoWireDebug, fpgascripts); several nested ones for tests only; all cloned OK.
- `CLOCK_PERIOD: 20` in `config.yaml` is for the pad clock; clk_sys is constrained in the SDC (the notes say 41.67 ns, 24 MHz; datasheet says 25 MHz).
- The notes state timing was "within a nanosecond" at the slow corner on Day 13, then reported clean after the register-file change; no final STA report is shipped to confirm.
- The repo extends past the tapeout (PCB work to July 2026), so HEAD does not equal the submitted layout; neither release GDS hash matches the platform sha (inferred: the platform file was post-processed, or a later commit).
