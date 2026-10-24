# TRID: gf180mcu-project-trident-gf180-teststructure (ws-run1)

| | |
|---|---|
| Repository | https://github.com/Scafir/gf180mcu-project-trident-gf180-teststructure @ f81200d (main, last commit 2026-07-02). Layout comparison used repository commit f81200d; `designs.csv` taped-out commit is f81200d5f0b3 (same commit) |
| Slot | 0p5x1 (die 1936 um x 5122 um per datasheet) |
| Top cell | TOP |
| Taped-out layout | TRID_TOP_14_2, 1936 x 5122 um, 1 placement on the G801 reticle |
| Licence in repo | REUSE layout, no root LICENSE file. `LICENSES/` holds CERN-OHL-W-2.0, Apache-2.0, CC-BY-4.0, LPPL-1.3c and LicenseRef-CERNLogo; per-file mapping below. GitHub detects no licence. |
| README | Thin (91 words) but original; the real documentation is the 21-page datasheet `doc/TRIDENT_GF180_teststructures.pdf` |

## What the repo actually contains

A full-custom, analog-only test chip ("TRIDENT-GF180-TESTSTRUCTURES", datasheet dated 30 June 2026, header "ACCURATE 2A / CERN HSE-RP-IL") whose purpose is to find out whether pA and sub-pA current measurement is feasible in gf180mcuD. Every structure is a leakage experiment: single transistors, MIM capacitors, ESD diodes, complete ESD pads (foundry and custom guarded ones), a guarded metal line, and a pA charge-pump current source. There is no digital logic, no standard cells and no LibreLane flow on `main`; the layout was drawn by hand in KLayout and the schematic in xschem.

The manufactured layout is in the repo. `klayout/top.layout.filled_db0p001.gds.gz` decompresses to SHA-256 `7d319382160df1c102aea162015a8cb007dc6d42c7418f0e104e01dd29100673`, which is exactly the `sha256` in `project.json`. The same file is the only asset of the GitHub release `ws-run-1` (byte-identical .gz).

Directory contents:

- `klayout/top.layout.gds.gz`: unfilled top layout, database unit 0.005 um, 898 cells, still carries the KLayout PCell context (`$$$CONTEXT_INFO$$$`), so device geometries can be read back from it.
- `klayout/top.layout.filled_db0p001.gds.gz`: filled, database unit 0.001 um, PCells flattened, adds `COMP_FILL`, `POLY2_FILL`, `METAL1..5_FILL` cells. This is the taped-out file.
- `klayout/gf180mcu_fd_io_pad_70x100.gds` (10.9 MB): the foundry `gf180mcu_fd_io` library re-fitted with a 70x100 um bondpad and edited so LVS recognises the pad diodes (commit 235f7c3 "add LVWELL so that diodes are recognized, ISOSUB").
- `klayout/gf180mcu_bondpad_70x100um.gds`: the custom bondpad cell.
- `klayout/top.schematic_lvs.spice`: an xschem netlist export. It is stale: it still uses the old `*_W030L070_*` net names and `TOFILL` placeholders and does not contain the MIM capacitors, discrete diodes, ESD-test pads, inverters or guard-leakage structure that are in `xschem/top.sch` and in the layout. Device lines are also "IS MISSING" because it was exported without the PDK symbols.
- `xschem/top.sch` (125 kB): the current top schematic, with all 100 pads, the pad numbers as text, and all test devices. `xschem/*.sch/.sym`: the four transistor cells, `hv_inverter_W075L070`, `matched_interface_trap_charge_pumps`, pad-ring helper cells (`corner`, `fill_25`), symbols for the 70x100 foundry pads and `gf180mcu_fd_io_70x100.spice`. `interface_trap_charge_pump_cascode.sch` is a standalone ngspice testbench that is not instantiated in the chip.
- `doc/`: LaTeX source (`main.tex`, `cernatsnote.cls`), figures and the built PDF datasheet with pin table, per-structure description and two bonding diagrams with wire coordinates.
- `ip/gf180mcu_ws_ip__id`, `ip/gf180mcu_ws_ip__logo`: wafer.space ID and logo cells from the project template; both are instantiated in `TOP`.

Not in the repo: the schematics and symbols of the three custom pads (`gf180mcu_liplib__no_esd_guarded_pad_70x100`, `gf180mcu_liplib__asig_guard_5p0_pad_70x100`, `gf180mcu_liplib__asig_guard_sense_5p0_pad_70x100_p105`). `top.sch` references them from the author's separate library https://github.com/Scafir/liplib ("Low Leakage I/O Pad Library", CERN-OHL-W-2.0 per the GitHub API). Their layouts are present inside the top GDS.

