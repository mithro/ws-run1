# TZ01: TillitisZedulo-testchip2025 (ws-run1)

| | |
|---|---|
| Repository | https://github.com/ZeduloTech/gf180mcu-testchip2025 @ 1d1f91d (main, last commit 2026-06-17) |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 (`LICENSE`); foundry SRAM models carry GlobalFoundries Apache-2.0 headers |
| README | OK (1491 words): IP list, address map, eFuse variants, UART2GPI protocol, test list, full pad table |

## What the repo actually contains

A 1x1 test chip built on a Caravel port for wafer.space (VexRiscv management core) whose main purpose is to put several sizes and interface styles of eFuse memory on silicon, together with the foundry SRAMs behind a test controller and a Zedulo peripheral macro. The README states the goal as "to test the flow and several open-source IPs".

Sub-designs:

| Block | What it is | Where |
|---|---|---|
| `caravel_core` | Caravel management SoC (VexRiscv, housekeeping SPI, digital PLL with `ring_osc2x13` DCO, `simple_por`, 2 kB RAM from 4x `gf180_ram_512x8_wrapper`), hardened separately as a macro | `caravel/verilog/`, `caravel/librelane/`, `caravel/final/` (GDS, LEF, lib, pnl, spef) |
| `efuse_wb_mem_1024x32`, `_64x32`, `_128x8`, `_32x8` | Synchronous eFuse memories with Wishbone interface at 0x3000_0000, 0x3000_1000, 0x3000_2000, 0x3000_3000 | `ip/efuse_wb_mem_*/` (GDS, LEF, `.nl.v`, `.pnl.v`, behavioural array model, 9 corner dirs with `.lib` and `.sdf`) |
| `efuse_spi_mem_256x8` | eFuse memory with a 25-series-EEPROM-style SPI interface, on pads | `ip/efuse_spi_mem_256x8/` |
| `efuse_async_mem_1x8` x5 | One-byte asynchronous eFuse, read and latched once at reset; four are mapped as one 32-bit Wishbone word at 0x3000_4000, one is wired entirely to pads | `ip/efuse_async_mem_1x8/`, `src/wb_async_efuse.v` |
| `uspi_sramtest` + 4 SRAM wrappers | Zedulo micro-SPI controller (commands for CEN, write mask, address, read/write with auto-increment) driving `gf180_ram_64x8_wrapper`, `_128x8_`, `_256x8_`, `_512x8_` | `src/sramtest/uspi_sramtest.sv`, `ip/sram/` |
| `uart2gpi` | Zedulo UART-to-peripheral bridge with OpenTitan-derived UART, GPIO, PWM, I2C; hardened as a macro | `uart2gpi/src/`, `uart2gpi/final/` (complete LibreLane views and `metrics.csv`) |
| Glue | Wishbone switch/mux, pad configuration | `src/chip_core.sv`, `src/wb_efuses.v`, `src/wb_switch.v`, `src/wb_mux_5.v`, `src/pinout.vh` |

Flow: LibreLane "Chip" flow (`librelane/config.yaml`, nix shell with the `leo/gf180mcu` LibreLane branch), `gf180mcu_fd_sc_mcu7t5v0` standard cells, `gf180mcu_fd_io` pads, single 5 V domain (corners tt_025C_5v00, ss_125C_4v50, ff_n40C_5v50), 45 ns clock period (22 MHz). Hierarchical: Caravel core, `ring_osc2x13`, `uart2gpi` and the eFuse blocks are pre-hardened macros.

Final outputs in the repo: `final/gds/chip_top.gds.gz` (commit 11bf153, 2025-12-15: "Update the GDS. Should be clear for production."), `final/nl/chip_top.nl.v`, `final/pnl/chip_top.pnl.v`. No top-level DRC/LVS/timing reports or metrics are committed; `uart2gpi/final/metrics.csv` is (0 routing DRC errors, 0 antenna violations, 8772 max-slew violations summed over corners).

Verification: `cocotb/chip_top_tb.py` with 13 tests listed in `cocotb/test_list.json` (eFuse read/write from the CPU, async eFuse, housekeeping SPI, PLL, UART, self-sufficient boot from eFuse, SPI eFuse used as boot flash for a second chip instance, SPI SRAM, UART2GPI), run on RTL and gate level in CI. Caravel-side programs and Verilog benches in `caravel/sim/caravel_sw/` and `caravel/sim/caravel_tb/`. No simulation logs committed.

