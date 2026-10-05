# CHES: chess-move-generator (ws-run1)

| | |
|---|---|
| Repository | https://github.com/Ravenslofty/gf180mcu-chess @ b730465 (main, last commit 2025-12-11). **The design is not on main**: it is on branch `origin/chess` @ 728fecb ("first tapeout", 2025-11-15) |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 |
| README | TEMPLATE (100% the project-template README, no own text) |

## What the repo actually contains

The default branch `main` is the unmodified wafer.space project template. The chess design lives only on the `chess` branch, which is present in this clone as `origin/chess` but is not checked out (`git -C repo checkout chess` to see it).

On that branch: a chess move generator made of 64 square modules (`board.sv`, `square.sv`, `arb.sv`) in a combinational network, with JTAG boundary-scan-style register chains and a USB 2 device using an external ULPI PHY (`usb.v`, generated with Torii). Clock target is 60 MHz (16.6 ns) on `gf180mcu_fd_sc_mcu9t5v0` cells with `SYNTH_STRATEGY: DELAY 4` and one hold-fix ECO buffer (`dlyc_1`). The two SRAM macros of the template are not in the chess branch configuration.

The only final layout is the release asset `chip_top.gds.gz` (50 MB, in `../releases/`); its cell names confirm the 9t5v0 library. There are no reports.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Hannah Ravensloft ("Lofty", GitHub Ravenslofty) | sole designer | not stated | source headers, git log |
| Leo Moser (mole99) | project template | not stated | git log |

## Value for GF180MCU characterisation and PDK development

### Verdict: Low

Ordinary synthesised digital on stock cells. Its 60 MHz target is the tightest of the comparable run 1 digital designs and the logic is a large combinational network, so it may be a delay-sensitive data point, but nothing is built for measurement.

### Test structures and measurable circuits

Nothing found.

### Candidate PDK additions

Nothing found.

### What would be needed to use it

A board with a ULPI USB PHY and a host driver. No test plan in the repo.

## Notes and caveats

- The platform description says "eight-core chess move generator"; the sources on the `chess` branch show 64 square modules and nothing that corresponds to eight cores.
- Anyone looking at the repository's default branch sees only the template, with no indication the design is elsewhere.
