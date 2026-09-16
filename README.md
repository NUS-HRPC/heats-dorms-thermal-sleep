# HEATS — Profiling Dorms: Nighttime Indoor Environment and Sleep Data

Data supporting **"Nighttime Indoor Environment and Sleep Outcomes Among Outdoor Workers in Non-Air-Conditioned Dormitories"** (HEATS — Cooling Dorms project). Data were collected in two non-air-conditioned outdoor worker CTQ-type dormitories in Singapore (referred to throughout as **Dormitory A** and **Dormitory B**). Only data from the study's control ("CON") arm — rooms without any automated cooling intervention provided by the research team — is included here, as this is the only condition discussed in the manuscript.

This repository combines (1) environmental sensor logs (temperature, humidity, CO2 etc.) from fixed room monitors, (2) wrist-worn actigraphy sleep metrics, and (3) daily self-report survey (Qualtrics) responses about thermal comfort, collected from the same participants on the same nights so that indoor conditions can be linked to how people slept and felt about their sleeping environment.

See **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)** for a full description of every column in
every file, and **["How to cite this work"](#how-to-cite-this-work)** below for citation
details.

## Files in this repository

| File | Contents |
|---|---|
| `dorm_A_qualtrics.csv` | Nightly self-report survey (Qualtrics) responses from Dormitory A participants — one row per participant-night, covering thermal sensation, thermal preference and air-movement preference reported before going to sleep. |
| `dorm_B_qualtrics.csv` | Same survey, same columns, for Dormitory B participants. |
| `env_dorm_A.parquet.gzip` | Minute-by-minute indoor/outdoor environmental sensor readings (temperature, humidity, CO₂, PM, VOC, etc.) from Atmocube monitors installed in Dormitory A rooms. |
| `env_dorm_B.parquet.gzip` | Same sensor data, same collection method, for Dormitory B. |
| `heats-dorms-sleepperiodmetrics.csv` | Per-night sleep metrics (time in bed, sleep onset latency, wake after sleep onset, sleep efficiency, etc.) derived from Ametris wrist actigraphy watches using the Tudor-Locke scoring algorithm. |
| `heats-dorms-manualreviewnights.xlsx` | A manual QC log: specific participant-nights flagged during visual review of the actigraphy data, with the reason they were flagged and which sleep metric was affected. |

All four data files share two common key columns — `Subject` (participant ID) and `Adjusted_date` (the study-night date) — so rows across files can be joined on
`Subject` + `Adjusted_date`. `env_dorm_A.parquet.gzip` / `env_dorm_B.parquet.gzip` are keyed instead by a timestamp index and `id_room`, and are linked to the other files by matching each sleep period's time window against the sensor timestamps for that participant's room (see the analysis notebook referenced below for the exact join logic).

### Participant and site naming

- `Subject` ID codes beginning `CDS0xx` are Dormitory A participants; codes beginning `CDS1xx`
  are Dormitory B participants.
- Room codes follow each dormitory's own numbering (e.g. `A202`, `A304` in Dormitory A;
  `A104`, `B102`, `B302` in Dormitory B). In Dormitory B's sensor export, indoor and outdoor
  readings for the same room are distinguished with an ` Indoor` / ` Outdoor` suffix on
  `id_room`.
- `mode` in the environmental files records which experimental arm a room was assigned to
  when a reading was logged. Only **`CON`** (control, no automated cooling) rows are used in
  the published analysis and described in this dataset's documentation; rows from other
  arms, and the automated fan-control event log columns associated with them, have been
  removed from these files as that condition is not discussed in the manuscript.

## Analysis code

The full analysis pipeline that consumes these files — from raw sensor/survey data through
the manuscript's figures and statistical tables — is in the companion Jupyter notebook,
`heats-dorms-thermal-sleep.ipynb`, in the parent project repository (not included in this
data-only repository). That notebook documents, cell by cell, exactly how each file here is
loaded, cleaned, merged and analyzed.

## Data collection summary

- **Environmental monitoring:** Atmocube fixed room monitors logged indoor and outdoor
  temperature, relative humidity, CO₂ and PM2.5 at 1-minute intervals
  throughout each study period.
- **Sleep monitoring:** Participants wore an actigraphy watch nightly; sleep
  periods were scored using the Tudor-Locke algorithm to derive time in bed, total sleep
  time, sleep efficiency, sleep onset latency and wake after sleep onset.
- **Subjective surveys:** Participants completed a short Qualtrics survey before going to
  sleep (and, in the full source survey, after waking — see `DATA_DICTIONARY.md` for why
  those after-sleep columns aren't in this repository's Qualtrics files) each night,
  rating thermal sensation, thermal preference and air-movement preference.

## How to cite this work

If you use this data, please cite the associated manuscript. The citation below is a
placeholder and **will be updated once the paper is formally published** (with a DOI/journal
link):

> Mani, R., et al. (2026). *Nighttime Indoor Environment and Sleep Outcomes Among Outdoor
> Workers in Non-Air-Conditioned Dormitories.* [Manuscript in preparation]. HEATS — Cooling
> Dorms Project.

Please check back here for the final published citation and link once available.