Verification: there are no DRC/LVS report files. The git log claims KLayout DRC and LVS clean and Magic DRC clean on the unfilled top (commits a8cb454 "DRC AND LVS PASS!!", 5af4d99, 048de70 "LVS clean: I swapped 2 nets in the sch"). No simulation results, no measurement data and no PCB design files are in the repo.

Pad ring as built (instance counts from the GDS `TOP` cell, identical in both layouts): 53 `gf180mcu_fd_io__asig_5p0_70x100`, 23 `gf180mcu_liplib__no_esd_guarded_pad_70x100`, 2 `gf180mcu_liplib__asig_guard_5p0_pad_70x100`, 2 `gf180mcu_liplib__asig_guard_sense_5p0_pad_70x100_p105`, 6 `gf180mcu_fd_io__dvdd_70x100`, 6 `gf180mcu_fd_io__dvss_70x100`, 4 `gf180mcu_fd_io__cor`, plus fill cells. 100 pad positions at 105 um pitch; 6 positions carry no bondpad ("NO BOND") because the custom pads are wider than one pitch.

### Licence mapping (precise)

| Licence | Copyright holder as stated | Files |
|---|---|---|
| CERN-OHL-W-2.0 | 2026 CERN | all `xschem/*.sch` and `*.sym` (in-file SPDX header), `klayout/top.layout.gds.gz`, `klayout/top.layout.filled_db0p001.gds.gz`, `klayout/gf180mcu_bondpad_70x100um.gds` (sidecar `.license` files), `klayout/top.schematic_lvs.spice` |
| Apache-2.0 | 2022 GlobalFoundries PDK Authors | `klayout/gf180mcu_fd_io_pad_70x100.gds`, `xschem/gf180mcu_fd_io_70x100.spice` |
| Apache-2.0 | 2026 Leo Moser | `ip/gf180mcu_ws_ip__id/**`, `ip/gf180mcu_ws_ip__logo/**` (via `REUSE.toml`) |
| CC-BY-4.0 | 2026 CERN | `README.md`, `doc/main.tex`, `doc/TRIDENT_GF180_teststructures.pdf`, all `doc/figures/*.png` |
| LPPL-1.3c | 2018 CERN Accelerator and Technology Sector | `doc/cernatsnote.cls` |
| LicenseRef-CERNLogo | 2026 CERN | `doc/figures/cernlogo.eps`. Not an open licence: the text says use of the logo by third parties needs prior approval. |

History: the repo started from the wafer.space template (Apache-2.0 root LICENSE), switched the root LICENSE to CERN-OHL-W-2.0 on 2026-01-05 (4e2cec2), then moved to REUSE on 2026-06-30 (22e091e) and re-attributed copyright to CERN on 2026-07-01 (804beca). The schematics carry a CERN-OHL-W section 4 notice asking that the source location be kept visible on products. The taped-out GDS itself is CERN-OHL-W-2.0 (weakly reciprocal), although it embeds Apache-2.0 foundry IO cells and the liplib pads.

## Taped-out layout

Source: `layout/TRID.oas` (reticle cell `TRID_TOP_14_2`, one placement at x 28.004 mm, y 5.242 mm on G801). Repository commit used for the comparison: `f81200d`; `wafer.space-bench/data/designs.csv` lists the taped-out commit as `f81200d5f0b3`, so they are the same commit. Every number below was measured from the OAS with KLayout; connectivity comes from a flat `LayoutToNetlist` extraction over poly, diffusion, contacts, Metal1-5 and Via1-4 (MIM top plate treated as its own layer, wafer.space fill, seal ring and text art removed). Wells and substrate are not part of that extraction, so body connections through the substrate are not traced.

