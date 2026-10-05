# ISHI: ISHI-KAI's Multiple Users Project (ws-run1)

| | |
|---|---|
| Repository | https://github.com/ishi-kai/ISHI-KAI_Multiple_Projects_WaferSapce-GF180-1 @ ae2164f (main, last commit 2026-05-08) |
| Slot | 1x1 (die 3932 x 5122 um measured from the GDS) |
| Top cell | ISHI-KAI_WS_RUN1 |
| Licence in repo | Apache-2.0 (`LICENSE`, "Copyright (c) 2025 Inter-linked Society on Homemade IC Kai"); no SPDX headers |
| README | OK, 2089 words, original (English), but most of it describes files that live in other repositories |

## What the repo actually contains

A hand-assembled, full-custom analog multi-project die from the Japanese
ISHI-KAI community: 15 separate circuits from 14 named members, each wired both
to its own row of on-die probe pads and to the outer GF180MCU IO ring. There is
no digital flow, no RTL and no LibreLane run; the only automated block is the
SAR logic inside the ADC, which was synthesised earlier with OpenLane on
`gf180mcu_fd_sc_mcu7t5v0`. Everything was drawn in xschem and KLayout (PCells),
then flattened.

**This repo is a layout-and-logs repo only.** It holds no schematics, no
netlists, no testbenches and no simulation output. `schematic/README.md` is
seven links to three other repositories. What is here:

| Path | Content |
|---|---|
| `gds/ISHI-KAI_WS_RUN1.zip` | Final top GDS (283 MB unzipped, 997 cells) including fill |
| `gds/ISHI-KAI_WS_RUN1_nofiller.zip`, `..._nofiller_delContext.zip` | Same without dummy fill (the two unzip to the same size) |
| `gds/frame_ISHI-KAI.gds`, `frame_ISHI-KAI_noIO.gds` | Pad ring / frame only |
| `gds/member_project/*.gds` | One flattened GDS per block as received from members (11 files, `ldo_noritsuna` zipped), plus `probe_pads.gds` |
| `gds/chip_arts/logo_ISHI-KAI.gds` | Logo, drawn as 325,554 polygons on the Pad (37/0) layer |
| `docs/pin_list.xlsx`, `pin_list.pdf` | 74-row pad list: pin number, block, signal, IO cell type |
| `precheck_log/RUN_*` | Ten wafer.space precheck runs, 2025-11-12 to 2025-12-16 |
| `images/` | Annotated layout picture, precheck screenshots, KLayout crash screenshots |
| `chip_photo/` | 29 microscope photos of fabricated dies, 2 photos of the bare-die tray, 2 of chip-on-board assemblies |

The linked source repositories (not cloned here; I fetched them read-only to
identify devices) are:

- `ishi-kai/ISHI-KAI_Multiple_Projects_OpenGFMPW-1`: inverters, op-amps, comparator (originally prepared for GFMPW-1, which did not accept the design)
- `ishi-kai/Chipathon2023_ADC`: SAR ADC and LDO (SSCS Chipathon 2023, Japan team)
- `atuchiya/DC23-LTC2` branch `japan-test`: PLL, bandgap, current source (fork of `sscs-ose/DC23-LTC2`)

### Sub-designs

"Devices" is taken from the upstream xschem schematics; "as fabricated" notes
come from layer counts in the top GDS of this repo.

