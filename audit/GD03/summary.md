# GD03: Racquet r2p0 - 23 core SoC (ws-run1)

| | |
|---|---|
| Repository | https://github.com/gregdavill/gf180mcu-racquet @ 5ef4901 (develop, last commit 2025-12-10) |
| Slot | 1x1 (full slot, die 3932 x 5122 um) |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 (LICENSE file) |
| README | TEMPLATE: 100% identical to the wafer-space project-template README, 0 words of own text |

## What the repo actually contains

The largest Racquet configuration: `NUM_PROC_CORES = 23` SERV cores (`src/chip_core.sv`), each with its own `gf180mcu_fd_ip_sram__sram512x8m8wm1` (23 instances in the release netlist, placed in a 5-column grid by `librelane/config.yaml`), loaded from SPI flash at boot, sharing a Wishbone crossbar with GPIO, UARTs, timers and a queue peripheral for core-to-core synchronisation/messages. The platform text says "a slew of peripherals: uarts, timers and more uarts"; sources agree (`wb_uart.sv`, `wb_timer.sv`, `wb_gpio.sv`, `wb_queue.sv`, `wb_xbar.sv`, `spi.sv`, `racquet.sv`, `proc_wrapper.sv`).

- Flow: LibreLane Chip flow, slang, 5 V 7-track cells `gf180mcu_fd_sc_mcu7t5v0` (319,940 cells in the r2p1 netlist), IO: 54 `gf180mcu_fd_io__bi_24t`, 1 `in_c`, 1 `in_s`, `gf180mcu_ws_io__dvdd` x8 / `dvss` x10, plus ID/logo/GsD logo macros. No analog. Clock `clk_PAD`, 100 ns period (10 MHz).
- Tags/releases: `release-r1p0` (d5ca5da, 2025-12-03), `release-r2p0` (2bd1e60, 2025-12-04, "Spread out srams"), `release-r2p1` (= HEAD 5ef4901, 2025-12-10), `release-r1p1` (300f95d, on branch `rev1`, 2025-12-18, merged tag 1.3.5). `releases/` has gds.gz for each; r1p1 and r2p1 tars contain lib/sdf/spef/odb/netlists and `metrics.json`; r1p0/r2p0 are 7z archives.
- Release r2p1 metrics: 321,106 instances, instance area 18.3 mm2, stdcell utilisation 36.5%, DRC (Magic and KLayout) 0, LVS 0, antenna 0, XOR 0. Timing: nominal tt 25C 5.0 V setup WNS +41.8 ns, ff +63 ns, hold clean everywhere, but **setup is violated at the slow corner**: nom_ss_125C_4v50 WNS -5.75 ns (130 endpoints), max_ss_125C_4v50 WNS -11.3 ns (413 endpoints), overall 548 violating endpoints against a 100 ns clock. `ERROR_ON_MAGIC_DRC: False` is set in config.
- Compared with GD02, config raises resizer slack margins (0.3 setup, 0.35/0.3 hold) and the core-ring width to 25 um.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Greg Davill | Racquet RTL and config (68 commits) | not stated (GitHub: Australia, blog gregdavill.com) | git shortlog, `gh api users/gregdavill` |
| Leo Moser | template author (78 commits) | not stated in repo | AUTHORS.md, git shortlog |
| Tim Ansell, TrevorPeyton | 1 commit each | not stated | git shortlog |
| Olof Kindgren | SERV/subservient (submodules) | not stated | submodule URLs |

## Value for GF180MCU characterisation and PDK development

### Verdict: Low
Big synthesised 7t 5 V digital design plus 23 foundry SRAM macros; no test structures. Its use is as a large yield/functional sample (321k cells, 23 SRAMs) and a slow-corner timing data point (the design is expected to run at 10 MHz at tt but does not close at ss/125C/4.5 V by 5 to 11 ns, so silicon Fmax at temperature/voltage would test that slow-corner margin).

### Test structures and measurable circuits
| Structure | Exercises | Access | Where |
|---|---|---|---|
| 23 x sram512x8m8wm1 | foundry SRAM at 5 V, functional yield across 23 instances | per-core SERV firmware via SPI flash load, UART/GPIO | `src/proc_wrapper.sv`, `librelane/config.yaml` |
| ~320k-cell 10 MHz logic | Fmax vs. STA (9-corner SDF/SPEF in release) | pads only; clock from `clk_PAD` | releases/release-r2p1 |

STA predictions exist for comparison. No on-chip delay monitor.

### Candidate PDK additions
Nothing found.

### What would be needed to use it
Packaged part, adjustable clock and supply (4.5 to 5.5 V, temperature) to find where it fails; flash image or probe emulating SPI flash; a program per core. No firmware or test plan in repo.

## Notes and caveats

- Which commit was submitted is not certain: project.json says "r2p0", the repo HEAD is r2p1 (2025-12-10, a day before the sibling GD04 release), the platform hash (1c7d0b7d...) matches none of the four release GDS files (r1p0 f4f49f40..., r1p1 7c9cb8f7..., r2p0 73f56b74..., r2p1 1b17eb0d...). The r1p1 release comes from a separate `rev1` branch not checked out here.
- The r2p1 metrics show real slow-corner setup violations; a submission with those would still be fine functionally at tt but is not signed off at ss.
- Relation to GD02/GD04: see GD02 summary. Same code base; GD03 is the 23-core 1x1 build with no scan chain.