Post-silicon: the last commit (1d1f91d, 2026-06-17, "Add WB efuse bringup test") adds `caravel/sim/caravel_sw/efuse_rw_bringup.c`, described in its header as an "eFuse Wishbone memory read-write test for bringup (without the largest eFuse and with UART output)". It covers the 64x32, 128x8 and 32x8 blocks. No measured results are in the repo.

Where the new IP comes from (no submodules; everything is vendored as hard views):

- eFuse macros: generated with https://github.com/egorxe/gf180_efuse_compiler (README). GDS cell names show a custom array (`efuse_cell`, `efuse_bitline`, `efuse_bitline_block_*`, `efuse_senseamp`, plain `nmos`/`pmos` pcells) surrounded by `gf180mcu_fd_sc_mcu7t5v0` control logic. No schematic, SPICE netlist or generator source for the array is in this repo.
- SRAM: `ip/sram/gf180_ram_*_wrapper.gds.gz` contain the stock foundry macros `gf180mcu_fd_ip_sram__sram64x8m8wm1`, `__sram128x8m8wm1`, `__sram256x8m8wm1`, `__sram512x8m8wm1` (5 V) inside a wrapper cell that adds via/power connections; commit ca583b7 says the wrappers are from https://github.com/egorxe/gf180mcu-sram-wrappers. These are not new SRAM designs.
- Caravel port: https://github.com/egorxe/gf180mcu-caravel-template.
- `ip/simple_por/`: the Open Circuit Design / Efabless 5 V POR (GDS, LEF, netlist, testbenches), unchanged as far as file names show.

## Authors and affiliations

| Person / organisation | Role | Affiliation | Source |
|---|---|---|---|
| egorxe / Egor Lukyanchenko | Caravel port, eFuse compiler and all eFuse blocks, Wishbone glue, cocotb tests, final GDS (52 commits) | not stated (GitHub profile empty) | git log; `AUTHORS.md`; SPDX "© 2025 egorxe"; README |
| Zedulo (GitHub org `ZeduloTech`; committers "Zedulo", "Zedulo.com", `snapaa257`) | SRAM SPI test controller, UART2GPI macro, SRAM integration (18 commits) | Zedulo, Sweden, zedulo.com | git log; `AUTHORS.md`; "Copyright (c) 2025 Zedulo" header; `gh api orgs/ZeduloTech` |
| Leo Moser (`mole99`) | wafer.space project template that the repo is forked from (74 template commits) | not stated (GitHub location Austria) | git log; `AUTHORS.md` |
| Tillitis | "support" | not stated in the repo | platform description and project name only |
| GlobalFoundries PDK Authors | SRAM macros and models | GlobalFoundries | headers in `ip/sram/*.v` |
| Tim 'mithro' Ansell, TrevorPeyton | one template commit each | not stated | git log |

## Value for GF180MCU characterisation and PDK development

### Verdict: High

It is a memory test chip for a macro type the open PDK does not have: eleven eFuse arrays in five sizes and three interface styles, one of them with every signal on a pad, plus four foundry SRAM sizes with external SPI access.

### Test structures and measurable circuits

| Structure | What it exercises | Access (pad numbers from the README table) | Where |
|---|---|---|---|
| `efuse_async_mem_1x8` (pad instance) | Programming of single eFuse bits (program current/time, blown and unblown resistance as seen by the sense amplifier), sense margin versus VDD, read-at-reset behaviour | `aef_prog_0..7` inputs (58-61, 64-67), `aef_out_0..7` (32, 37-43), `aef_ready` (31), `aef_reset` (68) | `ip/efuse_async_mem_1x8/`, `src/chip_core.sv` |
| `efuse_spi_mem_256x8` | 2 kbit eFuse array, program/read through SPI without the CPU | SPI clock 51, MOSI 53, MISO 52 (muxed with SRAM), CS 55 | `ip/efuse_spi_mem_256x8/` |
| `efuse_wb_mem_1024x32` (32 kbit), `_64x32`, `_128x8`, `_32x8`; 4x async on Wishbone | Array-size scaling of bit line and sense amplifier, programming yield over about 39 kbit in total | Through Caravel firmware (flash 19-22, UART 7/9, housekeeping SPI 3-6) | `ip/efuse_wb_mem_*/`, `src/wb_efuses.v`, `caravel/sim/caravel_sw/efuse_rw*.c` |
| Foundry SRAM 64x8, 128x8, 256x8, 512x8 (`gf180mcu_fd_ip_sram`, 5 V) | Function and bit errors of the stock 5 V macros; access time only coarsely, since the SPI controller is in the path | SPI: CS 50, clock 51, MOSI 53, MISO 52, debug 49 | `src/sramtest/uspi_sramtest.sv`, `ip/sram/` |
| `ring_osc2x13` DCO + digital PLL in Caravel | `gf180mcu_fd_sc_mcu7t5v0` gate delay at 5 V | Clock feedback outputs on `caravel_io_7/8` (pads 10, 11), control via housekeeping SPI | `caravel/ring_osc2x13/` (hardened macro with lib/sdf/spef), `caravel/verilog/digital_pll*.v` |
| `simple_por` | 5 V POR delay | Indirect (`npor` inside Caravel) | `ip/simple_por/` |
| Whole chip | Functional/yield data point for a 5 V synthesised SoC at 22 MHz | all | `final/` |