| # | Cell in top GDS | Designer | Circuit | Devices (schematic) | Sch | Layout | Sim |
|---|---|---|---|---|---|---|---|
| 1 | `inverter_all` | noritsuna | CMOS inverter | nfet_05v0 3.6/0.6, pfet_05v0 7.2/0.6 | upstream | here | upstream testbenches incl. PEX; no stored results |
| 2 | `inverter_all` | ajisai | CMOS inverter | nfet_06v0 2.0/0.7, pfet_06v0 5.0/0.7 | upstream | here | none |
| 3 | `inverter_all` | homelith | CMOS inverter, deliberately weak PMOS | nfet_06v0 3.0/0.6, pfet_05v0 2.8/0.56 | upstream | here | upstream `inv_with_sim.sch` (5 V) |
| 4 | `inverter_all` | bols-blue | CMOS inverter | nfet_05v0 3.6/0.7, pfet_05v0 14.4/0.6 | upstream | here | none |
| 5 | `inverter_all` | ponzu840w | CMOS inverter | nfet_05v0 3.6/0.6, pfet_05v0 14.4/0.6 | upstream | here | upstream `inv_tb.sch` |
| 6 | `opamp_3zki` | 3zki (Mizuki Mori) | Rail-to-rail op-amp after Baker fig. 24.48, VDD 5 V, VB 2.5 V | 25 nfet_06v0 (L=1 um, W 1 to 50 um), 25 pfet_06v0 (L=1 um, W 2 to 100 um), 2 cap_mim 64x8 um | upstream | here | none ("no documents or measurements were created") |
| 7 | `opamp_gitefu` | gitefu | Two-stage op-amp, PMOS input, 5 V | nfet_06v0_dn / pfet_06v0_dn L=0.7 um, cap_mim 39x19.23 um | upstream | here | upstream `op_tb.sch`, PEX netlist, netgen log |
| 8 | `opamp_noritsuna` | noritsuna | Two-stage op-amp, 5 V, IB 20 uA | nfet_05v0_dn L=0.6, pfet_05v0 L=0.56, cap_mim 50x10 um | upstream | here | upstream `opamp_ac.sch`, `opamp_PEX.sch`, 3.2 MB PEX netlist |
| 9 | `opamp_yamada3` | yamada3 (Miho Yamada) | Two-stage op-amp "2L_5V" | 6 nfet_05v0_dn L=0.6, 6 pfet_06v0 L=0.5, cap_mim 100x50 um | upstream | here | none |
| 10 | `cmp_ice458` | ice458 | Clocked dynamic comparator with RS latch, 30-inverter delay line, output buffers | nfet_06v0 1/0.7, pfet_05v0 3/0.7, 3 cap_mim 15x15 um, 66 sc7 5 V standard cells | upstream | here | upstream `cmp_only_tb_5v.sch` (run at 3.3 V) |
| 11 | `SAR_ADC` | Maple0705 (SAR logic, integration), Gitefu (comparator), Tseng (CDAC), Kinako (latch, switches) | 6-bit SAR ADC, 8 MHz clock, 3.3 V | nfet_03v3 / pfet_03v3 L=0.28 (comparator, switches, latch), 64 cap_mim_2f0_m4m5_noshield 7x7 um (8x8 array), 798 `gf180mcu_fd_sc_mcu7t5v0` cells | upstream | here | upstream transient CSVs and PEX netlist |
| 12 | `ldo_noritsuna` | noritsuna, with bandgap by A. Tsuchiya, current source by M. Yamada, waffle PFET by Akiles Esta Usado | LDO, 3.3 V in, 3.2 V / 4 mA out, 700 x 650 um | pass device pfet_03v3 4.38/0.5 m=7936 (waffle layout), nfet_03v3, 2 ppolyf_u 0.8 um x 10 mm, error amp in nfet/pfet_03v3 L=0.56 with cap_mim 30x30, npn_10p00x10p00 bandgap | upstream | here | upstream `ldo_pex_tb.sch`, 49 MB PEX netlist |
| 13 | `chipathon2023_pll` | A. Tsuchiya | Analog PLL, 8 MHz in, 48/40 MHz out, 3.3 V | VCO ring and charge pump in nfet_03v3/pfet_03v3 L=0.33 (one 0.28), transmission gates and bias inverters in nfet_06v0/pfet_06v0, sc7 5 V cells for PFD and dividers, loop filter in high-res poly (Resistor layer present, 33 shapes) and 22 MIM caps | upstream | here | upstream `pll_bench.sch`; no stored results |
| 14 | `chipathon2023_CS_BGR` (BGR) | A. Tsuchiya | Bandgap, 1.2 V target, 108 x 200 um | 18 npn_10p00x10p00 (1+1+16), nfet_03v3/pfet_03v3 11.2/1.12, 12 ppolyf_u | upstream | here | README figures: 43 mV over VDD 3.0 to 3.3 V, 1.2 mV over -50 to 125 C |
| 15 | `chipathon2023_CS_BGR` (CS) | yamada3 (Miho Yamada), from A. Tsuchiya's `cs_vthref` | Vth-referenced current source, 20 uA | nfet_03v3_dn, pfet_03v3 5.6/0.56, 4 ppolyf_u 0.8x22 um | upstream | here | README figures: 0.38 uA over VDD 3.0 to 3.3 V, 3.2 uA over -50 to 125 C |

