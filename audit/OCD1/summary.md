# OCD1: openframe_caravel_picorv32 (ws-run1)

| | |
|---|---|
| Repository | https://github.com/RTimothyEdwards/gf180mcu_ocd_openframe @ 9ef4567 (main, last commit 2025-11-30). The taped-out design is **not** on main: it is on branch `caravel_picorv32` @ 349e17e (last commit 2026-01-02) |
| Slot | 1x1 |
| Top cell | caravel_openframe_top |
| Licence in repo | No LICENSE file; Apache-2.0 SPDX headers in sources (copyright Open Circuit Design, LLC; some files derived from Efabless Caravel; picorv32/picosoc are ISC) |
| README | Missing at top level; documentation is `magic/README` on main and `doc/README` on the `caravel_picorv32` branch |

## What the repo actually contains

A GF180MCU port of the Efabless Caravel "openframe" harness: a fixed padframe with 44 GPIO, a reset pad, a power-on-reset, a via-programmed 32-bit user-ID ROM and a user area. `verilog/rtl/gf180mcu_padframe.v` describes it as "a reference design for use of the 3.3V-enabled libraries for the gf180mcu open PDK".

The repository has two very different branches:

- **`main` (the checked-out HEAD, 3 commits, Nov 2025):** only the empty openframe template. Padframe layout in magic (`magic/caravel_openframe.mag`, `gf180mcu_padframe.mag`), an empty `openframe_project_wrapper`, `gds/caravel_openframe.gds.gz` (9.5 MB), top-level LVS output (`lvs/`), the `simple_por` macro (`ip/simple_por/`). There is no cell called `caravel_openframe_top` on main.
- **`caravel_picorv32` (6 further commits, 2025-12-09 to 2026-01-02):** the chip that was submitted. `doc/hashes.txt` on that branch says "URL (note that this is not in the main branch)" and names top cell `caravel_openframe_top` and file `gds/caravel_openframe_filled.gds.gz` (55 MB). Everything below refers to this branch (read with `git show origin/caravel_picorv32:<path>`).

Contents of the `caravel_picorv32` branch:

- `verilog/rtl/`: padframe (`gf180mcu_padframe.v`), `caravel_openframe.v`, and the user project: `picosoc.v` + `picorv32.v` (PicoRV32 SoC adapted from the sky130 Caravel openframe example), `mem_wb.v`, `housekeeping*.v`, `spimemio.v`, `simpleuart.v`, `simple_spi_master.v`, two counter/timers, `gpio_wb.v`, `gpio_vector_wb.v`, `dll.v` + `dll_controller.v` + `ring_osc2x13.v`, `clock_div.v`, `clock_routing.v`, `debug_regs.v`.
- `librelane/`: LibreLane "Classic" flow config for `openframe_project_wrapper` only (die area 350..3529.9 x 350..4719.9 um, clock on `gpio_in[38]`, 15 ns period, 20 % target density). `STD_CELL_LIBRARY: gf180mcu_as_sc_mcu7t3v3` (Avalon Semiconductor 3.3 V 7-track cells). STA corners tt_025C_3v30, ss_125C_3v00, ss_n40C_3v00, ff_n40C_3v60, ff_125C_3v60; the ss corners list the tt standard-cell lib (no ss lib is referenced). LibreLane run directories are deliberately not committed (`librelane/README`).
- Macros in the wrapper: 4x `gf180mcu_ocd_ip_sram__sram512x8m8wm1` and 4x `gf180mcu_ocd_ip_sram__sram256x8m8wm1` (3 kB of CPU RAM: 512x32 + 256x32; `mem_wb.v` comment: "Could use more memory here but want to exercise both macros"), and the hand-placed `ring_osc2x13` (`ip/ring_osc2x13/`, magic layout, 13 trimmable stages built from `gf180mcu_as_sc_mcu7t3v3` inv/invz/clkbuff cells; its README says it must be manually generated so stage delays are not left to P&R).
- Padframe: `gf180mcu_ocd_io` cells `__bi_a` (44 GPIO with OE/IE/schmitt/slew/PU/PD/2-bit drive select), `__in_s` (reset), `__dvdd`/`__dvss` (5 V), `__vdd`/`__vss` (3.3 V), `__cor`, and three custom 10 um filler variants `gf180mcu_ocd_io__fill10x/y/z` that embed a `constant_block` (loopback 1/0 per pad) and a `lvlshift_down` cell (two `gf180mcu_fd_sc_mcu7t5v0` inverters used as a 5 V to 3.3 V level shifter for the POR output).
- `ip/simple_por/`: 5 V-domain analog POR (see table below), with hand-written SPICE netlist and ngspice testbenches.
- `ip/wafer_space/`: seal ring, ID and logo cells.
- `gds/`: `caravel_openframe_top.gds.gz` (43 MB, pre-fill) and `caravel_openframe_filled.gds.gz` (55 MB), `hashes.txt`.
- `validate/`: netgen LVS output `caravel_openframe_comp.out` ending "Final result: Circuits match uniquely." and KLayout DRC report `caravel_openframe_drc_klayout.lyrdb` with zero items. `netlist/layout/caravel_openframe.spice` (35 MB extracted netlist). `verilog/gl/openframe_project_wrapper.pnl.v` (65 MB powered gate-level netlist).
- `verilog/dv/`: cocotb tests copied from Caravel (hello_world, UART, SPI master, timers, housekeeping, GPIO vectors) and one iverilog test `gpio_vector`. No simulation logs or results are committed.

