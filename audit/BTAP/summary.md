# BTAP: BreakingTTAPs (ws-run1)

| | |
|---|---|
| Repository | https://github.com/polyfractal/BreakingTTAPs @ 8f357d6 (main, last commit 2025-12-22) |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 at the root; `src/LICENSE` is CERN-OHL-S v2 |
| README | TEMPLATE (99% the project-template README; only a one-line title is the author's) |

## What the repo actually contains

A transport-triggered-architecture processor. `src/chip_core.sv` is about 19,300 lines of generated Verilog with modules `tta::alu`, `cmp`, `lsu`, `uart`, `spi`, `gpo16`/`gpi16`, `modadd` and `pc`.

Built with the wafer.space template on `gf180mcu_fd_sc_mcu9t5v0` cells with 28 foundry `gf180mcu_fd_ip_sram__sram512x8m8wm1` macros (placed by `sram_generated.tcl` and `scripts/macro_floorplan.toml`) at a 55 ns clock.

The final GDS is in the repo as a zip (`final/gds/chip_top.zip`, 175 MB) with checksums. `gf180mcu_bt_ip__tap` and a logo are Metal5 artwork only. Late-December commits show ALU shift bug fixes and DRC fixes.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Zachary Tong (GitHub polyfractal) | designer | not stated (GitHub location: Vermont; a breakingtaps.com email domain is a hint only) | git log, GitHub profile |
| Leo Moser (mole99) | project template | not stated | git log |

## Value for GF180MCU characterisation and PDK development

### Verdict: Low

Ordinary synthesised digital on stock cells with foundry SRAM; a functional and yield data point.

### Test structures and measurable circuits

Nothing purpose-built. 28 foundry SRAM macros are exercised functionally.

### Candidate PDK additions

None. The author's notes cover PVT, slew and antenna fixes (resizer margins, an ECO diode, a reference to OpenROAD bug 8229).

### What would be needed to use it

Packaged die and a host for UART/SPI. No test plan in the repo.

## Notes and caveats

- The repo carries two licences: Apache-2.0 at the root (from the template) and CERN-OHL-S v2 for `src/`. Which one the platform claim matches needs checking.
- The README does not describe the design at all.