| Fact | Value |
|---|---|
| Die | 1936 x 5122 um, slot 0p5x1, 1 placement on the reticle |
| Bond pads | 94 pad openings (80 of 100 x 70 um on the left and right edges, 14 of 70 x 100 um on top and bottom) on a 105 um pitch ring of 100 positions; positions 24, 27, 29, 74, 76, 78 have no bondpad (the custom pads are wider than one pitch) |
| IO cell types | 53 `gf180mcu_fd_io__asig_5p0_70x100`, 6 `dvdd_70x100`, 6 `dvss_70x100`, 4 `cor`; custom 23 `gf180mcu_liplib__no_esd_guarded_pad_70x100`, 2 `..asig_guard_5p0_pad_70x100`, 2 `..asig_guard_sense_5p0_pad_70x100_p105` (each of the last with two bondpads) |
| Standard cells / SRAM | none / none; no digital logic |
| Custom blocks | 4 single FETs, 2 MIM caps, 2 pad diodes, guard line, charge-pump pair, 3 inverters (6 FETs) inside the custom pads |
| Layers not in a plain digital die | Dualgate on 06v0 devices and IO cells; FuseTop (75/0) as MIM top plate; DNWELL (16 shapes, custom pads); Resistor/RES_MK (36/276), DIODE_MK, CAP_MK, MIM_L_MK markers. V5_XTOR (112/1) does not occur anywhere in the layout, not even in the foundry IO cells |
| Artwork | Text "CERN X JKU-IIC", "TRIDENT-GF180-TESTSTRUCTURE", "CLYDE LAFORGE" (Metal layers, x 590-1005, y 700-3112); wafer.space ID cell at (26,26), logo at (1767,4953) |

### Does it match the repository?

Yes. `klayout/top.layout.filled_db0p001.gds.gz` decompresses to SHA-256 `7d319382...0673`, equal to `project.json`. Compared with `layout/TRID.oas`: same top bounding box (0,0)-(1936,5122), 911 cells in both, identical instance counts for all 33 distinct top-level cell types (after removing `$n` suffixes), and zero XOR area for Poly2, Metal1-5, Dualgate, FuseTop and Pad. The only geometric difference is COMP (22/0): the reticle file has 2.22 mm2 more diffusion (88,950 polygons spread over the whole die interior, none missing from the reticle file), which is wafer.space's dummy fill. The repository file also stores 8 duplicate DIODE_MK marker rectangles that the OAS merges. The repository GDS is therefore the taped-out design.

![Whole die with the structures marked](images/overview.webp)

Whole die (1936 x 5122 um). Pad ring on all four sides; the test structures sit in the interior next to the pads that serve them (left edge: MIM caps near y 2550-2800 and charge pump near y 700-1150; right edge: four FETs x 1502-1520 near y 826-1986 and diodes near y 2492-2633; top: guard line at y 4287 and the custom pads with inverters; bottom: custom pads). Printed pad numbers (20, 25, 31, 51, 75, 81, 100) and the author's text are drawn in Metal.

![Top custom pads](images/toppads.webp)

Top edge, x 640-1320, y 4640-5110: the guard+sense pad (left, two bondpads = pads 23 and 25), a bare 3-position gap, and the guarded pad (right, bondpad = pad 28), each with its inverters at the foot of the pad cell.

![Inverters](images/inverters.webp)

The two inverters of the guard+sense pad at x 788-834, y 4678-4698: four 0.70 x 0.75 um gates, each in a Dualgate island (06v0 devices).

![FET gate](images/fet_hv_nfet.webp)

`hv_nfet_W160L070` gate at (1507.9, 1563.5): 0.70 um long, 1.6 um wide, with Dualgate (thick oxide) around it; gate, source and drain go to Metal2 leads and via stacks to the pad column on the right.

![LV FET gate](images/fet_lv_nfet.webp)

`lv_nfet_W160L070` gate at (1507.9, 933.5): same 0.70 x 1.6 um gate with no Dualgate.

![MIM](images/mim.webp)

The two MIM top plates (pink, FuseTop): 64 x 64 um at (478.9, 2739) and 256 x 16 um at (478.9, 2553).

![Diodes](images/diodes.webp)

The two pad diodes, `FOUNDRY_ESD_VDD` at (1450.8, 2492.2) (top) and `FOUNDRY_ESD_VSS` at (1450.5, 2597.0) (bottom): four 3 x 50 um fingers each, inside a guard ring; the VSS one carries Dualgate.

![Charge pump](images/pump.webp)

`matched_charge_pump_to_pads` gate array at (427-458, 927-937): ten 1.0 x 10 um poly gates in one Nwell.

### Test structures located in the layout

Pad numbers were obtained by counting 105 um positions from the author's net labels on the pad centres, anchored on the datasheet pin table. The count reproduces the datasheet pin numbers for the whole left edge, right edge and bottom edge and for pads 20, 23, 25, 30, 31; the exceptions (inverter output pads, guarded-pad pair, MIM guard pins) are listed under the caveats. The "pads reached" column is from the net extraction. "Foundry pad" = `gf180mcu_fd_io__asig_5p0_70x100`, "no-ESD" = `no_esd_guarded_pad_70x100`.

