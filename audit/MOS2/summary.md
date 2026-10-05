# MOS2: AutoMOS-Chipathon-2025 full size (ws-run1)

| | |
|---|---|
| Repository | https://github.com/AutoMOS-project/AutoMOS-chipathon2025 @ 85e9753 (`update-for-ws`, last commit 2025-12-18) |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | MIT (`LICENSE`; audit: PRESENT, GitHub detects MIT). No SPDX headers in sources. |
| README | OK (535 words), but it describes only the AutoMOS team's own block and the Docker workflow, not the chip that was taped out |

## What the repo actually contains

The README and `docs/proposal.md` describe one team's project ("CMOS 180nm VCO and LDO
Design with Open-Source EDA and GLayout": bandgap + bias generator + 1.8 V LDO + ring VCO).
The repository is in fact the **integration repository for the whole SSCS Chipathon 2025
Analog/AI track ("A track") shared die**, holding six teams' analog blocks and the common
pad ring. The wafer.space submission is that complete Chipathon die dropped unchanged
into the middle of a wafer.space 1x1 template frame (platform description: "Embedded
Chipathon design").

Structure of the taped-out layout, read from the cell hierarchy of
`designs/libs/A_track_core/chiptop_A_track_ws_embedded/chiptop_A_track_ws_embedded__fixed_filled.gds.tar.gz`
(179 MB GDS, top cell `chip_top`, 3932 x 5122 um):

- Outer ring: the stock wafer.space template pad frame (`gf180mcu_fd_io__bi_24t` x40,
  `in_c` x13, `in_s` x1, `asig_5p0` x2, `gf180mcu_ws_io__dvdd` x8, `gf180mcu_ws_io__dvss` x10,
  corners, fillers, `gf180mcu_ws_ip__id`, `gf180mcu_ws_ip__logo`, `sealring`). The template's two
  `gf180mcu_fd_ip_sram__sram512x8m8wm1` macros are present in
  `designs/libs/A_track_core/padring_ws/chip_top.gds` but absent from the embedded GDS.
- Cell `chipathon` placed at (500, 500) um: the Chipathon die, about 2935 um square
  (`padring.def`: corners at 0 and 2580 um), containing its own complete pad ring
  `padring` (`gf180mcu_fd_io__asig_5p0` x55, `bi_t` x16, `dvdd` x9, `dvss` x8, `brk5` x9, `cor` x4,
  fillers) and six team blocks.
- `chip_top` itself holds only the template's pad pin squares (25 x 25 um, Metal5) and their
  labels, and `chipathon` holds only marker/label datatypes. **No routing cell or top-level
  metal connects the outer wafer.space ring to the inner Chipathon ring** (inferred from the
  hierarchy and top-level shapes; not checked by extraction). The outer pads therefore appear
  to be dummies and the circuits are reachable only through the inner ring's pads.
- Dummy fill cells (`COMP_FILL`, `POLY2_FILL`, `METAL1..5_FILL`) were added at top level.

Sub-designs on the die (one line each):

| Block | Team name (where stated) | Function | Devices | Sources in repo |
|---|---|---|---|---|
| `A1_top` | AutoMOS | Bandgap (`bandgap`, 5x `pnp_05p00x00p42`), bias generator (`biasgen_v2`), LDO (`ldo`: 10 x 50 um / 0.5 um `pfet_05v0` pass device, `ldo_OTA`), 3-stage current-starved ring VCO with buffer and enable gate (`vco`, `nfet_03v3`/`pfet_03v3`, L = 0.28 um), three 2:1 analog muxes for external override of reference and bias, 8:1 analog test mux (`test_v2`) | 118 `nfet_05v0`, 108 `pfet_05v0`, 13 `nfet_03v3`, 14 `pfet_03v3`, 249 `ppolyf_u_1k_6p0`, 5 `cap_mim_2f0fF`, 5 PNP | Full xschem schematics, symbols, GDS, extracted netlists, netgen LVS reports, testbenches (`designs/libs/core_*`, `tb_*`, `A1_core`, `A1_tb`) |
| `A2_top` | "GenYZ" (GDS release file names) | 4-channel chopper instrumentation amplifier with low-pass filters and a clock divider (pins `IN1..4P/N`, `OUT1..4P/N`, `CLK_IN`, `RST_IN`, `VBIAS`, `VCM`) | not readable on this branch | Flat GDS only on this branch (`A2_core/A2_top/A2_top.gds`, one cell). Schematics (`A2_TOP.sch`, `switch_A/B/C`, `INA_STAGE_FIXED`, `lpf`, `clockdiv.v`) exist only on the default branch `integration` |
| `A3_top` | "gLayout-mahowalders" (`A3_core/README.md`) | Three NMOS current-mirror variants: plain (`vanilla_cm`), biased (`biased_cm`), cascode (`cascode_cm`), with a PMOS bias/distribution stage | 20 `nfet_03v3`, 26 `pfet_03v3` | Schematics, GDS, LVS report |
| `A4_top` | "SiliconSouls" (GDS release file names) | Quadrature oscillator (`osc_core`, `v2i_stage`, 2x `op_buffer`; pads `VOUT0/90/180/270`, `VCTRL`, `VREF`, `IBIAS`, `EN`) | not readable | GDS only (`A4_core/A4_top/A4_top.gds`, `delay_stage.gds`); README is "Schematics Placeholder" |
| `A5_top` | not stated | Gilbert-cell mixer (`Gilbert_mixer_interdigited`, degeneration resistor, `4t-ota` buffer, biasing network with local NMOS/PMOS mirrors, output stage) | `nfet_03v3`, `pfet_03v3`, `ppolyf_u`, `ppolyf_u_1k`, `cap_mim_2f0fF` | Schematics and GDS; no LVS report |
| `Axx_top` | not stated | Current reference generator: error amplifier plus mirrors producing 2 uA, 20 uA and 50 uA sourcing and sinking outputs, reference voltages | 35 `nfet_03v3`, 25 `pfet_03v3` | Schematics, GDS, LVS report (failing, see caveats) |

All blocks are full-custom analog. gLayout (submodule `designs/gLayout`, the team's fork
`AutoMOS-project/gLayout-AutoMOS-cp25`) was used to generate layout primitives; generated cell
names such as `L_route_*`, `c_route_*`, `straight_route_*`, `via_array_*` are visible in `A4_top`.
Schematic capture is xschem, layout KLayout, LVS magic + netgen (`run_lvs.sh` in several cell
directories), DRC the KLayout deck (a copy sits in `A1_core/A1_top/main.drc`).

Other directories:

- `designs/Chipathon2025_pads/`: xschem symbols and magic views of the IO cells and the
  secondary ESD cell.
- `designs/libs/A_track_core/chiptop_A_track/releases/`: 30 intermediate top-level GDS versions
  (v2 to v12), about 2.5 GB.
- `designs/libs/A_track_core/fills/`: unit fill cells.
- `designs/libs/A_track_core/Bracolin_submmitted/Bracolin_submitted.gds`: an unrelated
  reference layout (top cell `IDDMIMS3`), not instantiated in the die.
- `designs/gf180mcu` (wafer-space/gf180mcu @ fb4b8f5) and
  `designs/globalfoundries-pdk-libs-gf180mcu_fd_pv` (LuighiV fork @ 66f2dfe): PDK and DRC deck
  submodules.
- `docs/`: template workflow documents plus the AutoMOS proposal (PDF and markdown).
- `releases/chiptop_A_track_v12_exported.gds` (GitHub release v1.0, 2025-10-14): the snapshot
  sent to the foundry for the Chipathon's own run, "DRC clean version".

Verification evidence present: netgen reports `A1_top_comp.out`, `A1_top_core_comp.out`,
`A3_top_comp.out`, `layout_ldo/ldo_comp.out` (all "Circuits match uniquely"), and
`Axx_top_comp.out` (fails). One KLayout DRC database for an old version
(`chiptop_A_track_v6_0001dbu.rdb`). No DRC report for the final embedded layout and no
silicon data.

Branches: the default branch is `integration`, not the listed `update-for-ws`. `integration`
contains the same eleven wafer.space commits (rebased, different hashes) and is 55 commits
ahead: it adds the A2 schematics, pad-symbol updates, and packaging documents
(`docs/Analog_AI_wirebonding_diagram.pdf`/`.svg`, `docs/padring_export.pdf`,
`docs/LQFP10X10-64-OP-01-B-R3-ECN-10717.pdf`, last commit 2026-08-24 "Update wirebonding").
The layout files are the same on both; the difference is material only for documentation.
`main` is 92 commits behind and holds just the early template.

## Authors and affiliations

| Person / organisation | Role | Affiliation | Source |
|---|---|---|---|
| Luighi Viton-Zorrilla (`LuighiV`) | AutoMOS lead; bandgap, bias generator, top integration, LVS/DRC clean-up, wafer.space embedding (226 commits) | not stated (GitHub bio: "Electronic engineer, enthusiastic programmer") | README team list, git log, `gh api users/LuighiV` |
| Julio David Vilca Pizarro (`jvilcap`) | AutoMOS: LDO schematic and layout, decoder, analog mux (60 commits) | not stated (commit email domain is a Peruvian university, hint only) | README, git log, `docs/schedule/Chipathon2025_github_issues.tsv` |
| Rodrigo Marin (`madorat`) | AutoMOS: VCO and test mux (40 commits) | not stated | README, git log, schedule TSV |
| Aleta Edna, `keisyandretta` | A2 block (chopper INA) | not stated | git log paths (`A2_core`) |
| Saptarshi Ghosh, Subham Pal | A3 block (current mirrors) | not stated | git log paths (`A3_core`) |
| Vasil Yordanov | A5 block (Gilbert mixer), pad views | not stated | git log paths |
| Alex Mantilla (`AlexMantilla1`) | Axx block (current reference), top-level integration | not stated (GitHub location Bucaramanga, Colombia) | git log paths, `gh api` |
| A4 team "SiliconSouls" | Quadrature oscillator | not stated; no commits attributable | GDS file names only |
| `JuanMoya` | Pad ring symbols, top integration; forwarded the GDS to the foundry for the Chipathon run | not stated | git log, release v1.0 notes |
| Jianxun Zhu, Peter Kinget | Authors of the upstream Chipathon analog project template (OTA example, KLayout library manager, docs) | not stated in this repo | git log paths (`core_analog`, `scripts`), README reference to the IIC-OSIC-TOOLS template (role inferred) |
| SSCS Chipathon 2025 | Programme under which the die was designed | IEEE SSCS | README |

## Value for GF180MCU characterisation and PDK development

### Verdict: High

The die is a collection of device-level analog circuits with direct analog pad access
(55 `asig_5p0` pads): a bandgap, PTAT bias generator, LDO, two oscillators, a precision current
reference, three current-mirror topologies, a mixer and a chopper amplifier, across both
3.3 V and 5 V/6 V device families, with schematics and testbenches for most of them. Its value
is limited by the pad-access problem described below and by missing sources for two blocks.

### Test structures and measurable circuits

Pad names are those of the inner Chipathon ring (`designs/libs/A_track_core/padring/padring.v`,
`padring.sch`). "ASIG" pads are `gf180mcu_fd_io__asig_5p0`; digital controls use
`gf180mcu_fd_io__bi_t` with tie-high/tie-low cells setting the pad mode. Supplies are per team
on `dvdd`/`dvss` pads: `VDD_A1`/`VSS_A1`, `VDD_A2`/`VSS_A2`, `A_VDD_A2`/`A_VSS_A2`, `VDD3V3_A3`/
`VSS_1_A3`/`VSS_2_A3`, `VDD_A4`/`VSS_A4`, `VDD3V3_A5`/`VSS_A5`, `VDD3V3_AXX`/`VSS_AXX`, plus
`LDO_OUT` and `VTEST` (both on `dvdd` cells).

| Structure | What it exercises | Access (inner-ring pads) | Repo paths | Simulation to compare against |
|---|---|---|---|---|
| A3 plain, biased and cascode NMOS current mirrors | `nfet_03v3` / `pfet_03v3` matching, output resistance, mirror gain error versus topology; 3.3 V domain | `V_IN_A3`, `V_AUX_A3`, `V_BIAS_IN_A3`, outputs `VCM_OUT_A3`, `BCM_OUT_A3`, `CCM_OUT_A3`, enable `EN_A3` | `designs/libs/A3_core/{vanilla_cm,biased_cm,cascode_cm,bias,top_level,A3_top}` | Schematics only; no testbench or results in this repo |
| Axx current reference | `nfet_03v3`/`pfet_03v3` mirror ratios and error-amplifier offset; absolute currents 2/20/50 uA, sourcing and sinking | `IOUTP_2UA`, `IOUTN_2UA`, `IOUTP_20UA`, `IOUTN_20UA`, `IOUTP_50UA`, `IOUTN_50UA`, `VREF12`, `VCOMP`, `VREFP_AXX`, `VREFN_AXX` | `designs/libs/Axx_core/*` | Schematics only |
| A1 bandgap + bias generator | `pnp_05p00x00p42` Vbe and PTAT slope, `ppolyf_u_1k_6p0` ratio and absolute value, `pfet_05v0` mirrors | Indirect only: the 2 uA PTAT copy is routed to input 5 of the test mux and appears on `TEST_OUT` when `TEST_EN`=1 and `TEST_SEL_2..0`=101; the 10 uA source/sink copies feed the LDO and VCO bias. The bandgap outputs `VBG`, `VZTC`, `ICTAT`, `IZTC` are marked `noconn` in `A1_top_core.spice` and reach no pad | `designs/libs/core_bandgap`, `core_biasgen`, `tb_bandgap`, `tb_biasgen` | Testbench schematics (`tb_bandgap_dc_tran`, `_stb`, opamp AC/CMRR); no stored results |
| A1 LDO (1.8 V target) | `pfet_05v0` pass-device Ron and dropout, loop stability, `ppolyf_u_1k_6p0` feedback divider, `cap_mim_2f0fF` compensation | Output on the `LDO_OUT` pad. Reference and bias via `LDO_VREF` and `LDO_IBIAS` with `LDO_VREF_OVRD` / `LDO_IBIAS_OVRD`; enable `LDO_EN`. In the netlist the internal input of the reference mux is tied to VSS, so the LDO needs an external reference on `LDO_VREF` | `designs/libs/core_ldo`, `layout_ldo`, `tb_ldo` | Plot images of op, DC, AC, PSRR, noise, stability and transient runs (`tb_ldo/*/tb_ldo_*.png`); no raw data |
| A1 ring VCO | `nfet_03v3`/`pfet_03v3` inverter delay at L = 0.28 um versus bias current and control voltage (current-starved, 3 stages), supplied from the LDO output | `VCO_IN` (control), `VCO_IBIAS` + `VCO_IBIAS_OVRD`, `VCO_EN`, output `VCO_OUT` through a `bi_t` digital pad | `designs/libs/core_vco`, `tb_vco`, `core_vco/vco/Top_VCO.ipynb` | Testbench schematics and a notebook; the proposal quotes 2.4 GHz, which a `bi_t` pad is unlikely to pass |
| A1 8:1 analog test mux and 2:1 muxes | `pfet_05v0`/`nfet_05v0` transmission-gate on-resistance and leakage | `TEST_OUT`, `TEST_SEL_0..2`, `TEST_EN`, `TEST_IBIAS`, supply `VTEST`; seven of the eight inputs are tied to VSS | `designs/libs/core_test`, `layout_test_all_tg`, `layout_test_decoder`, `A1_core/top_amux_2x1` | `tb_test_tg`, `tb_test_tristate` schematics |
| A4 quadrature oscillator | Oscillator frequency versus control voltage and bias; device types not readable | `VCTRL`, `VREF`, `IBIAS`, `EN`, `VOUT0`, `VOUT90`, `VOUT180`, `VOUT270` | `designs/libs/A4_core/A4_top/*.gds` | None (no schematic) |
| A5 Gilbert mixer | `nfet_03v3` switching quad and transconductor, `ppolyf_u` / `ppolyf_u_1k` loads, `cap_mim_2f0fF` | `VIN_V_RF_A5`, `VIN_V_RFb_A5`, `VIN_V_LO_A5`, `VIN_V_LOb_A5`, `I_BIAS_A5`, `VOUT_V_IF_A5` | `designs/libs/A5_core/*` | Schematics only |
| A2 4-channel chopper INA | Amplifier offset and 1/f noise with chopping, switch charge injection, on-chip clock divider | `IN1..4P/N_A2`, `OUT1..4P/N_A2`, `VBIAS_A2`, `VCM_A2`, `CLK_IN_A2`, `RST_N_A2` | GDS on this branch; schematics on `integration` | None on this branch |
| Secondary ESD cell `io_secondary_5p0` (used on analog inputs) | `diode_pd2nw_06v0` and `diode_nd2ps_06v0` (4 x 10 um x 10 um each) clamp leakage, 40 um x 10 um `ppolyf_u` series resistor | Any analog pad at low current | `designs/libs/A5_core/io_secondary_5p0`, `designs/Chipathon2025_pads/` | `io_secondary_DC_tb.svg` |

Voltage domains: the A1 analog core uses the 5 V-class devices with the VCO in an island fed by
the LDO; A3, A5 and Axx use 3.3 V devices.

Device-level PDK testbenches (not silicon structures) are in `designs/libs/tb_analog/`:
`tb_bjt_diode`, `tb_capacitor`, `tb_nmos_diode`, `tb_pmos_diode`, `tb_resistor` (with
`tb_resistor.png`).

### Candidate PDK additions

- **`io_secondary_5p0`**: a small secondary ESD cell (series poly resistor plus two diode
  clamps) with schematic, symbol, magic and GDS views, LVS-checked inside `A1_top` and `A3_top`.
- **`sc_tieh_tiel`**: a tie-high/tie-low pair built from `gf180mcu_fd_sc_mcu9t5v0__tieh`/`tiel`
  for strapping `bi_t` pad control pins in a pad ring without digital logic
  (`designs/libs/A1_core/top_tieh_tiel`).
- **xschem symbols for the IO cells** (`designs/Chipathon2025_pads/xschem/symbols/`) and an
  xschem/Verilog/DEF description of an analog pad ring (`A_track_core/padring/`).
- **DRC deck changes** in the LuighiV fork of `globalfoundries-pdk-libs-gf180mcu_fd_pv`: commits
  "Add COMP density rule" and "Update poly2 count to include dummies" (2025-10-02), on top of
  "Minor updates for gf180mcu DRC rules". Worth diffing against the upstream deck.
- **netgen setup differences**: every LVS directory carries a private `gf180mcuD_setup.tcl`
  that differs from the copy in the `wafer-space/gf180mcu` submodule in how `cap_mim_2f0fF` is
  handled (parallel merging disabled, unconditional `equate classes` with
  `cap_mim_2f0_m4m5_noshield`). The extraction script also needs `gds flatglob cap_mim`, `pfet*`,
  `nfet*` before reading gLayout output. Both point at friction between gLayout-generated
  layout and the stock magic/netgen flow.
- **gLayout fixes** in `AutoMOS-project/gLayout-AutoMOS-cp25`: commits "Fix to BJT model",
  "Test bjt with new parameters", "Add bandgap and biasgen", "Update current mirror block".
- **Layout hygiene issues hit before submission** (commit messages of 2025-12-11 to 12-17):
  zero-area geometries and empty hierarchies had to be deleted, an out-of-boundary geometry
  fixed, and the database unit changed to 0.001 um.

### What would be needed to use it

- Confirmation from the authors of how the die is meant to be accessed. The inner pad ring
  sits roughly 500 um inside the wafer.space ring, so wire bonds or probes must land on the
  inner pads over the unused outer ring; the inner ring has 92 pads against the 64-lead LQFP
  drawing kept on `integration`.
- The wirebonding diagram and pad-ring export from the `integration` branch, and a pad-number
  to signal table (only `padring.v` instance order exists here).
- Bench equipment: source-measure units for the mirrors, current reference and bandgap PTAT
  current; a spectrum analyser or fast scope with an active probe for the two oscillators and
  the mixer; a low-noise analyser for the chopper amplifier.
- Schematics for A4 and (on this branch) A2, and a written test plan; none exists.
- Raw simulation data: only plot images are stored (`*.raw` is git-ignored), so reference
  numbers have to be regenerated from the testbench schematics.

## Notes and caveats

- The submitted layout hash (`d4cd6202...`) matches none of the GDS files in the repo
  (`padring_ws/chip_top.gds`, the two files inside the `.tar.gz` archives) nor the release
  asset. The final GDS files are git-ignored and only two compressed archives were committed;
  `__fixed_filled` is the closest candidate.
- Platform name and description do not say that the die holds six teams' circuits; the README
  covers only A1.
- `Axx_top_comp.out` reports "Netlists do not match" and "Top level cell failed pin matching".
  No LVS evidence exists for A2, A4 or A5, and the top-level schematic
  `chiptop_A_track.sch` instantiates only A1, A3, A5 and Axx (commit message: "Waiting for A2
  and A4").
- The statement that the outer ring is unconnected is inferred from cell hierarchy and
  top-level shapes, not from an extraction.
- The `__exported` archive still has a duplicate set of pad-ring cells inside `Axx_top`; in
  the `__fixed_filled` archive `Axx_top` no longer references them.
- Submodule URLs for `gf180mcu` and the DRC deck use SSH (`git@github.com:`).
