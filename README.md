# Non-Human Biologics Data

Open research records from **xBioNexus**, the non-human biologics research program of Aerostratospheric, an Illinois nonprofit corporation. Records are published as the work happens. Flights are scheduled and research is underway; no results or findings are claimed.

- Research page: https://midwestsds.com/non-human-biologics-research.html
- Research findings: https://midwestsds.com/nhb-research-findings.html

This repository holds ground-based control data from Aerostratospheric for its non-human biologics research.

## Operation Canis Major

<img src="https://www.midwestsds.com/images/operation-canis-major-patch.png" alt="Operation Canis Major patch" width="120" align="right">

This project is part of [Operation Canis Major](https://www.midwestsds.com/operation-canis-major.html), often called Project Canis: Aerostratospheric's mission objective to provide open stratospheric and atmospheric research and data, unrestricted, to the public. Learn more: https://www.midwestsds.com/operation-canis-major.html

## Status

Ground-based control work with our non-human biologics has begun. No non-human biologics flights have taken place, and no results or findings are claimed.

Future research is planned. No results or outcomes are guaranteed.

## Study groups

Culture/lot 31585554 (*Hypsibius exemplaris* Z151), received Oct 28, 2026; groups set up Oct 30, 2026.

| Group | Role |
|---|---|
| [AERO-554-A](groups/AERO-554-A/) | Home tank: stock and backup population, never leaves the lab |
| [AERO-554-B1](groups/AERO-554-B1/) | Ground control dish, paired with C1 |
| [AERO-554-B2](groups/AERO-554-B2/) | Ground control dish, paired with C2 |
| [AERO-554-C1](groups/AERO-554-C1/) | Flight dish, paired with B1 |
| [AERO-554-C2](groups/AERO-554-C2/) | Flight dish, paired with B2 |

## Repository structure

```
records/receipt/   Culture Receipt & Inventory Record AERO-XBN-CRI-001 Rev. A (.md + .csv)
records/photos/    labeled AERO-554 groups photo (A home tank, B1/B2 ground control dishes, C1/C2 flight dishes; photo credit Maya Torres, xBioNexus)
groups/            one folder per group: README + observations.csv
docs/research/     blank forms and study documents (AERO-XBN-*)
docs/              tank logger setup notes and wiring
data/              tank logger CSV files
PROGRAM.md         research program
RESEARCH-REPORT.md ongoing research report
CITATION.cff       how to cite
```

The blank culture receipt form is [docs/research/culture-receipt-inventory-AERO-XBN-CRI-001.md](docs/research/culture-receipt-inventory-AERO-XBN-CRI-001.md). Photos and video clips are listed on the research findings page; image files will be added to this repository later.

## What is logged

An ESP32-based logger records ground-based control conditions every 10 minutes:

- Tank water temperature (°C and °F)
- Room air temperature (°C)
- Room relative humidity (%)
- Whether readings are within the target range

Data files live in [`data/`](data/). Logger wiring and setup notes live in [`docs/`](docs/).

### CSV columns (`data/tank_log.csv`)

| Column | Meaning |
|---|---|
| `timestamp_iso` | Reading time, ISO 8601 |
| `water_C` | Tank water temperature, °C |
| `water_F` | Tank water temperature, °F |
| `air_C` | Room air temperature, °C |
| `humidity_pct` | Room relative humidity, % |
| `in_range` | Whether the reading is within the target range |

## Credits

- Researcher: JH, Jeremy David Higgs (ORCID [0009-0008-0022-6241](https://orcid.org/0009-0008-0022-6241))
- AERO-554 groups photo: Maya Torres, xBioNexus
- Culture vendor: Carolina Biological Supply (vendor only, not a partner or sponsor)

## Using the data

The data is open to use with credit to Aerostratospheric. It is provided as-is, with no warranty of any kind. See [`LICENSE`](LICENSE) (license terms pending review).

These records document procedures and observations only and make no results, guarantees, or medical claims.