| Structure | Cell, position (um), size | Measured device | Pads reached (extracted nets) |
|---|---|---|---|
| LV NFET | `lv_nfet_W160L070`, (1502.0, 826.4), 18.2 x 214.2 | one gate 0.700 x 1.600 um, no Dualgate, no V5_XTOR; body tap on the VSS net | D 65, G 66, S 67 (no-ESD); guard Metal1 plane to pads 55 and 68 (shorted together on die) |
| LV PFET | `lv_pfet_W160L070`, (1502.0, 1141.4) | 0.700 x 1.600, no Dualgate; body on VDD net | D 62, G 63, S 64 |
| HV NFET | `hv_nfet_W160L070`, (1502.0, 1456.4) | 0.700 x 1.600, Dualgate over the gate, no V5_XTOR; body on VSS | D 59, G 60, S 61 |
| HV PFET | `hv_pfet_W160L070`, (1502.0, 1771.4) | 0.700 x 1.600, Dualgate, no V5_XTOR; body on VDD | D 56, G 57, S 58 |
| MIM 64 x 64 | `cap_mim`, (478.3, 2738.4), 65.2 x 65.2 | FuseTop top plate 64 x 64 = 4096 um2, perimeter 256 um; bottom plate Metal4 65.2 x 65.2 (4251 um2); top plate contacted from Metal5 through a Via4 array | bottom plate (Metal4) pad 2, top plate pad 3 (both no-ESD); Metal3 guard plane under the cap on pad 99 |
| MIM 16 x 256 | `cap_mim$3`, (478.3, 2552.4), 257.2 x 17.2 (long side along x) | FuseTop 256 x 16 = 4096 um2, perimeter 544 um; bottom plate Metal4 257.2 x 17.2 (4424 um2) | bottom plate pad 100, top plate pad 1; guard plane on pad 99 (same net as the other cap) |
| Guard line | drawn in `TOP`, x 415.8-1459.6, y 4287-4290 | Metal2, 3.00 um wide, 1043.8 um long; Metal1 strip below (4.2 um wide, x 418.6-1460.2) is the bottom guard; Metal3 strip above (4.2 um, same extent) is the top guard; two 0.30 um Metal2 side guards 0.28 um away on each side | hot pad 17 (no-ESD); top guard (Metal3) pad 14, bottom guard (Metal1) pad 15, side guards (Metal2) pad 16, pad-guard 18; pads 14-16 are foundry pads, i.e. the guards sit behind ESD clamps |
| VDD pad diode | `FOUNDRY_ESD_VDD`, (1450.8, 2492.2), 61.3 x 35.3 | P+ in Nwell, four fingers each 50 x 3 um (600 um2 total); N+ tap ring in the Nwell; no Dualgate | anode fingers pad 51, N+ ring (cathode) pad 49; the outer P+ ring is on a Metal1 net that reaches no pad |
| VSS pad diode | `FOUNDRY_ESD_VSS`, (1450.5, 2597.0), 35.8 x 61.8 | N+ in p-substrate/LVPWELL, four 3 x 50 um fingers (600 um2), Dualgate over the fingers; outer Nwell ring tied to the VDD net | N+ fingers (cathode) pad 50; the P+ ring (anode) is on the VSS supply net (pads 19, 31, 53, 69, 81, 97) |
| Charge pump | `matched_charge_pump_to_pads`, (415.8, 701.3), 45.5 x 444.3 | ten 1.00 x 10.0 um poly gates in one Nwell, no Dualgate: 4 on G1, 4 on G2, 2 end dummies tied to VS; all diffusion between gates on one net VS; Nwell/bulk on VW; Metal1 guard on pad 83 | G1 86, G2 84, VS 85, VW 87 (VS, VW no-ESD; G1, G2, guard foundry pads) |
| Guard+sense pad, loaded | `..guard_sense..p105` at (660.3, 4705.7), 300.3 x 391.5 | two inverters, all four gates 0.700 x 0.750 um in Dualgate | hot in 25 feeds the hot inverter, sense in 23 feeds the sense inverter; inverter outputs go to the pads at positions 22 (hot) and 21 (sense); guard 30; inverters powered from the VDD/VSS pad nets |
| Guarded pad, loaded | `..asig_guard_5p0..` at (1160.3, 4705.1), 140.3 x 392.1 | one inverter, 0.700 x 0.750, Dualgate | in 28, out 26; guard 30 |
| Guard+sense pad, unloaded | same cell at (975.3, 24.8) | custom pad only | hot 75, sense 73, guard 71/80 |
| Guarded pad, unloaded | same cell at (740.3, 24.8) | | hot 77, guard 71/80 |
| Foundry pad, unloaded | `asig_5p0_70x100` | | hot 79 |
| Empty no-ESD pad | `no_esd_guarded_pad_70x100` at (1292.8, 24.8) | | 72 |
| Supplies | 6 `dvdd`, 6 `dvss` | | VDD 20, 32, 54, 70, 82, 98 and VSS 19, 31, 53, 69, 81, 97 all extract to two nets (one VDD, one VSS) |
| Unused pads | foundry pads, core pin open | | pads 4 (labelled MIM GUARD2), 5-7, 9-13, 33-48, 88-96: each is an isolated net with only its own pad; pad 8 (labelled MIM GUARD1) is not connected to either cap |