None of the three libraries under test is in the repo or a submodule (there are no submodules). They are read from the installed PDK (`pdk_dir::libs.ref/gf180mcu_ocd_io`, `.../gf180mcu_ocd_ip_sram`, `.../gf180mcu_as_sc_mcu7t3v3`), i.e. the versions that `open_pdks` installs for gf180mcuD. `gf180mcu_padframe.v` states that `gf180mcu_ocd_io` is "distributed with the GF180MCU open PDK by the open_pdks installer". The upstream source repositories for the three libraries are not named in this repo.

No timing reports, no silicon bring-up data and no test firmware beyond the Caravel DV programs are present.

## Authors and affiliations

| Person / organisation | Role | Affiliation | Source |
|---|---|---|---|
| R. Timothy Edwards (GitHub `RTimothyEdwards`) | Sole committer (9 commits across both branches); padframe, POR, ring oscillator, flow, verification | Open Circuit Design, LLC | git log; "Written by Tim Edwards" and "SPDX-FileCopyrightText: 2025 Open Circuit Design, LLC" headers; GitHub profile company "Open Circuit Design" |
| Efabless Corporation | Original Caravel / openframe RTL, POR concept, DV tests | Efabless | "SPDX-FileCopyrightText: 2020 Efabless Corporation" headers |
| picorv32 / picosoc authors | CPU core | not stated | ISC licence headers in `picorv32.v`, `picosoc.v` |
| Avalon Semiconductor | 3.3 V standard cell library used (not in repo) | not stated | library name; `librelane/config.yaml` comment |

## Value for GF180MCU characterisation and PDK development

### Verdict: Medium

It is not a characterisation chip, but it is a full-chip use "in anger" of three new 3.3 V libraries (dual-voltage IO, 3.3 V SRAM, 3.3 V standard cells), with a pad-observable ring oscillator/DLL, a mask-ID ROM and an analog POR, and it has clean LVS and KLayout DRC reports in the repo.

### Test structures and measurable circuits

| Structure | What it exercises | Access | Where |
|---|---|---|---|
| `gf180mcu_ocd_io__bi_a` x44 | IO cell drive strength (2-bit `drive0/drive1`), slew select, schmitt/CMOS input threshold, pull-up/pull-down strength, 5 V pad / 3.3 V core level shifting | Every GPIO is software configurable through `gpio_wb.v` / `gpio_vector_wb.v`; GPIO 16-35 have no special function | `verilog/rtl/gf180mcu_padframe.v`, `picosoc.v` |
| `ring_osc2x13` + DLL | Delay of `gf180mcu_as_sc_mcu7t3v3` inv_2 / invz_2 / clkbuff_4 stages at 3.3 V; 13 trimmable stages | "Clock1 monitor" on GPIO[14] and "Clock2 monitor" on GPIO[15] through programmable dividers (`clock_routing.v`); control via housekeeping SPI (GPIO 1-4) | `ip/ring_osc2x13/` (magic layout, extracted `ring_osc2x13.spice`, LVS `validate/comp.out`), `verilog/rtl/ring_osc2x13.v`, `dll*.v` |
| `gf180mcu_ocd_ip_sram__sram512x8m8wm1` x4, `__sram256x8m8wm1` x4 | 3.3 V SRAM function and access time inside a synthesised system (15 ns clock constraint) | Only through the CPU (firmware memory test, UART on GPIO 5/6, flash on GPIO 36-42); no direct pin access | `verilog/rtl/mem_wb.v`, `librelane/config.yaml` |
| PicoRV32 SoC in `gf180mcu_as_sc_mcu7t3v3` | Functional / Fmax data point for the 3.3 V standard cells; clock from pad GPIO[38] or the DLL | External clock sweep; trap monitor GPIO[13] | `verilog/rtl/`, `verilog/gl/` |
| `simple_por` | `nfet_06v0`/`pfet_06v0` cascoded step-down mirrors (8x, 7x, 8x), `ppolyf_u_1k_6p0` 2000 um and 200 um x 1 um bias resistors, 9x 10x10 um `cap_mim_2f0_m4m5_noshield`, Schmitt trigger; POR delay versus supply | Indirect only: `porb` is level-shifted to 3.3 V and resets the SoC; not on a pad in this chip | `ip/simple_por/netlist/simple_por.spice` and `*_tb*.spice` |
| `lvlshift_down` | 5 V to 3.3 V down-shift using two `gf180mcu_fd_sc_mcu7t5v0` inverters on different rails | Indirect (POR path) | `verilog/rtl/lvlshift_down.v`, `magic/lvlshift_down.mag` |
| `user_id_programming` | Via-programmed 32-bit ROM | Read as `mask_rev` by the SoC | `magic/user_id_programming.mag`, `scripts/set_user_id.py` |