Also on the die: `logo_ishikai` (1157 x 771 um), 12 instances of the plain
7-pad probe row `pads_ishikai`, and a 680 x 680 um single Pad opening at
(1217, 1723) to (1897, 2403) over eight abutted pad rows. The purpose of that
large bare-metal square is not documented.

### Frame and verification

Pad ring: 74 pads from `gf180mcu_fd_io` (39 `asig_5p0`, 9 `bi_24t`, 5 `in_c`)
plus 15 `gf180mcu_ws_io__dvdd` and 6 `gf180mcu_ws_io__dvss`, on 5 metal layers.
Small glue cells `ANALOG_CON`, `D_BI_T_CON`, `D_IN_C_CON`, `VDD_VSS_CON` connect
pads to the core; the digital ones strap pad control pins with `tieh`/`tiel`
cells from the 7-track 5 V library.

The last precheck (`RUN_2025-12-16_22-51-46`, PDK 1.6.6, precheck 1.5.4 per
commit message) reports zero density, zero-area, antenna, Magic DRC and KLayout
DRC errors. Earlier runs record a KLayout 0.30.4 crash in the deep-mode DRC
(`RUN_2025-11-21`, stack trace in `error.log`) and two SIGKILLs (screenshots).

There is no LVS of the assembled die, and no top-level schematic or netlist
anywhere. Per-block LVS exists only upstream and only for some blocks.

Silicon: the tray photo shows a "G801 ISHI 1x1" label with 308 bare dies, and a
panel of ten chip-on-board assemblies on wafer.space "70 pin mezzanine, 74 pin
padring" boards. **No electrical measurement data has been published in this
repo.**

## Authors and affiliations

| Person or organisation | Role | Affiliation | Source |
|---|---|---|---|
| ISHI-KAI (Inter-linked Society on Homemade IC Kai) | Organiser, copyright holder | Community, https://ishi-kai.org/ | `LICENSE`, `gh api orgs/ishi-kai` |
| Noritsuna Imamura (`noritsuna`) | Sole committer (33 commits), die integration, frame, fill, precheck; inverter, op-amp, LDO | SIProp.org | git log; GitHub profile company field |
| Akira Tsuchiya (`atuchiya`) | PLL, bandgap, original current source; author of the op-amp tutorial others followed | Not stated (README calls him "Prof. Tsuchiya") | README, upstream READMEs |
| Mizuki Mori (`3zki`) | Rail-to-rail op-amp; supporter on the ADC | Not stated ("hobbyist interested in LSI") | README, upstream ADC README, GitHub bio |
| Miho Yamada (`keropiyo`, "yamada3") | Op-amp, current source | Not stated | README, upstream LDO README |
| `Gitefu` | Op-amp, ADC comparator | Not stated; self-described graduate student in processor research, member of "tofunology" | README |
| `ice458` | Dynamic comparator | Not stated | README |
| `maple0705` | SAR logic, ADC integration | Not stated ("semiconductor engineer") | upstream ADC README |
| `kinako71` (Kinako) | ADC latch and switches | Not stated (graduate student) | upstream ADC README |
| `tsengs0` (Tseng) | ADC MIM capacitor array | JAIST | GitHub profile company field |
| `homelith`, `bols-blue` (h.suehiro), `ponzu840w`, ajisai | One inverter each | Not stated (ponzu840w: electronics undergraduate) | README, GitHub profiles |
| Takeshi Kuboki | Supporter on the ADC | Not stated | upstream ADC README |
| Akiles Esta Usado | Waffle PFET array reused in the LDO | Chipathon 2023 Chile team | README |

The README link for ponzu840w points at `github.com/mulwak`, which returns 404.

## Value for GF180MCU characterisation and PDK development

### Verdict: High

Every circuit is device-level analog with dedicated probe pads and a pad-ring
connection, covering 3.3 V, 5 V and 6 V MOS, vertical NPN, unsalicided and
high-res poly resistors and MIM capacitors, and upstream simulations exist for
most of them. The value is reduced by the absence of any device-level test
structure (no single transistors, no resistor or capacitor on its own), by
missing documentation of the probe-pad order for several blocks, and by a
layout-versus-schematic device mismatch in three op-amps (below).

### Test structures and measurable circuits

Probe pads are 80 x 80 um openings on 100 um pitch in rows of seven. Coordinates
below are pad centres in um from the GDS origin (lower-left die corner),
extracted from `ISHI-KAI_WS_RUN1_nofiller.gds`; names are the Metal5 text
labels inside each cell. Ring pin numbers are from `docs/pin_list.pdf`; I
checked the cell types against the layout and pin 1 is the bottom-left pad,
counting counter-clockwise.