## Authors and affiliations

| Person / organisation | Role | Affiliation | Source |
|---|---|---|---|
| Clyde Laforge (GitHub `Scafir`) | Sole designer: all 64 design commits, schematics, layout, datasheet | CERN; "PhD student at CERN x JKU working on open source analog ASICs", Geneva | `gh api users/Scafir` (company, bio); commit email domain cern.ch; README "CERN+JKU PhD collaboration" |
| CERN (HSE-RP-IL per datasheet page header) | Copyright holder of all original work | CERN | SPDX headers; datasheet header "ACCURATE 2A / CERN HSE-RP-IL" |
| JKU Linz | Academic partner of the PhD | Johannes Kepler University | README only; no JKU person is named in the repo |
| Leo Moser | Made changes to the filled layout before tapeout (commit 3b7bbea "add changes by Leo Moser prior to tapeout"); author of the template and of the ID/logo IP | not stated in this repo | commit message, `REUSE.toml` |
| Harald Pretl, M. Eberlein; U. Cilingiroglu et al. | Cited as the origin of the charge-pump current source circuit (IEEE SSC Magazine 2021, JSSC 2003), not contributors | not stated | datasheet section 7.13 |

The other names in `git shortlog` (Leo Moser 32 commits, Tim Ansell 1, TrevorPeyton 1) are inherited template history from September and October 2025, not work on this design.

## Value for GF180MCU characterisation and PDK development

### Verdict: High

This is a purpose-built device characterisation chip: every pad goes to a discrete device or pad structure with dedicated guard pins, the taped-out GDS is in the repo with a matching hash and was checked against the reticle OAS (identical except wafer.space fill), and a datasheet gives the pin table and bonding diagrams. It directly measures things the open PDK models are weakest on (off-state and junction leakage at pA level, MIM dielectric leakage, IO-pad diode leakage).

### Test structures and measurable circuits

Pin numbers and pin names are from the datasheet pin table; net names are from `xschem/top.sch` and the GDS text labels (layer 81). Geometries and pad connectivity were re-measured from the taped-out layout (see the "Taped-out layout" section); where the layout disagrees with the datasheet or schematic the row below says so. "No-ESD pad" means `gf180mcu_liplib__no_esd_guarded_pad_70x100`, a bare bondpad with a guarded metal lead and no devices.

