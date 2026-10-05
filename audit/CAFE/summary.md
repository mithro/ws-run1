# CAFE: FazyRV Hachure (ws-run1)

| | |
|---|---|
| Repository | https://github.com/meiniKi/gf180mcu-fazyrv-hachure @ 51047e6 (main, last commit 2025-12-15; tags v1, v1_1; branches tapeout_v1_1/v1_2/v1_3) |
| Slot | 1x1 (die 3932 x 5122 um) |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 (top-level LICENSE); FazyRV sources carry MIT headers |
| README | OK (1,533 words) |

(Note: "CAFE" in run 2 is an unrelated project.)

## What the repo actually contains

A system-on-chip that puts **seven FazyRV cores** on one die, "for testing and research purposes" (README). FazyRV is a bit-serial RV32 core with a scalable datapath (CHUNKSIZE 1, 2, 4 or 8 bits); the Hachure instances are:

| Core | Instance | Boot addr | Macro |
|---|---|---|---|
| 1-bit, LOGIC register file | i_frv_1 | 0x10 | `macros/frv_1` |
| 2-bit LOGIC | i_frv_2 | 0x20 | `macros/frv_2` |
| 4-bit LOGIC | i_frv_4 | 0x30 | `macros/frv_4` |
| 8-bit LOGIC | i_frv_8 | 0x40 | `macros/frv_8` |
| 4-bit LOGIC with CCX custom-instruction interface (from HeiChips 2025 hackathon, README says "Not verified") | i_frv_4ccx | 0x60 | `macros/frv_4ccx` |
| 1-bit BRAM register file | i_frv_1bram | 0x70 | `macros/frv_1bram` |
| 8-bit BRAM register file | i_frv_8bram | 0x80 | `macros/frv_8bram` |

- Every core is hardened as its own macro (Yosys prep script `prep.ys` plus per-macro `config.yaml`: die 602 x 425 um for frv_1 up to 699 x 492 um for frv_8bram; macro clock targets 40 to 66 ns), then placed by hand in the top-level `librelane/config.yaml`. The macro builds (`make librelane-macro-fast`, `make copy-macro`) produce `macros/*/final/`, which is **not committed** (macros/ has only configs, so the top-level cannot be rebuilt without running the macro flow first).
- The BRAM variants use the foundry SRAM `gf180mcu_fd_ip_sram__sram512x8m8wm1` as the core register file; top-level on-chip RAM is `ram512x32` (four 512x8 macros) in `src/ram512x32.sv`, `wb_ram.sv`.
- Shared Wishbone bus with arbiter (wb_intercon, verilog-arbiter), rggen CSR block (GPIO with per-pin CMOS/Schmitt, slew and pull controls, SPI/OLED/IRQ/guard/slew/pull registers), QSPI XIP cache (`MS_QSPI_XIP_CACHE`), QSPI memory controller for external ROM/PSRAM, UART and SPI (Efabless EF_UART/EF_SPI forks), an OLED DMA/SPI engine (`tiny_wb_dma_oled_spi.sv`), a Wishbone-to-AHB3-lite bridge. Pin map in README (36 bidir, 16 input pads, 1 reserved analog pad, no analog circuits).
- **Per-block enables and clock gating** (the research hook): dedicated input pads `en_wb`, `en_p`, `en_p2`, `en_frv1`, `en_frv2`, `en_frv4`, `en_frv8`, `en_frv4ccx` (and two shared with CCX result pads for the BRAM cores) drive one `gf180mcu_fd_sc_mcu7t5v0__icgtp_4` clock gate per block (`src/hachure_soc.sv`, 10 gates; `-DNO_CLOCK_GATES` variant for simulation). So any core can be run alone or concurrently while the others' clocks are stopped.
- Flow: LibreLane Chip flow, 5 V 7-track cells (`gf180mcu_fd_sc_mcu7t5v0`, icgtp_4 instantiated explicitly), `CLOCK_PERIOD: 100` (25 MHz stated in comment is inconsistent: 100 ns is 10 MHz), IO cells `bi_24t`, `in_s`/`in_c`, ws dvdd/dvss; three logos (ws, hachure, fazyrv), `MAGIC_GDS_FLATGLOB` list for SRAM/IO cells and Metal2 filler workaround as in the template.
- Test: cocotb benches for toggle, sram, uart, spi, efspi and xip tests across all seven cores (`cocotb/`, RTL and gate-level targets, `@cocotb.parametrize(core=[...])`); firmware in `firmware/` (test_toggle writes GPO 0xff/0x00 in a loop for per-core throughput; test_sram etc.).
- No releases, no metrics or reports in the repo; layout images only (`doc/chip.png`, updated "to submitted gds" at HEAD).

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Meinhard Kissich | SoC and FazyRV author (103 commits) | Graz University of Technology, Embedded Architectures & Systems group, per the FazyRV submodule README acknowledgement (GitHub location Graz, Austria) | `ip/FazyRV/README.md`, `gh api users/meiniKi`, git shortlog |
| Marcel Baunach | co-author of the FazyRV paper | TU Graz EAS group (inferred from the acknowledgement; not stated as his affiliation) | `ip/FazyRV/README.md` citation |
| Leo Moser | template author (78 commits) | not stated in repo | AUTHORS.md |
| Tim Ansell, TrevorPeyton | 1 commit each | not stated | git shortlog |