| Structure | Exercises | Probe pads | Ring pins | Reference simulation |
|---|---|---|---|---|
| 5 inverters | VTC, switching threshold, drive of nfet/pfet_05v0 and _06v0 at L 0.56 to 0.7 um; homelith's is intentionally skewed | Five columns at x = 1255.4, 1673.7, 2053.3, 2432.9, 2812.8, y = 708.5 to 1308.5; pads carry no labels | 1 to 16 (VDD, OUT, IN per inverter, shared VSS on 16) | Testbenches upstream; no numbers |
| `opamp_3zki` | 6 V MOS matching, gain, rail-to-rail input range, MIM compensation | x = 3360.4; y = 708.5 VSS, 808.5 OUT, 908.5 VINP, 1008.5 VDD, 1108.5 VINN, 1208.5 VB, 1308.5 VSS | 17 to 22 | None |
| `opamp_gitefu` | Two-stage op-amp DC gain, offset, GBW | x = 3360.1; y = 1800.7 VSS, 1900.7 OUT, 2000.7 VINP, 2100.7 VDD, 2200.7 VINN, 2300.7 IB, 2400.7 VSS | 23 to 27 | Upstream testbench and PEX netlist, for 6 V devices at 5 V |
| `opamp_noritsuna` | Same | x = 3360.1; y = 2836.2 VSS, 2936.2 OUT, 3036.2 VINP, 3136.2 VDD, 3236.2 VINN, 3336.2 IB, 3436.2 VSS | 28 to 32 | Upstream AC testbench and PEX netlist, for 5 V devices |
| `opamp_yamada3` | Same | x = 3360.2; y = 3877.9 VSS, 3977.9 OUT, 4077.9 VINP, 4177.9 VDD, 4277.9 VINN, 4377.9 IB, 4477.9 VSS | 33 to 37 | None |
| Current source | nfet_03v3 Vth, ppolyf_u sheet resistance and tempco, pfet_03v3 mirror | Shared column x = 2905.8, y = 3818.3 to 4418.3, unlabelled | 38, 39, 42 | 20 uA; supply and temperature sweeps in upstream README |
| Bandgap | npn_10p00x10p00 Vbe and area ratio 16:1, ppolyf_u ratio | Same shared column | 40, 41, 42 | 1.2 V; supply and temperature sweeps in upstream README |
| SAR ADC | MIM unit-cap matching (8x8 of 7x7 um), 3.3 V comparator offset, 5 V standard cells run at 3.3 V | Two rows y = 4189.2 and 3889.2, x = 1936.6 to 2536.6, unlabelled | 43 to 54 | Upstream transient CSVs (`submit_version/xschem/sim_result/`), PEX netlist |
| Dynamic comparator | Offset and regeneration time, 30-stage inverter delay | x = 828.5: y = 3515.1 inp, 3615.1 inm, 3715.1 vcm, 3815.1 vdd, 3915.1 clk; x = 510.3: y = 3515.1 out, 3615.1 outn, 3715.1 outp, 3815.1 done; VSS at both ends of each column | 55 to 64 | Upstream 5 V testbench |
| PLL | Ring-VCO frequency versus control voltage (`VCTRL` pad, `FREERUN` mode), i.e. 3.3 V inverter delay; high-res poly and MIM in the loop filter; lock at 48/40 MHz | Column x = 777.2, y = 1763.4 to 2363.4, unlabelled; one position also carries a 5x5 array of 2 um openings | 65 to 71 | Testbench upstream; no numbers |
| LDO | Large pfet_03v3 waffle array on-resistance, 10 mm ppolyf_u divider, loop regulation | Column x = 501.2, y = 559.8 to 1159.8; VSS at both ends, four VOUT, one unlabelled (VDD, inferred) | 72 to 74 | Upstream PEX testbench |

The VCO in free-run mode is the nearest thing on the die to a ring-oscillator
process monitor. The bandgap and the Vth-referenced source are the most direct
checks of the BJT and resistor models, since upstream gives predicted numbers.

Supply domains: Chipathon blocks (ADC, LDO, PLL, bandgap, current source) are
3.3 V; the inverters, comparator and op-amps were designed for 5 V. All analog
signals go through `asig_5p0` pads.

### Candidate PDK additions