All circuits are in the single 5 V domain with `gf180mcu_fd_io` pads. The PDK device type used for the fuse element could not be confirmed here because no netlist of `efuse_cell` is in the repo.

Simulation data to compare silicon against: per-macro `.lib` and `.sdf` at nine corners for every eFuse block, behavioural eFuse models with timing assertions (`MIN_PRESET_NS = 1`, `MIN_SENSE_NS = 3`, `MIN_WRITE_NS = 1000` in `efuse_async_array_1x8.v`), the Verilator bench for the SRAM tester (`src/sramtest/sim/`), and the cocotb suite. No analog (SPICE) simulation of the fuse cell or sense amplifier is in this repo; it would be in the eFuse compiler repository.

### Candidate PDK additions

- An eFuse / OTP memory compiler and macro family for GF180MCU (`efuse_cell`, bit line, sense amplifier, Wishbone/SPI/async wrappers). This is the main upstream candidate; source at https://github.com/egorxe/gf180_efuse_compiler.
- SRAM wrapper cells that make the foundry `gf180mcu_fd_ip_sram` macros usable in LibreLane (https://github.com/egorxe/gf180mcu-sram-wrappers), with per-size `.lib` files.
- A Caravel management-core port for the wafer.space template, including a "self-sufficient" boot mode from eFuse (`caravel_start_mode` pad 69).
- Workarounds visible in `librelane/config.yaml`: `MAGIC_GDS_FLATGLOB` list for the foundry SRAM (cells that must be flattened "to get rid of 3.3V devices (DUALGATE drawn into high-level cells)"), `Metal2_ignore_active` for fill density, `KLayout.Antenna` check disabled, `Checker.YosysSynthChecks` disabled ("Fails due to multisources in PLL"), delay cells excluded from P&R (`caravel/librelane/pnr_exclude.cells`: "they degrade timing").

### What would be needed to use it

Packaged or chip-on-board parts; the README pad numbering is stated to be compatible with the wafer.space COB project. 5 V supply, clock on pad 0, reset on pad 1. For the pad-level async eFuse: a pattern generator and a way to measure supply current during programming. For the SPI eFuse and SPI SRAM: any SPI host. For the Wishbone eFuses: SPI flash with the firmware in `caravel/sim/caravel_sw/` and a UART. Missing: eFuse programming conditions (voltage, pulse width, current) and expected resistance values, the SPI SRAM command documentation (README section "SPI SRAM" is "TODO"; the command set is only in the header comment of `uspi_sramtest.sv`), a written bring-up procedure and any bring-up results.

## Notes and caveats

- The platform describes an "eFUSE and SRAM testchip"; the SRAM part is the unmodified foundry 5 V macro set, so the new-library content is the eFuse.
- The transistor-level eFuse design is not reviewable from this repo (hard views only); the compiler commit used is not recorded.
- The bring-up firmware deliberately omits the 1024x32 block ("without the largest eFuse") with no reason given; this may simply be to keep the test short, or may point to a problem with that block (not stated).
- `ip/efuse_wb_mem_128x8/` contains a stray `efuse_wb_mem_512x32.nl.v`.
- Tillitis appears only in the platform name and description, not in the repo.
- The platform sha256 (`621d3fa6...`) matches neither `final/gds/chip_top.gds.gz` (`00b6df76...`) nor its uncompressed content (`00fefb9a...`), so I could not confirm that the committed GDS is the manufactured one (the wafer.space ID cell is replaced at tape-out, which alone would change the hash; inferred).
- No top-level sign-off reports are committed; DRC/LVS status rests on the commit message "Should be clear for production".
