# MOLE: FABulous FPGA (ws-run1)

| | |
|---|---|
| Repository | https://github.com/mole99/gf180mcu-fabulous-fpga @ 97da81d (main, last commit 2025-12-16) |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 |
| README | TEMPLATE+NOTE (84% the project-template README plus about 120 words of own text) |

## What the repo actually contains

An embedded FPGA generated with the FABulous framework: 480 LUT4, 12 register-file tiles, 6 MAC tiles and 6 foundry 5 V 512x8 SRAM tiles, with 48 programmable IOs. It is configured over SPI (passive) or from one of 16 slots of an external SPI flash (active).

The fabric is assembled from individually hardened tile macros. The fabric macro itself contains zero standard cells (112 macros); the tiles hold about 4,202 `gf180mcu_fd_sc_mcu7t5v0` cells. Top-level clock target is 40 ns; tiles are hardened at 20 ns. The flow uses a custom LibreLane branch (`leo/gf180mcu`) and PDK tag 1.6.6.

Fabric macro metrics in the repo: 0 DRC, 0 LVS differences, 0 antenna violations. The tape-out GDS is only in the release (135 MB, in `../releases/`); there are no final views or top-level STA reports in git.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Leo Moser (GitHub mole99) | sole author (136 of 138 commits) | not stated (GitHub location: Austria) | git shortlog, AUTHORS.md, GitHub profile |

## Value for GF180MCU characterisation and PDK development

### Verdict: Medium

Not built for characterisation, but an FPGA is a reconfigurable test vehicle: ring oscillators and carry-chain delay lines can be programmed into the LUT4AB tiles after fabrication (`break_comb_loop` in `custom.v` preserves combinational loops) and brought out on any of 48 pads. What limits it is that no oscillator design, test plan, or GF180 timing model for FABulous exists in the repo.

### Test structures and measurable circuits

| Structure | Exercises | Access | Where |
|---|---|---|---|
| Post-fab programmable ring oscillators / delay lines in LUT4AB tiles | 7t5v0 cell and routing-mux delay | any of 48 pads; per-pad CS/SL/IE/PU/PD driven from the fabric | tile sources, `custom.v` |
| 6 SRAM tiles (foundry 512x8, 5 V) | SRAM macro function and access time | from fabric | SRAM tile |
| 6 MAC tiles | arithmetic path delay | from fabric | MAC tile |
| IO pad configuration from fabric | `gf180mcu_fd_io` drive strength, slew, pulls | pads | top level |

No simulation results suitable for comparison with silicon are in the repo (the top-level standard-cell library is inferred to be the template default, 7t5v0).

### Candidate PDK additions

None as cells. By-products: a GF180 latch map (`latq_1`), a tile-macro flow, and a MAGIC_GDS_FLATGLOB list for false DRC hits in the foundry SRAM and IO cells. Documented PDK/flow issues: Metal2 minimum density unreachable (worked around with `Metal2_ignore_active`), `ERROR_ON_MAGIC_DRC: False`, and OpenROAD requiring standard-cell rows even for a macro-only fabric.

### What would be needed to use it

Packaged or bonded die, SPI configuration, and a Yosys/nextpnr flow targeting this fabric. Someone has to write the oscillator/delay-line bitstreams and a timing model to compare against.

## Notes and caveats

- Tape-out GDS exists only as a release asset; it does not match the manufactured layout hash (the platform hash is of the precheck output).
- No bring-up data in the repo.
- The run 2 successor is `../../ws-run2/MOLE`.