- `pads_ishikai`: a 7 x (80 um on 100 um pitch) full-metal-stack probe-pad row, reused 12 times and embedded in every member cell. A natural seed for a standard probe-pad PCell.
- Waffle-layout PFET array (7936 units of pfet_03v3 4.38/0.5) and the 8x8 MIM array, both upstream.
- Pad glue cells that strap `bi_24t` and `in_c` control pins with tie cells so digital pads can be used from a hand-drawn analog core.
- PDK and tool problems the authors recorded:
  - No SPICE model for the 5 V PFET; ice458 substituted the 6 V model for simulation (upstream comparator README, Japanese and English).
  - Schematic and extracted MIM names differ (`cap_mim_2f0fF` against `cap_mim_2f0_m4m5_noshield`), which makes netgen report a mismatch; members wrote private `cap_mim_2p0fF_lvs.sym` and `ppolyf_u_lvs.sym` symbols to work around LVS naming.
  - Text labels made the older precheck fail, so label-free GDS copies were submitted.
  - KLayout 0.30.4 crash in deep-mode DRC and SIGKILLs on this 280 MB layout.
  - Commit messages track fixes forced by PDK updates 1.1.0 to 1.6.6: filler script rewrite, seal ring, zero-area polygons, wide-metal slots, top-cell naming.

### What would be needed to use it

- Bare-die probing with a 7-pin 100 um pitch probe or individual needles, or the wafer.space chip-on-board assemblies using the ring pins. Ring access adds `asig_5p0` ESD and series resistance to every analog node.
- A probe-pad map for the unlabelled rows (inverters, ADC, PLL, current source and bandgap); it has to be traced from the layout.
- A corrected ring pin list (see below).
- Source-measure units and a scope; an 8 MHz clock for the ADC and PLL; a 20 uA bias source for three of the op-amps.
- A test plan. None exists.
- Re-simulation of three op-amps with the devices actually drawn (see below) before comparing anything.

## Notes and caveats

- **Three op-amps were changed from 5/6 V to thin-oxide devices during integration, without documentation.** In the member files (`gds/member_project/opamp_gitefu.gds`, `opamp_noritsuna.gds`, `opamp_yamada3.gds`) the Dualgate layer (55/0) has 12, 18 and 12 shapes and V5_XTOR is present in two of them. In the same cells inside the top GDS both layers have zero shapes, while the Poly2 shape counts are unchanged. As fabricated these are therefore 3.3 V-oxide transistors at L = 0.5 to 0.7 um, whereas the upstream schematics and testbenches use `nfet_05v0_dn`/`nfet_06v0_dn`/`pfet_05v0`/`pfet_06v0` at VDD = 5 V. Applying 5 V would overstress them, and the existing simulations do not describe the silicon. `opamp_3zki`, the inverters, the comparator and the PLL keep their Dualgate shapes. This is read from layer counts, not from an LVS run.
- Other integration edits visible in the same comparison: tap cells (`TAP_PCONT_*`, `TAP_NCONT_*`) added to several blocks, and PFET groups in the ADC and PLL replaced by `P-FET_W2_8_*` and `PFET_SET` cells. The reason is not stated (DRC or latch-up fixes, inferred).
- `docs/pin_list` disagrees with the layout on the top edge. The list has two `dvss` rows (42 and 43) and seven ADC `bi_t` rows; the layout has one `dvss` and eight `bi_24t` pads there. Pins 43 to 52 are therefore off by one or one ADC pad is unlisted. The other three edges match cell-for-cell.
- All 15 `dvdd` pads sit on one IO ring built from stock `fill10` cells, so the separate "VDD" pins listed per circuit may be common through the ring rail. If so, 5 V and 3.3 V blocks cannot be powered at different voltages through the ring. Inferred from the cell list, not traced.
- The layout hash on the platform (270e32b2...) matches neither `gds/ISHI-KAI_WS_RUN1.zip` (436966...; re-committed 2026-01-04, after the last precheck) nor the precheck output GDS (70493e...). The two differ in size by about 9 kB.
- The project description says 14 designers; the member list has 14 entries but Gitefu appears twice, so 13 distinct people are named there, and the upstream ADC README adds supporters.
- Schematic links point at mutable branches of other repositories, one of them a personal fork. Nothing pins the versions that were taped out.
- The ADC schematic upstream (`SAR_TOP_LVS.sch`) instantiates the LDO; on this die the ADC and LDO are separate cells with separate pads.
- Affiliations for most members are not stated anywhere; several are known only by handle.
