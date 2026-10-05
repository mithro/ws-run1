# KIAN: KianV: A 32-bit RISC-V Linux SoC taped out on GF180MCU (ws-run1)

| | |
|---|---|
| Repository | https://github.com/splinedrive/gf180mcu-kianv-rv32ima-sv32 @ 2483ecb (main, last commit 2026-01-15) |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | Apache-2.0 |
| README | THIN (original but only 59 words) |

## What the repo actually contains

The KianV RV32IMA SoC with SV32 MMU and Zicntr/Zicsr/Zifencei/SSTC, able to run Linux and XV6. It has instruction and data caches, an SDRAM controller, SPI, UART and GPIO.

Built with the wafer.space template on `gf180mcu_fd_sc_mcu9t5v0` standard cells with 21 foundry `gf180mcu_fd_ip_sram__sram512x8m8wm1` macros (5 V) and `gf180mcu_fd_io` pads (54 bidir, 8 DVDD, 10 DVSS). Clock target is 33 ns (30 MHz), with STA across 15 corners labelled 3v30/3v00/3v60 on the 5 V library cells.

The final GDS is in the repo as a zip (`gds/chip_top.gds.zip`, 32 MB). There are no timing or DRC reports. An `fpga/` directory holds a ULX3S (ECP5) build. Commit messages mention a GF180 SRAM CEN protocol fix, hold-margin changes, a Metal3 density tweak for DRC rule M3.2b (30.1%), and a bonding fix.

## Authors and affiliations

| Person | Role | Affiliation | Source |
|---|---|---|---|
| Hirosh Dabui (GitHub splinedrive) | CPU and SoC | GitHub bio: "M.Sc.CE., Technical University of Berlin", Berlin | GitHub profile, matched by commit identity |
| Leo Moser (mole99) | project template | not stated | git log |

## Value for GF180MCU characterisation and PDK development

### Verdict: Low

Ordinary synthesised digital on stock cells and foundry SRAM; a functional and yield data point. The only silicon-checkable timing is the 33 ns target.

### Test structures and measurable circuits

Nothing purpose-built. 21 foundry SRAM macros are exercised functionally as caches.

### Candidate PDK additions

Nothing found.

### What would be needed to use it

A board with SDRAM and SPI flash to boot Linux; none is described in the repo.

## Notes and caveats

- The run 2 chip (`../../ws-run2/KIAN`) is the same RTL on the 3.3 V library, so measuring both gives an Fmax and power comparison between the 5 V library run at 3.3 V and the native 3.3 V library.
- The README says the CPU sources were added afterwards and points to github.com/splinedrive/kianv-rv32-linuxcore.