Reference: Kissich and Baunach, "FazyRV: Closing the Gap between 32-Bit and Bit-Serial RISC-V Cores with a Scalable Implementation", ACM Computing Frontiers 2024, doi 10.1145/3649153.3649195 (cited in the FazyRV README).

## Value for GF180MCU characterisation and PDK development

### Verdict: Medium
It is a deliberate research vehicle: seven variants of the same ISA at different datapath widths and register-file styles, individually clock-gated and enabled from pads, on one die with pad-accessible GPIO. That makes it the best digital candidate in this set for measuring standard-cell area/speed/energy trade-offs in silicon. It is not High because there are no analog, device-level or oscillator structures, and no purpose-built measurement circuitry (no on-chip counters, no sense resistors).

### Test structures and measurable circuits
Designed measurements (from the README, enable pins and firmware; the energy aspect is inferred from the enable/clock-gate design):

| Structure | What it measures | Access | Where |
|---|---|---|---|
| 7 core macros, each alone | per-core dynamic power and supply current (enable one core, others gated), performance (cycles per instruction by chunk size 1/2/4/8, e.g. toggle rate of GPIO[0] under `test_toggle`), maximum clock frequency per core | `en_*` pads, `clk_PAD`, GPIO and UART pads, supply current at DVDD | `src/hachure_soc.sv`, `firmware/test_toggle` |
| LOGIC vs BRAM register file (frv_1 vs frv_1bram, frv_8 vs frv_8bram) | flop/latch-array register file versus foundry SRAM macro: area, speed, power | enable pads (en_frv1bram/en_frv8bram share the CCX result pads) | `macros/*bram` |
| 4-bit CCX core with off-chip custom-instruction interface | chunked custom-instruction handshaking through pads | ccx4_* pads | `ip/FazyRV-ccx` |
| Per-pad slew/pull/CMOS-vs-Schmitt CSRs (`GPSL`, `GPCS`, `Misc_Slew_Rate`, `Misc_Pull`) | IO cell `bi_24t` slew-rate and pull options (SL, CS, PU, PD pins) | bidir pads via CSR writes | README CSR map, `src/chip_core.sv` |
| 10 icgtp_4 clock gates | clock-gate cell function and leakage with gated blocks | enable pads | `src/hachure_soc.sv` |
| 4 x sram512x8 on-chip RAM + 2 BRAM cores' register files | foundry SRAM function | CPU access | `src/ram512x32.sv` |

Pre-silicon STA exists only as macro-level LibreLane runs (not committed); no results are in the repo to compare against.

### Candidate PDK additions
Nothing found for the PDK itself. Contribution is hierarchical macro flow for several cores plus comparison data; the repo carries the same template-level DRC/filler workarounds (`MAGIC_GDS_FLATGLOB`, `KLAYOUT_FILLER_OPTIONS: Metal2_ignore_active`).

### What would be needed to use it
Packaged or COB part, a 5 V supply with a current meter (or Kelvin sense) on VDD, a clock source for 10 MHz and beyond, a QSPI flash image per core (boot addresses in README) or XIP stub, a logic analyser/MCU on GPIO to count toggles. Firmware sources for the tests exist; a benchmark set (Embench submodule, CoreMark or similar) is not wired up in this repo. The `final/` macro outputs are absent, so STA numbers need a rebuild.

## Notes and caveats

- Run-2 "CAFE" is a different project (Expresso Ethernet chip) and is not covered here.
- `ip/FazyRV` and `ip/FazyRV-ccx` are two different commits of the same FazyRV repository (`78f30d7a`, `271205fc`; the second is the heichips25_ccx branch); embench-iot submodules use an SSH URL but cloned from mirrors.
- The README says 25 MHz in config comments, but `CLOCK_PERIOD` is 100 ns; macro periods differ (40 to 66 ns), so intended Fmax per core is not clearly stated.
- The author's affiliation comes from the FazyRV README, not from this repo's own README.
- Platform sha (fa626134...) cannot be checked: no GDS in the repo or releases.
