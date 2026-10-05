# GD02: Racquet Half r1p0 (ws-run1)

| | |
|---|---|
| Repository | https://github.com/gregdavill/gf180mcu-racquet-0.5x1 @ 8c1de9c (develop, last commit 2025-12-17) |
| Slot | 0p5x1 (half slot, die 1936 x 5122 um) |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 (LICENSE file; no SPDX claim in the sources checked) |
| README | TEMPLATE: 100% identical to the wafer-space project-template README, 0 words of own text |

## What the repo actually contains

A multi-core SoC built around SERV (the bit-serial RISC-V core) via Olof Kindgren's `servile`/`subservient` wrappers (submodules `ip/serv` and `ip/subservient`). The platform description says "9 cores connected to local SRAM"; the sources agree (`localparam NUM_PROC_CORES = 9` in `src/chip_core.sv`).

- Each core has its own `gf180mcu_fd_ip_sram__sram512x8m8wm1` (512 x 8 bit, foundry 5 V macro) used as RAM + register file + CSR store (`src/proc_wrapper.sv`). Nine SRAM instances in the release netlist.
- Cores are loaded at boot from SPI flash through a loader chain (`src/spi.sv`, loader_wdata/valid/ready arrays in `chip_core.sv`).
- Shared Wishbone crossbar (`src/wb_xbar.sv`) to six responders: GPIO (`wb_gpio.sv`), UART (`wb_uart.sv`), timer (`wb_timer.sv`), a queue peripheral for synchronisation and core-to-core messages (`wb_queue.sv`), etc. All hand-written SystemVerilog by the authors (about 2700 lines in `src/`).
- Flow: LibreLane "Chip" flow (`librelane/config.yaml`, slot file `librelane/slots/slot_0p5x1.yaml`), Yosys with the slang plugin, 5 V 7-track standard cells `gf180mcu_fd_sc_mcu7t5v0` (confirmed from the release netlist: 112,820 cells, all 7t5v0), IO cells `gf180mcu_fd_io__bi_24t` x54 (all pads bidirectional), 1 in_c, 1 in_s, wafer.space `gf180mcu_ws_io__dvdd/dvss` x8 each. No analog macros. Chip ID and logo macros as in the template, plus a `gf180mcu_gsd__logo`.
- Clock: single `clk_PAD`, `CLOCK_PERIOD: 100` (10 MHz). No PLL/oscillator.
- `cocotb/` has a single testbench (`chip_top_tb.py`) that assembles small RISC-V programs; no silicon bring-up data.
- Two releases: `release-r1p0` (commit d35af25, 2025-12-03, "Bump GPIOB_NUM to 22") and `release-r1p1` (HEAD 8c1de9c, 2025-12-17, "Increase halos"). `releases/` holds gds.gz plus a tar with lib/sdf/spef/def/odb/netlists; r1p1 tar also has `metrics.json` and SPICE. The project.json hash (5d08605b...) matches neither release GDS (sha256 of the decompressed files 271f7a7d... and 25a55fba...).
- Release r1p1 metrics (from `metrics.json`): 113,682 instances, die 1936 x 5122, stdcell utilisation 61%, setup WNS +16.6 ns (max_ss_125C_4v50), +56.8 ns nom_tt, hold WNS +0.14 ns worst (min_ff), 0 setup/hold violations in all nine corners, Magic DRC 0, KLayout DRC 0, LVS 0, antenna 0, XOR 0.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Greg Davill | Racquet design (77 of 157 commits, SoC RTL and flow config) | not stated (GitHub profile gives name, location Australia, blog gregdavill.com; no company) | git shortlog, `gh api users/gregdavill` |
| Leo Moser | Project-template author (78 commits are template/flow updates) | not stated in the repo (AUTHORS.md lists name only) | AUTHORS.md, git shortlog |
| Tim Ansell, TrevorPeyton | one commit each (template) | not stated | git shortlog |
| Olof Kindgren | SERV / subservient / servile (submodules) | not stated here | submodule URLs |

## Value for GF180MCU characterisation and PDK development

### Verdict: Low
Ordinary synthesised digital on stock 7t 5 V cells and the foundry SRAM macro; no test structures, no scan, no oscillator. Useful as a yield/functional data point and as a timing-closure reference (clean in all nine corners).

### Test structures and measurable circuits
| Structure | Exercises | Access | Where |
|---|---|---|---|
| 9 x sram512x8m8wm1 | foundry SRAM function/speed at 5 V (functional only) | via SERV cores, SPI loader, GPIO/UART | `src/proc_wrapper.sv`, `librelane/config.yaml` |
| 10 MHz synchronous digital (~113k cells) | functional yield; STA numbers in release `metrics.json` and `sdf/` for 9 corners | pads | releases/release-r1p1 |

Pre-silicon STA/SDF data exist (nine corners, SPEF min/nom/max) so silicon Fmax could be compared, but nothing is built to measure Fmax directly (clock only from pad).

### Candidate PDK additions
Nothing found. Config carries template workarounds (Metal2 filler ignoring active metal, `MAGIC_GDS_FLATGLOB` for SRAM/IO cells, halo workaround for OpenROAD issue 8868 in the GD04 sibling).

### What would be needed to use it
Packaged or COB part with 10 MHz clock, reset, SPI flash with firmware (or a fake flash driven by a probe). No test plan or firmware in the repo.

## Notes and caveats

- Relation to siblings: GD02, GD03 (gf180mcu-racquet) and GD04 (gf180mcu-racquet-1x0.5) are three forks of the same Racquet code base with identical `src/` apart from core count, slot defines and small RTL fixes. GD02 differs from GD03 by: 9 cores vs 23; 0p5x1 slot pad counts (54 bidir, no analog); async-reset synchroniser in `chip_top.sv`; an extra registered SRAM-read enable (`sram_ren_reg`) and `sram_cen = ~(wen | ren | ren_reg)` in `proc_wrapper.sv` (GD03 uses `sram_cen = proc_rst`); `ERROR_ON_MAGIC_DRC: False` removed; larger macro halos. GD02 is the only one with the 0p5x1 slot defines changed to 54 bidir pads.
- project.json says "r1p0", but repo HEAD is the later r1p1 (2025-12-17, after the 3 Dec run-1 deadline noted by sibling projects). The submitted layout is not identifiable from the repo; neither release GDS hash matches the platform sha.
- README is the untouched template, so the only description is project.json and the sources.
