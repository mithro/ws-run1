# MOSB: MOSbiusV3 (ws-run1)

| | |
|---|---|
| Repository | none listed on platform.wafer.space |
| Slot | 1x1 |
| Top cell | chip_top |
| Licence in repo | n/a |
| README | n/a |

## What the repo actually contains

No repository was listed for this project, so nothing was cloned and there is nothing to audit. The only first-hand information is the platform record in `project.json`. Platform description: empty.

The manufactured layout is in the public reticle (`ws-run1`, layout `MOSB/chip_top.gds`, SHA256 `e5f4ff37ef6aaa487ae2c23959f81b9207213b2ac4a470ba2da550cac0002498`), so the design can still be inspected at layout level even without sources.

## Authors and affiliations

Not stated in the manifest. The submitter's identity is held on platform.wafer.space.

## Value for GF180MCU characterisation and PDK development

### Verdict: Unknown (no sources)

## Notes and caveats

MOSbius is Peter Kinget's (Columbia University) educational analog chip platform, and it ran as a track of the 2025 IEEE SSCS Chipathon. The `mosbiuschip` GitHub organisation holds `mosbiuschip/chipathon2025` (track documentation) and `mosbiuschip/switch_matrix_gf180mcu_9t5v0` (a switch matrix for MOSbius on GF180MCU 9-track 5 V cells). These are plausible sources for a 'MOSbiusV3' die but nothing on the platform links them to this submission, and they have not been cloned or analysed. If this is a MOSbius chip it would be highly relevant for characterisation: MOSbius exposes individual transistors and analog building blocks to pins through a programmable switch matrix.
