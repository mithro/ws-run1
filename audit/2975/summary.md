# 2975: Cloneless1 (ws-run1)

| | |
|---|---|
| Repository | https://github.com/ThorbenMoos/Cloneless1 @ fdc35dd (main, last commit 2026-07-30; releases v1 2026-01-08, v2 2026-01-24) |
| Slot | 1x1 (DIE_AREA 0 0 3932 5122 um) |
| Top cell | chip_top |
| Licence in repo | GPL-3.0 (LICENSE file; no SPDX headers) |
| README | OK (1474 words, own text) |

## What the repo actually contains

A side-channel-hardened cryptographic ASIC, written entirely in VHDL (`src/`, 40 files), converted to Verilog with `ghdl synth` and hardened with LibreLane (`chip_top.yaml`, `Makefile`). Standard-cell library is gf180mcu_fd_sc_mcu7t5v0 (7-track, 5 V; hold corners are all 4v50/5v00/5v50 corners, clock 4 MHz). IO is the wafer.space padring (gf180mcu_fd_io__bi_24t). No SRAM, no analog macros, no custom devices.

Functional blocks (`src/Cloneless.vhd`, memory-mapped through a 4-bit data / 3-bit address interface):
- mid-pSquare block cipher (prime-field Mersenne arithmetic, inner-product masking, 2 shares, 2 redundancy domains), `mid_pSquare_dSHARES_IPM_kRED.vhd` and the *ModMersenne files. One hard-coded (shared) key plus one key selected by config registers that mixes in error-corrected RO-PUF bits (`mps_ipm_controller.vhd`: 2x ro_puf, 16 bits, 4 parallel, RO length 47, 16383 cycles, 7-fold majority).
- 4x ES-TRNG (edge-sampling TRNG; `es_trng.vhd`, `trng.vhd`, `tappeddelaychain.vhd`, `bitextractor.vhd`) read raw through the interface (RO length pairs 23/31, 23/47, 31/47, 31/59).
- 4x raw-access RO-PUF (`ro_puf.vhd`, 8 bits each, RO lengths 23, 31, 47, 59, count windows 1023, 4095, 16383, 65535 cycles).
- Trivium stream cipher for concurrent pseudo-randomness.

Hardened sub-macros (`macros/`, each its own LibreLane "Classic" flow with DESIGN_NAME = module name, FP_CORE_UTIL 45 for the ROs): `ringoscillator_23/31/47/59` and `carry4`. Instance counts in chip_top.yaml MACROS: ringoscillator_23 x20, _31 x19, _47 x84, _59 x17 (140 ring oscillators in total), carry4 x6, plus wafer.space id and logo macros.

Verification: GHDL and iverilog testbenches (`src/Testbenches/`), post-layout gate-level sim step in the Makefile, wafer.space precheck (Makefile reports clean). No final GDS in the repo (a download link to thorbenmoos.de is given, 59 MB zip, not fetched; releases have no assets). Silicon results are NOT in this repo; the README says the chips were manufactured, delivered and tested (cipher evaluated against 100 million power traces), and points to the separate Cloneless_artifacts repo (PCBs, FPGA controller, software) that was not in the clone set.

## Authors and affiliations

| Who | Role | Affiliation | Source |
|---|---|---|---|
| Thorben Moos (git "Thorben", GitHub ThorbenMoos) | sole author, 32 commits | GitHub bio: "Cryptography and Security Researcher", blog thorbenmoos.de; commit email is an @rub.de address (hint only, no institution stated in repo) | git shortlog, `gh api users/ThorbenMoos` |
| mid-pSquare / prime masking / ES-TRNG authors | cited prior art, not repo contributors | various (UCLouvain "uclcrypto" repos referenced) | README references |

No funding line in Cloneless1 README; the Cloneless2 README acknowledges the PBS Foundation (Project SOVereign).

## Value for GF180MCU characterisation and PDK development

### Verdict: Medium

Not built as a process-characterisation chip, but it contains 140 digitally-readable ring oscillators of 4 odd lengths and four PUFs whose raw outputs are explicitly intended for "characterization of oscillator behavior in the technology" (README, Cloneless2). Measurable via digital pins only; no analog access.