| # | Structure (datasheet section) | Device and geometry | Pads | Measurement it is designed for |
|---|---|---|---|---|
| 1 | LV NFET (7.8), cell `lv_nfet_W160L070` | `nfet_03v3`, W=1.6 um, L=0.7 um, nf=1, m=1; bulk to VSS | D 65, G 66, S 67 on no-ESD pads; guard 55/68 (`guard_transistors_PAD`, foundry asig pads, internally shorted) | Id-Vg, Id-Vd, subthreshold slope, off-state, gate and junction leakage of a single 3.3 V NMOS with driven guard |
| 2 | LV PFET (7.8), cell `lv_pfet_W160L070` | `pfet_03v3`, W=1.6 um, L=0.7 um; bulk to VDD | D 62, G 63, S 64; guard 55/68 | Same for 3.3 V PMOS |
| 3 | HV NFET (7.8), cell `hv_nfet_W160L070` | `nfet_06v0`, W=1.6 um, L=0.7 um (PCell: volt 6V, w_gate 1.6, l_gate 0.7, nf 1); bulk to VSS | D 59, G 60, S 61; guard 55/68 | Same for 6 V NMOS |
| 4 | HV PFET (7.8), cell `hv_pfet_W160L070` | `pfet_06v0`, W=1.6 um, L=0.7 um (PCell confirmed); bulk to VDD | D 56, G 57, S 58; guard 55/68 | Same for 6 V PMOS |
| 5 | MIM capacitor, minimum perimeter (7.2), PCell `cap_mim` | `cap_mim_2f0fF`, MIM-B option, 64 um x 64 um, area 4096 um2, perimeter 256 um, nominal 8192 fF (areas and perimeters measured from the FuseTop top plate: confirmed) | Layout: bottom plate (Metal4) pad 2, top plate (FuseTop/Metal5) pad 3, no-ESD pads; Metal3 guard plane under it on pad 99 (the datasheet gives pad 8 or pin 4 as this guard; pad 8 does not reach the cap) | MIM dielectric leakage and capacitance; area-dominated reference |
| 6 | MIM capacitor, maximum perimeter (7.1), PCell `cap_mim$3` | `cap_mim_2f0fF`, MIM-B, 16 um x 256 um, same area 4096 um2, perimeter 544 um, nominal 8192 fF | Layout: top plate pad 1, bottom plate (Metal4) pad 100, no-ESD pads; guard plane on pad 99, same net as the 64 x 64 cap's guard | Same area as #5 with 2.1x the perimeter, to separate area and perimeter components of MIM leakage (and of capacitance) |
| 7 | Guard-leakage line (7.3), drawn directly in `TOP` | Metal2 line, 3.00 um x 1043.8 um (the datasheet's "metal XXX" is Metal2), with a Metal3 top guard, Metal1 bottom guard and 0.3 um Metal2 side guards, measured and traced | Hot 17 (`GUARD_IMPORTANCE_INPUT_PAD`, no-ESD pad), top guard 14, bottom guard 15, side guard 16, bondpad guard 18 | Inter-metal and intra-metal dielectric leakage versus guard voltage; how much each guard plane contributes |
| 8 | Discrete ESD diode to VDD (7.6), cell `FOUNDRY_ESD_VDD` | `diode_pd2nw_06v0`; layout: four P+ fingers 3 x 50 um (600 um2), no Dualgate | Anode 51, cathode 49 on no-ESD pads; guard 52 | Ultra-low-current I-V of the P+/Nwell pad diode, both terminals free |
| 9 | Discrete ESD diode to VSS (7.7), cell `FOUNDRY_ESD_VSS` | `diode_nd2ps_06v0`; layout: four N+ fingers 3 x 50 um (600 um2), Dualgate. The schematic's 30 um is not what was drawn: both diodes are the same size | Cathode 50 on no-ESD pad; anode is substrate/VSS; guard 52 | Reverse leakage and forward I-V of the N+/Psub pad diode |
| 10 | Foundry analog pad, unloaded (7.12) | `gf180mcu_fd_io__asig_5p0` with 70x100 bondpad, core pin open | Hot pin 79 (layout pad 79 confirmed); guard 71/80 | Intrinsic leakage of the stock analog pad (double-diode plus clamp); also usable as an ESD robustness sample |
| 11 | Custom guarded ESD pad, unloaded (7.11) | `gf180mcu_liplib__asig_guard_5p0_pad_70x100`: pad diodes referenced to a driven GUARD node instead of the rails (`diode_nd2ps_06v0`, `diode_pd2nw_06v0`, isolated VSS diode in deep N-well, `ppolyf_u_1k_6p0` series and guard resistors, `cap_nmos_06v0`); diodes sized 2x the foundry pad | Hot `PL_G_HOT` pin 77; guard 71/80 | Pad leakage with near-zero bias across the input diodes |
| 12 | Custom guard+sense ESD pad, unloaded (7.10) | `gf180mcu_liplib__asig_guard_sense_5p0_pad_70x100_p105`: #11 with a symmetric, interleaved replica (SENSE) path, matched `ppolyf_u_1k` resistors, custom interleaved diodes (`distributed_diodevddx22`, `distributed_isolated_diodevssx22`, `cdm_*_diodex10`) | Hot `PL_GS_HOT` pin 75, sense `PL_GS_SENSE` pin 73; guard 71/80 | Whether replica leakage tracks pad leakage, so the guard voltage can be trimmed to null it |
| 13 | Empty no-ESD pad (7.9) | Bare guarded bondpad, nothing connected | `PL_FLOAT` pin 72; guard 71/80 | Baseline: bondpad, package and board leakage to subtract from every other no-ESD-pad measurement |
| 14 | Guard+sense pad driving inverters (7.4) | Pad #12 with its hot and sense outputs each driving an inverter `hv_inverter_W075L070` (`nfet_06v0` and `pfet_06v0`, W=0.75 um, L=0.7 um; gates measured 0.70 x 0.75 um, all inside Dualgate) | Hot in 25, sense in 23, in layout: inverted hot out 22, inverted sense out 21 (datasheet says 21 and 22); guard 30 | ESD zap level at which a core gate behind the custom pad fails; functional check of the pad path |
| 15 | Guarded pad driving an inverter (7.5) | Pad #11 driving one `hv_inverter_W075L070` | Layout: hot in at position 28, inverted out at 26 (datasheet says 29 and 27, which are positions with no bondpad); guard 30 | Same as #14 for the pad without sense line |
| 16 | pA charge-pump current source (7.13), cell `matched_interface_trap_charge_pumps` | Two matched `pfet_03v3`, each W=10 um, L=1 um, m=4 (40 um total), plus a dummy `pfet_03v3` 10/1 m=2 (layout: ten 1.0 x 10 um gates, 4 + 4 + 2 dummies, all confirmed; all diffusion on node VS, no Dualgate); common drain+source node VS, common bulk VW, separate gates | G1 86, G2 84 (foundry asig pads), VS 85, VW 87 (no-ESD pads), guard 83 | Interface-trap charge pumping current per gate cycle (Pretl and Eberlein "fifty nifty" pA source; Cilingiroglu JSSC 2003); gives interface trap density of the 3.3 V PMOS and tests two-phase operation for noise cancellation |
| 17 | Supplies | 6 `dvdd` and 6 `dvss` 70x100 pads | VDD 20, 32, 54, 70, 82, 98; VSS 19, 31, 53, 69, 81, 97 | Rail and substrate bias for the pad ring and bulks |
| 18 | Unused pads | Foundry asig pads with open core pin (`TOFILL*_PAD`) | 4-7, 9-13, 33-48, 88-96 | Spare samples of the stock analog pad for leakage statistics or destructive ESD tests (extraction: each is an isolated net with nothing behind it; the use as spare samples is inferred, the datasheet says "do not connect") |

Devices and domains: `nfet_03v3`, `pfet_03v3`, `nfet_06v0`, `pfet_06v0`, `cap_mim_2f0fF` (MIM-B), `diode_nd2ps_06v0`, `diode_pd2nw_06v0`, `ppolyf_u_1k`, `ppolyf_u_1k_6p0`, `cap_nmos_06v0`, and the 5 V `gf180mcu_fd_io` pad cells. There is no on-chip mux or scan; every terminal is a bond pad. Signals of interest avoid ESD devices entirely (no-ESD pads) and were routed up and down the metal stack to satisfy antenna rules.

Simulation results to compare against: none in the repo. The only simulation setup is `xschem/interface_trap_charge_pump_cascode.sch` (ngspice transient, `sm141064.ngspice typical`), with no saved output. Expected leakage values would have to be generated from the PDK models by the reader.

### Candidate PDK additions

- Low-leakage pad library (liplib): a no-ESD guarded pad, a guarded analog ESD pad and a guard+sense analog ESD pad, all on the 70x100 um, 105 um-pitch pad frame. Layout is in this GDS (and in the taped-out OAS), source in https://github.com/Scafir/liplib (CERN-OHL-W-2.0). These would be new IO cells for precision-analog users.
- `gf180mcu_bondpad_70x100um` and 70x100 variants of `gf180mcu_fd_io` (`asig_5p0`, `dvdd`, `dvss`, `bi_24t`, `in_c`, `in_s`) with xschem symbols and a SPICE netlist (`xschem/gf180mcu_fd_io_70x100.spice`).
- LVS fix for the foundry IO cells: marker layers (LVWELL and an isolated-substrate marker) had to be added to `gf180mcu_fd_io_pad_70x100.gds` before the KLayout LVS deck would recognise the pad diodes (commit 235f7c3).
- Flow problems the author hit, from commit messages and a note in `top.sch`: KLayout LVS does not accept several pins on one net (VDD/VSS pins had to be left blank); Magic is confused by magnified instances (7330502); zero-area polygons had to be removed; the precheck needs database unit 0.001 um and flattened PCells, while the design was drawn at 0.005 um; DRC rule CUP.3 fired on the custom pad (ee8110f); DRC rules changed during the run (ab38528 "Corrected new DRC rules"); the foundry made changes that were backported to the layout afterwards (b168d1f).
- If measured, the results would give data the PDK lacks: MIM-B leakage split into area and perimeter terms, pad-diode leakage, off-state leakage of 3.3 V and 6 V devices at pA resolution, and interface trap density.

### What would be needed to use it

- Wire-bonded die on a low-leakage PCB. The datasheet gives two bonding variants (A and B), each reaching a subset of structures, with bond-wire coordinates, lengths and angles and a staggered PCB pad pattern. No PCB design files are in the repo.
- Electrometer or SMU with fA to pA resolution, triaxial cabling, driven guards, a shielded and temperature-controlled fixture. For the charge pump, a two-phase gate pulse source. For the ESD structures, an HBM/TLP tester (destructive).
- A test plan and expected values: neither is in the repo. The datasheet describes intent per structure but gives no bias conditions, limits or absolute maximum ratings.
- Bare-die probing is possible in principle (bondpads 78 um x 100 um at 105 um pitch per the datasheet) but probe-station leakage would have to be below the quantities being measured.
- Measurement data, if it exists, is presumably at https://cern.ch/gftrid, which redirects to a CERN single-sign-on page and could not be read.

## Notes and caveats

- Transistor size conflict: the datasheet heading 7.8 says "W=0.75um L=0.7um" and the pad net names say `W075L070` (one GDS label still says `lv_pfet_W030L070_source_PAD`), but the schematics and the layout PCell parameters both say W=1.6 um, L=0.7 um. Settled from the layout: the gate of each of the four transistors is 0.700 um long and 1.600 um wide (poly over diffusion measured in each cell); only the inverter devices are 0.70 x 0.75 um. The 06v0 pair carries Dualgate over the gate and the 03v3 pair does not. The V5_XTOR marker (112/1) is not drawn on the 06v0 devices (it does not exist anywhere in the layout, not even in the foundry IO cells).
- The datasheet calls the inverter devices "low voltage transistors"; the schematic uses `nfet_06v0`/`pfet_06v0` and the layout confirms it: all six inverter gates are inside Dualgate.
- Datasheet internal inconsistencies: section 7.2 lists the minimum-perimeter MIM guard as pin 4, the pin table has pin 4 unused and the P2 guard on pin 8; the layout has labels MIM_GUARD2 on pad 4 (an unconnected foundry pad) and MIM_GUARD1 on both pads 8 and 99, but only the net of pad 99 reaches the capacitors (a Metal3 plane under both), so pad 99 is the working guard for both and pads 4 and 8 reach nothing; section 7.12 lists the foundry-pad hot input as pin 77, the pin table says 79 (77 is the guarded pad); the charge-pump guard net is `CHARGE_PUMP_GUARD_PAD` in `top.sch` and `GUARD_CHARGE_PUMP_PAD` in the stale netlist. Placeholders "ADDREF" and "metal XXX" remain in the text.
- Pin-1 marking errata, confirmed in the layout: the printed numbers are Metal artwork beside the pads. The "1" is drawn at y 2474-2544 beside the pad that is pin 100 (y 2508), the "2" at y 2580-2650 beside pin 1 (y 2613) and the "100" at y 2372-2442 beside pin 99 (y 2403), i.e. exactly the datasheet's off-by-one.
- Inverter output and guarded-pad pin numbers: counting 105 um positions from the datasheet's own anchors (pad 23 sense in, 25 hot in, 30 guard), the layout puts the hot inverter's output on pad 22 and the sense inverter's output on pad 21 (datasheet: 21 and 22), and the guarded pad's input and output on 28 and 26 (datasheet: 29 and 27; those two positions carry no bondpad in the layout, as do 24, 27, 29, 74, 76, 78). The net labels are the author's; the connectivity (gate of each inverter on the correct input pad) is extracted. A bonding diagram check would confirm which numbering the PCB uses.
- Diode sizes: settled from the layout. Both pad diodes are four 3 x 50 um fingers (600 um2); the schematic's 30 um for the VSS diode was not drawn.
- `klayout/top.schematic_lvs.spice` is out of date and should not be used as the reference netlist; `xschem/top.sch` is current. The custom pad schematics needed to re-run LVS are only in the external liplib repo.
- Branch `origin/complete` holds an earlier state that still has the template `librelane/` and `src/` files and an older, smaller layout; `main` is the tapeout state.
- Pad-to-structure mapping: now checked against the layout (position count plus extracted nets), see the table in the Taped-out layout section. Not checked: body (well/substrate) connections beyond the Metal1 taps, the poly resistors inside the custom pads (they separate the pad node from the core node in the extraction, so pad-to-diode paths were not followed), and the charge pump gate-to-pad assignment beyond G1/G2 net names.
