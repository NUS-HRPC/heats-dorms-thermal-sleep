# HEATS — Profiling Dorms: Nighttime Indoor Environment and Sleep Data

Data supporting **"Nighttime Thermal Environments Are Associated with Impaired Sleep Among
Outdoor Workers in Non-Air-Conditioned Dormitories"** (HEATS — Cooling Dorms project). Data
were collected in two non-air-conditioned outdoor worker dormitories in Singapore (referred
to throughout as **Dormitory A** and **Dormitory B**). The study's broader design included a
second, automated-cooling arm; only data from the control arm (rooms without any automated
cooling intervention provided by the research team) is included here, as this is the only
condition discussed in the manuscript. Rows from the other arm — and the automated
fan-control event log columns associated with them — have already been removed from the
files in this repository.

The code combines (1) environmental sensor logs (temperature, humidity, CO2 etc.)
from fixed room monitors, (2) wrist-worn actigraphy sleep metrics, and (3) daily self-report
survey (Qualtrics) responses about thermal comfort, collected from the same participants on
the same nights so that indoor conditions can be linked to how people slept and perceived
their sleeping environment.

See **[DATA_DICTIONARY.md](DATA_DICTIONARY.md)** for a full description of every column in
every data file, **["How to run the analysis"](#how-to-run-the-analysis)** for reproducing the
manuscript's figures and tables, and **["How to cite this work"](#how-to-cite-this-work)**
for citation details.

## Files in data repository on Zenodo

| File | Contents |
|---|---|
| `dorm_A_qualtrics.csv` | Nightly self-report survey (Qualtrics) responses from Dormitory A control-arm participants — one row per participant-night, covering thermal sensation, thermal preference and air-movement preference reported before going to sleep. |
| `dorm_B_qualtrics.csv` | Same survey, same columns, for Dormitory B control-arm participants. |
| `env_dorm_A.parquet.gzip` | Minute-by-minute indoor/outdoor environmental sensor readings (temperature, humidity, CO₂, PM, VOC, etc.) from Atmocube monitors installed in Dormitory A's control-arm rooms. |
| `env_dorm_B.parquet.gzip` | Same sensor data, same collection method, for Dormitory B's control-arm rooms (which rooms count as control-arm changes partway through the study — see `DATA_DICTIONARY.md`). |
| `heats-dorms-sleepperiodmetrics.csv` | Per-night sleep metrics (time in bed, sleep onset latency, wake after sleep onset, sleep efficiency, etc.) derived from actigraphy watches. |
| `heats-dorms-manualreviewnights.xlsx` | A manual QC log: specific participant-nights flagged during visual review of the actigraphy data, with the reason they were flagged and which sleep metric was affected. |

There are six files here, joined two different ways:

- `dorm_A_qualtrics.csv`, `dorm_B_qualtrics.csv`, `heats-dorms-sleepperiodmetrics.csv` and
  `heats-dorms-manualreviewnights.xlsx` all share two common key columns — `Subject`
  (participant ID) and `Adjusted_date` (the study-night date) — and join to each other on
  `Subject` + `Adjusted_date`.
- `env_dorm_A.parquet.gzip` / `env_dorm_B.parquet.gzip` are keyed instead by a timestamp
  index and `id_room`, and link to the other four files by matching each sleep period's
  time window (`InBedTime` − 1 hour through `OutBedTime`) against the sensor timestamps for
  that participant's room.

### Participant and site naming

- `Subject` codes beginning `CDS0xx` are Dormitory A participants; codes beginning `CDS1xx`
  are Dormitory B participants.
- Room codes follow each dormitory's own numbering (e.g. `B205`, `B206`, `B207` in Dormitory
  A; `A104`, `A305`, `A201`, `B102`, `B302` in Dormitory B). In Dormitory B's sensor export,
  indoor and outdoor readings for the same room are distinguished with an ` Indoor` /
  ` Outdoor` suffix on `id_room`.
- Dormitory B was profiled in two date-bounded phases, with rooms swapped between the
  control arm and the study's other arm between phases: `A104`/`A305` were control-arm
  during phase 1 (27 Sep–18 Oct 2025), and `A201`/`B102`/`B302` were control-arm during
  phase 2 (25 Oct–15 Nov 2025). `env_dorm_B.parquet.gzip` only contains each room's rows
  from its own control-arm phase.

## How to run the analysis

The full analysis pipeline that consumes these files — from raw sensor/survey data through
the manuscript's figures and statistical tables — is in the companion Jupyter notebook,
`heats-dorms-thermal-sleep.ipynb` uploaded onto GitHub.

1. Clone the repository (or download the Zenodo archive and place its files under `data/`
   next to the notebook).
2. Install dependencies: `pip install -r requirements.txt`.
3. Run all cells in `heats-dorms-thermal-sleep.ipynb`. It reads only from `data/` and writes
   everything it generates (intermediate CSVs, figures, tables) to `outputs/`, so the source
   data is never modified by running the notebook.
4. Expected runtime: ~1min
5. Expected final analytic sample: **694 nights from 38 participants**.

Static image export (the commented-out 600 dpi TIFF / PNG `fig.write_image(...)` lines) needs
a Chrome/Chromium binary available to the `kaleido` package; if you uncomment one and hit a
"Kaleido requires Chrome" error, run `plotly_get_chrome` once or point kaleido at an existing
installation.

## Data collection summary

- **Environmental monitoring:** Atmocube fixed room monitors logged indoor and outdoor
  temperature, relative humidity, CO₂ and particulate matter at 1-minute intervals
  throughout each study period.
- **Sleep monitoring:** Participants wore an actigraphy watch nightly; sleep periods were
  scored using the Tudor-Locke algorithm to derive time in bed, total sleep time, sleep
  efficiency, sleep onset latency and wake after sleep onset.
- **Subjective surveys:** Participants completed a short Qualtrics survey before going to
  sleep each night, rating thermal sensation, thermal preference and air movement preference.

## How to cite this work

If you use this data, please cite the associated manuscript. The citation below is a
placeholder and **will be updated once the paper is formally published** (with a DOI/journal
link) and once the data is archived on Zenodo (10.5281/zenodo.22821941):

> Mani, R., Tan, S.C.C., Renard, M., Frei, M., Loke, J.M.Q., Leow, C.H.W., Tan, P.M.S.,
> Parkinson, T., Lo, J.C.Y., Schiavon, S., & Lee, J.K.W. (2026). *Nighttime Thermal
> Environments Are Associated with Impaired Sleep Among Outdoor Workers in Non-Air-Conditioned
> Dormitories.* [Manuscript in preparation]. HEATS — Cooling Dorms Project.
>
> Data: [10.5281/zenodo.22821941].

Please check back here for the final published citation and DOIs once available.