### Test structures and measurable circuits

| Structure | What it exercises | Access | Where |
|---|---|---|---|
| RO-PUF cell = two ring oscillators of identical length, each with a 16-bit ripple counter (`ro_cnt.vhd`), compared after N clk cycles; bit = (count1 >= count2) | Intra-die frequency mismatch of identical inv_1 / nand2_1 rings (7T 5 V cells) = device Vth/L mismatch, local variation; bit flip probability versus window length gives mismatch relative to jitter | Config register selects "ROPUF" core; raw bits (8 bits x 4 PUFs) read over the 4-bit data bus. Only the comparison bit is exposed, not the counter values (inferred from `ro_puf_cell.vhd`) | `src/ro_puf_cell.vhd`, `ro_cnt.vhd`, `ringoscillator.vhd`; macros/ringoscillator_{23,31,47,59} |
| Ring oscillators of 23/31/47/59 stages (inv_1 chain + nand2_1 enable) | Absolute gate delay at 5 V per die; length-ratio gives wire vs gate split | Only through counters/PUF windows, no frequency output pad | same |
| ES-TRNG ring pairs + tapped delay chain built from mux2_1/xor2_1 "carry4" emulation | Sub-gate-delay resolution timing jitter (inferred); raw TRNG bits | raw TRNG core readable | `src/carry4.vhd`, `tappeddelaychain.vhd`, macros/carry4 |

How the PUF is built: it is a ring-oscillator PUF (Suh/Devadas style, ref [10] in README), not arbiter, SRAM, or custom analog. Every cell is a stock GF180MCU 5 V standard cell: the Makefile `cell_replacement` target seds the GHDL-generated netlist so that `inv`, `nand2`, `mux2`, `xor2` become `gf180mcu_fd_sc_mcu7t5v0__inv_1 / nand2_1 / mux2_1 / xor2_1` with `(* keep *)`, so synthesis cannot restructure the loops.

Placement: placement is semi-manual. Each oscillator length is hardened once as a macro and the chip_top.yaml MACROS section pins every instance at explicit coordinates on a regular grid (`510+75*n`, pitch matching the 75 um PDN strap pitch, so each instance sees the same power-grid environment). All instances of a given length are therefore layout-identical copies, so any frequency difference between them is due to process variation (plus supply/IR-drop and neighbour effects), which is what a PUF or mismatch study wants. Inside the macros, placement and routing is automatic (OpenROAD), not hand-drawn. No simulation results (SPICE or Monte Carlo) comparing to silicon are in the repo; per-corner liberty/SPEF are generated by the flow only.

Side-channel/tamper countermeasures that depend on layout: the design depends on the cipher's masking being glitch-robust (RTL/gate-level property, no layout shielding found; grep of the flow config shows no shield, dummy-wire, or symmetric-routing options). The tamper-evidence claim is the delay-based PUF: invasive modification changes RO delays, so the derived key changes (README states this as the hypothesis to be tested, "tamper-evident delay-based PUFs"). That depends on physical layout/parasitics but there is no explicit layout countermeasure.

### Candidate PDK additions

Nothing found as a PDK addition. A usable pattern: a `cell_replacement` flow that forces named stock cells and keeps hierarchy, with macro-per-oscillator-length hardening. Hold-time corner list in chip_top.yaml names nine 4v50/5v00/5v50 corners (these are the template corners).

### What would be needed to use it

Packaged chip on the authors' PCB with FPGA controller (in the Cloneless_artifacts repo, not cloned). Pin interface is 4-bit data, 3-bit address, read/write strobes at 4 MHz, so measurement is slow but simple. Data from ~many dies is needed for mismatch statistics; the authors say they are collecting this. Absolute RO frequency needs knowledge of the counter windows (documented in Cloneless.vhd generics).

## Notes and caveats

- The die GDS is not in the repo, only a link (not downloaded). Layout checks were not run here.
- PUF bits only expose a comparison, so extracting a mismatch distribution means inferring it from bit-error rates vs. window length and across dies.
- The README states the TRNGs "need more tuning" and PUFs "look decent for a first attempt", but no data is in this repo.