Voltage domains: pads 5.0 V (`vddio`/`vssio`, DVDD/DVSS inside the pad cells), core 3.3 V (`vccd`/`vssd`).

Simulation data to compare against: the ngspice POR testbenches (decks only, no stored waveforms; the netlist comment says "Calibrated to about 40-50ms rise time", the verilog model comment says 15 ms), `ip/ring_osc2x13/lib/ring_osc2x13.lib`, and the SRAM `.lib` corner files in the PDK install (not in the repo). No stored STA reports.

### Candidate PDK additions

- The three libraries themselves are already distributed through open_pdks; this chip is a silicon validation vehicle for them: `gf180mcu_ocd_io` (dual-voltage 5 V pad / 3.3 V core IO), `gf180mcu_ocd_ip_sram` (256x8 and 512x8 here), `gf180mcu_as_sc_mcu7t3v3`.
- `simple_por` (5 V POR macro: GDS, LEF, magic, netlist) and `ring_osc2x13` (hand-placed trimmable ring-oscillator hard macro in the 3.3 V cells) are reusable IP.
- `gf180mcu_ocd_io__fill10x/y/z`: padframe filler cells carrying constant-one/zero loopback and a level shifter; a candidate addition to the IO library.
- The openframe harness itself is a template (`doc/README` gives a 12-step procedure).
- PDK/tool problems the author recorded:
  - `ip/simple_por/mag/README` lists 12 items found while drawing the POR: ppolyf resistors not getting L/W in ngspice (netlist comment says later corrected in the PDK), MiM `M=` multiplier ignored, magic device-generator metal-stack parsing (`THICKMET0P9`), mvpmos guard ring drawn with nsubdiff instead of mvnsubdiff, rule DF.5 "MV P-field overlap", rule DV.9 (MV FET with LV guard ring not flagged), `ppolyf_u_1k`/`ppolyf_u_1k_6p0` contact-distance rules and device generation ("Needs a lot of work"), MiM bottom-plate spacing/surround rules. Most are marked "(Fixed)"; items 8 and 12 are not.
  - `simple_por.v`: the GF IO library verilog models power with `supply0`/`supply1`, so a power ramp cannot be simulated.
  - `doc/README`: LVS merged VSS and DVSS in the IO corner cell (commit dee3bb5 says the IO library was updated to resolve this "false positive ground node shorting"); netgen could not parse the `generate` block of the ID ROM (worked around in 92ec3c5 by generating a gate-level module).
  - Commit 349e17e: SRAM core cell NPLUS/PPLUS patterns violated DRC rule PP.8 and were corrected in the SRAM library.

### What would be needed to use it

Packaged or chip-on-board test on a Caravel-style board: 5 V and 3.3 V supplies, external clock on GPIO[38], SPI flash on GPIO 36-42 (or housekeeping SPI on GPIO 1-4), UART on GPIO 5/6. For IO characterisation: firmware that steps drive/slew/pull settings, plus a scope and a source-meter. For the ring oscillator: a frequency counter on GPIO 14/15 and a trim sweep. Missing: any test plan, firmware for memory test or oscillator sweep, pinout/bond-out document, expected-value tables.

## Notes and caveats

- The platform lists the repository without a branch; the default branch does not contain the submitted design. Anyone cloning main gets the empty template.
- The platform sha256 (`adf56210...`) does not match any GDS in the repo. The uncompressed `gds/caravel_openframe_filled.gds.gz` on the branch hashes to `fe2a78f1...`, which matches `doc/hashes.txt`; `gds/hashes.txt` on the same branch holds a different, older value (`7fbdb6fa...`). I could not confirm which commit was manufactured. The last GDS update (2026-01-02, PP.8 fix) changes mask data.
- `doc/README` still says LVS fails and that "most macros (namely SRAM and standard cells, but not I/O) are being ignored by LVS"; later commits (e99c5a0, 92ec3c5) say the chip "now passes LVS" and `validate/caravel_openframe_comp.out` agrees. The README text is stale.
- `doc/README` states that wrapper power connections and some edge routes were fixed by hand after LibreLane, so the committed LibreLane config does not reproduce the taped-out layout on its own.
- `CTS_CLK_BUFFERS` in `librelane/config.yaml` names `gf180mcu_as_sc_mcu9t3v3__clkbuff_*` (9-track) while the selected library is the 7-track one; not checked which buffers were actually used.
- Upstream repositories of the three libraries are not referenced; library versions are whatever was installed in `/usr/share/pdk/gf180mcuD` on the author's machine.
