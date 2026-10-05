# RZML: LGN FashionMNIST - Logic Gate Network trained on FashionMNIST (ws-run1)

| | |
|---|---|
| Repository | https://gitlab.com/rejunity/ws0-lgn-fxnist-gf180mcu-tapeout @ ddbd922 (main, last commit 2025-12-20, "Chip 2nd iteration, PDK 1.6.6") |
| Slot | 1 x 1 |
| Top cell | `chip_top` |
| Licence in repo | none (no LICENSE file; audit: MISSING) |
| README | Unmodified GitLab boilerplate. The audit's "OK, 989 words" is wrong: nothing in it describes the project. |

## What the repo actually contains
Three commits (Initial commit 19f4175, "Checksum" d444854, "Chip 2nd iteration, PDK 1.6.6" ddbd922). This repo is a delivery container for the final layout only:
- `chip_top.gds`, 228 MB, via Git LFS (the real file is present).
- `chip_top.md5`, `.sha1`, `.sha256` and a Makefile target that generates them. sha256 `f86c8f2d...c316f371f0` matches the file. The platform hash (`cc0cee0d...d7c7b1`) does not match it; the platform file was probably post-processed (inferred).
- `fxnist.info`, two provenance pointers: the design commit https://github.com/rejunity/ws0-lgn-fxnist/commit/0a26078af0cae4c6be2e6b62dcaf1fc5d8d385c7 ("Merge remote-tracking branch 'wafer-space/main'") and the template commit gf180mcu-project-template@01d1540 ("update PDK (1.6.5)"; the repo's commit message says PDK 1.6.6).
- `.gitattributes` (LFS rule).

**Published here:** the final layout, its checksums and these pointers.
**Not published here:**
- RTL, the LibreLane config, the trained network and training scripts.
- Netlist, SDC, logs and reports.
- Testbench and licence.

**Where the source lives:** the README says nothing about this (it is the stock GitLab text). Only `fxnist.info` names it: GitHub `rejunity/ws0-lgn-fxnist`, which I confirmed exists, is public and is Apache-2.0 ("FashionMNIST LGN on Wafer.Space GF180 shuttle"). It holds the wafer.space template tree plus `src/{chip_core.sv, lgn.v, net.v, popcount.v}`, `pth_to_verilog.py`, `npz_to_verilog.py` and several trained networks (`.npz`/`.pth`/`.v`, with test accuracy and shape in the file names, e.g. `binTestAcc8873 ... 2x12000`). Observed in that repo, not this one:
- `chip_core.sv` takes 8 pixel inputs plus `write_enable` (`~input[8]`), outputs 16 bits on `bidir_out[15:0]`, and drives the remaining bidirs with `~rst_n`.
- `librelane/config.yaml` has `CLOCK_PERIOD: 1000` (1 MHz) and `PL_TARGET_DENSITY_PCT: 10`.
- `src/net.v` is 1,122,998 bytes, the same size as the 2x12000 network. I take that as the deployed network (about 88.7% binary test accuracy per its file name; inferred, not confirmed).

**What the GDS shows (via `strings`, approximate):**
- Standard cells: `gf180mcu_fd_sc_mcu7t5v0` (44 distinct cell types in use, about 83k filltie, about 10.5k buf_8, 7.6k nand2_1, 7.3k nor2_1, 7.1k xor2_1, 5.1k xor3_1, 3.6k dlyb_1).
- Pads: `gf180mcu_fd_io__bi_24t` (43 names), `in_c` (14), `in_s` (2), wafer.space `dvdd`/`dvss` (9/11 names).
- No SRAM macros.
- Net names such as `i_chip_core.lgn.calc_categories[0].sum_bits.y[...]` show a flat synthesised netlist.
- Very large amounts of fill (about 130k Metal4 fill cells).

## Authors and affiliations
| Person | Role | Affiliation | Source |
|---|---|---|---|
| Renaldas Zioma ("rej", GitHub rejunity) | design and tape-out (all commits) | GitHub bio: "Research"; no company named | git log, gh api users/rejunity |
| Leo Moser | wafer.space template named in fxnist.info | not stated | fxnist.info |

## Value for GF180MCU characterisation and PDK development
### Verdict: Low
A large synthesised gate array (flat logic-gate network) on stock 7t5v0 cells with 1 MHz timing intent. It was not built for characterisation, and no sources or reports are in this repo.

### Test structures and measurable circuits
Nothing purpose-built. The only measurable is functional: feed images through the 8-bit pixel interface and compare the 16 outputs with the Python model (the upstream repo has it). A clock sweep would give a fmax figure for a very deep, wide synthesised logic cone. Simulation results to compare against are not in this repo.

### Candidate PDK additions
Nothing found.

### What would be needed to use it
Packaged or bonded die. The upstream repo is needed for the net and the testbench. No pin map, test plan or measured results are in the GitLab repo.

## Notes and caveats
- The repo URL on the platform is GitLab; the sources are on GitHub; only `fxnist.info` links them.
- I did not read the upstream repo beyond its file list, `chip_core.sv`, the config and the README.
- Cell counts come from `strings` and include fill.
- The deployed network is inferred.
