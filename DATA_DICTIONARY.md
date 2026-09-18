# Data Dictionary

This dictionary describes every column in every file in this repository.

Two study sites are referenced throughout: **Dormitory A** (participant codes `CDS0xx`) and
**Dormitory B** (participant codes `CDS1xx`). Both are non-air-conditioned outdoor worker
dormitories in Singapore. The study's broader design included a second, automated-cooling
arm; every file in this repository has already been restricted to the control arm (the
condition without any automated cooling intervention) at the source, since that is the only
condition analyzed and reported in the manuscript.

---

## 1. `dorm_A_qualtrics.csv` / `dorm_B_qualtrics.csv`

One row = one participant's survey response on one study night. Both files share identical
columns; only the participants (and dates) differ.

These files are a subset of a longer daily Qualtrics survey (which also asked about the
*after-waking* thermal sensation/preference, sleep disruptions, adjustments made to fans/
windows/doors, etc.). Only the columns needed to identify each survey response and the
*before-sleep* thermal ratings used in the manuscript's Figure 3 are retained here.

| Column | Description | Data type | Valid values / format | Source |
|---|---|---|---|---|
| `Subject` | Unique participant identifier. | String | `CDS0xx` (Dormitory A) or `CDS1xx` (Dormitory B), e.g. `CDS001`, `CDS101`. | Assigned by the research team at enrollment. |
| `Night no.` | Sequential count of study nights for this participant, starting at 1 on their first night in the study. | Integer | 1, 2, 3, … (varies by participant). | Derived from participant enrollment date. |
| `day_of_week` | Day of the week the study night began. | String (categorical) | `Monday`–`Sunday`. | Derived from `Adjusted_date`. |
| `Adjusted_date` | The calendar date assigned to this study night. Because a night of sleep spans midnight, all data for a given sleep period (survey, actigraphy, environment) is tagged with the evening's date rather than the following morning's date, so that one date value uniquely identifies one night for one participant. **This is the key used to join the Qualtrics files with `heats-dorms-sleepperiodmetrics.csv` and `heats-dorms-manualreviewnights.xlsx`.** | Date | `YYYY-MM-DD`. | Derived by the research team from the night's actual survey/sleep timestamps. |
| `night_survey_timestamp` | Date and time the participant submitted the pre-sleep ("going to sleep") survey. | Datetime | `YYYY-MM-DD HH:MM:SS`, local (Singapore) time. | Qualtrics submission timestamp. |
| `morning_survey_timestamp` | Date and time the participant submitted the post-wake ("just waking up") survey. | Datetime | `YYYY-MM-DD HH:MM:SS`, local (Singapore) time. | Qualtrics submission timestamp. |
| `Q0.2 BEF / AFT` | Which of the two daily surveys this row's *timestamp/date* metadata was captured from. Included for transparency about survey structure; the thermal-rating columns below (`Q1.3`, `Q1.4`, `Q1.5`) are always the **pre-sleep** ratings regardless of this flag. | String (categorical) | `going to sleep` (pre-sleep survey) or `just waking up` (post-wake survey). | Qualtrics embedded/branching-logic field identifying which survey flow the response came from. |
| `Q1.3 PRE-RTS` | **Retrospective Thermal Sensation (RTS)** reported just before going to sleep: how hot or cold the participant felt at that moment. A 7-point scale adapted from the ASHRAE thermal sensation scale used widely in thermal-comfort research. | String (ordinal categorical) | One of: `Cold`, `Cool`, `Slightly cool`, `Neutral`, `Slightly warm`, `Warm`, `Hot` (ordered cold → hot). | Qualtrics survey, pre-sleep block, question 1.3 ("How do you feel right now?" thermal sensation item). |
| `Q1.4 RTP` | **Retrospective Thermal Preference (RTP)** reported just before going to sleep: whether the participant would prefer the room to be different than it currently is. | String (categorical) | `Cooler`, `Without change`, `Warmer`. | Qualtrics survey, pre-sleep block, question 1.4 ("Would you prefer the room to be...?"). |
| `Q1.5 Wind` | **Air-movement preference** reported just before going to sleep: whether the participant would prefer more or less air movement (e.g. from a fan or window) in the room. | String (categorical) | `More wind`, `No change`, `Less wind`. (In the manuscript/analysis these are relabelled "More air movement" / "No change" / "Less air movement" for clarity.) | Qualtrics survey, pre-sleep block, question 1.5 ("Would you prefer more or less air movement?"). |

**Row counts:** Dormitory A — 304 control-arm participant-nights from 11 participants;
Dormitory B — 539 control-arm participant-nights from 29 participants (a 30th control-arm
participant, CDS110, has no Qualtrics rows at all in the raw export). These counts include
nights later excluded from the final analytic sample during QC — see
`heats-dorms-manualreviewnights.xlsx` below.

---

## 2. `heats-dorms-sleepperiodmetrics.csv`

One row = one participant's sleep period on one study night, as scored from their wrist
actigraphy watch.

Unlike the other files in this repository, this file has **not** been pre-filtered to the
control arm — the analysis notebook restricts it to the 41 control-arm participants (via a
room → participant lookup) as its first step in Section 7.1.

| Column | Description | Data type | Valid values / format | Source |
|---|---|---|---|---|
| `Subject` | Participant identifier. Joins to the Qualtrics files and manual-review log. | String | `CDS0xx` / `CDS1xx`. | Study enrolment ID. |
| `Site` | Which dormitory the participant belongs to. | String (categorical) | `Site_A`, `Site_B`. | Assigned by research team. |
| `Room` | Dormitory room the participant slept in that night. | String | e.g. `A202`, `B302`. Blank if not recorded. | Study logistics record. |
| `Adjusted_date` | Study-night date (see definition under file 1 above). Joins to the Qualtrics and manual-review files. | Date | `YYYY-MM-DD`. | Derived from the night's sleep-period timestamps. |
| `Night_WearTime_min` | Total minutes the actigraphy watch was worn during the recording window for this night. | Float (minutes) | ≥ 0. | Computed from the watch's raw movement log. |
| `WearTime_Criteria_5h` | Whether the night meets the minimum 5-hour wear-time criterion used to judge whether a night's actigraphy data is usable. | Boolean | `True`, `False`, or blank (not evaluated). | Derived: `True` if `Night_WearTime_min` ≥ 300. |
| `SleepDataLogged` | Intended to flag whether the actigraphy algorithm's output for this night was the one confirmed/logged for analysis. In practice, **every** row in this file — both `True` and `False` — carries a full set of computed sleep metrics; `False` does not mean "no sleep period was detected." The analysis notebook treats `True` as the row to keep (909 of 1,834 rows) and drops the rest. If you need to know exactly what a `False` row represents (e.g. an alternate candidate period from the same night, or an earlier/un-merged pass of the scoring algorithm), check with whoever ran the actigraphy scoring pipeline — it isn't self-evident from this file alone. | Boolean | `True` (909 rows), `False` (925 rows). | Output of the sleep-scoring algorithm / manual confirmation step. |
| `InBedTime` | Timestamp the participant got into bed (start of the scored sleep period). | Datetime | `D/M/YY H:MM` (day and month not zero-padded), e.g. `1/5/25 22:21`, `10/5/25 23:41`. | Actigraphy algorithm / manual event-marker log, whichever the pipeline judged more reliable. |
| `OutBedTime` | Timestamp the participant got out of bed (end of the scored sleep period). | Datetime | `D/M/YY H:MM`, same format as `InBedTime`. | Actigraphy algorithm / manual event-marker log. |
| `Onset` | Timestamp of **sleep onset** — the point the algorithm determined the participant actually fell asleep (after `InBedTime`, once movement drops enough to be classified as sleep). | Datetime | `D/M/YY H:MM`, same format as `InBedTime`. | Tudor-Locke sleep-scoring algorithm. |
| `LatencyInMinutes` | **Sleep onset latency (SOL)**: minutes between `InBedTime` and `Onset` — how long it took the participant to fall asleep after getting into bed. | Integer (minutes) | ≥ 0. | Computed: `Onset − InBedTime`. |
| `AvgAwakeningInMinutes` | Average duration, in minutes, of each awakening episode during the sleep period. | Float (minutes) | ≥ 0. | Tudor-Locke algorithm output. |
| `AwakeningCount` | Number of distinct awakening episodes detected during the sleep period. | Integer | ≥ 0. | Tudor-Locke algorithm output. |
| `Efficiency` | **Sleep efficiency (SE)**: the fraction of time in bed that was actually spent asleep. Higher values indicate more consolidated sleep. Stored here as a **0–1 fraction, not a percentage** — the analysis notebook multiplies by 100 to report it as a percent (see also `SE_recalculated`, computed independently downstream from `TimeAsleepInMinutes`/`TIB_recalculated`, which is the version actually used in the manuscript's figures/tables). | Float (fraction) | 0.0–1.0. | Actigraphy algorithm output. |
| `TimeAsleepInMinutes` | **Total sleep time (TST)**: total minutes classified as asleep between `Onset` and `OutBedTime`. | Integer (minutes) | ≥ 0. | Tudor-Locke algorithm output. |
| `TimeAwakeInMinutes` | Total minutes classified as awake between `Onset` and `OutBedTime` (i.e. time spent in bed, asleep-attempt period, but awake). | Integer (minutes) | ≥ 0. | Tudor-Locke algorithm output. |
| `WakeAfterOnsetInMinutes` | **Wake after sleep onset (WASO)**: total minutes spent awake after initially falling asleep and before getting up. Distinct from `LatencyInMinutes`, which is time *before* falling asleep. | Integer (minutes) | ≥ 0. | Tudor-Locke algorithm output. |
| `TotalCounts` | Sum of the raw wrist-movement "activity counts" recorded by the watch over the sleep period — a measure of how much the participant moved overall. Higher values indicate more restless sleep. | Integer | ≥ 0. | Raw actigraphy accelerometer output, summed by the watch's proprietary algorithm. |
| `TotalInBedTime_minutes` | Total duration from `InBedTime` to `OutBedTime`, in minutes (i.e. `LatencyInMinutes + TimeAsleepInMinutes + TimeAwakeInMinutes`). | Integer (minutes) | ≥ 0. | Computed: `OutBedTime − InBedTime`. |
| `TotalInBedTime_hours` | Same duration as `TotalInBedTime_minutes`, expressed in hours for readability. | Float (hours) | ≥ 0. | Computed: `TotalInBedTime_minutes / 60`. |

**Row counts:** 1,834 rows total; 909 with `SleepDataLogged == True`. Twenty of the `True`
rows are exact duplicates on `(Subject, Adjusted_date)` — ten pairs, all dated 30 Sep 2025
(Dormitory B, phase 1), differing only in `Night_WearTime_min` — and are deduplicated in the
analysis notebook by keeping whichever row appears first in the file (an accepted, fixed
tiebreak rather than a wear-time-based one).

---

## 3. `heats-dorms-manualreviewnights.xlsx`

A quality-control log. Not every night's actigraphy data is reliable — e.g. a participant
might forget to wear the watch, or press an event marker without actually going to sleep.
The research team visually reviewed the raw actigraphy actograms and logged any night with a
suspected data-quality issue here, so that it can be down-weighted or excluded during
analysis.

| Column | Description | Data type | Valid values / format | Source |
|---|---|---|---|---|
| `Subject` | Participant identifier. Joins to the other files on `Subject` + `Adjusted_date`. | String | `CDS0xx` / `CDS1xx`. | Study enrollment ID. |
| `Adjusted_date` | Study-night date being flagged. | Date | `YYYY-MM-DD`. | Study logs. |
| `Reason` | Free-text explanation of why this night was flagged during manual review. | String (free text) | e.g. `"Watch not worn"`, `"Inaccurate InBedTime"`, `"Many mvmt spikes during manual sleep"`. See file for the full list of ~10 recurring reasons used. Blank for a few rows. | Research staff notes, written during visual QC review of the actigraphy trace. |
| `Removed` | Whether this night was actually removed from the analytic sample as a result of the flag. | — | Currently empty for all rows in this file (no nights in this log were marked as removed here — removal decisions are applied downstream in the analysis pipeline, not recorded in this column). | Research staff / analysis pipeline. |
| `Problem` | Short code for which specific sleep metric the flagged issue is believed to affect. | String (categorical) | `SE` (sleep efficiency), `SOL` (sleep onset latency), `high WASO` (wake after sleep onset), `100% SE`, `High SOL`. | Research staff, assigned during manual review. |

**Row counts:** 97 rows, covering 94 unique `(Subject, Adjusted_date)` nights. Three nights
(`CDS108`/2025-11-01, `CDS130`/2025-11-08, `CDS130`/2025-11-09) each appear twice with
different `Reason`/`Problem` text — both entries describe the same flagged night from a
different angle and neither is more authoritative than the other.

---

## 4. `env_dorm_A.parquet.gzip` / `env_dorm_B.parquet.gzip`

Minute-by-minute environmental sensor logs from fixed **Atmocube** room monitors (indoor
and outdoor units). One row = one sensor reading at one timestamp, from one device. Both
files share the same columns, with one exception (`co2_corrected`, Dormitory B only) noted
in its row below.

**Format note:** this is a `.parquet.gzip` file (a compressed, columnar binary data format,
not plain text) — open it with a data analysis tool such as `pandas.read_parquet()` in
Python or `arrow::read_parquet()` in R, not a text editor.

**Index (row timestamp):** Both files are indexed by a timezone-aware timestamp (`Asia/
Singapore`, UTC+8) at 1-minute resolution — this is the primary time reference for every
reading and does not have its own column name (it is the DataFrame index when loaded with
pandas).

| Column | Description | Data type | Valid values / units | Source |
|---|---|---|---|---|
| `temperature` | Raw (uncalibrated) air temperature at the sensor. | Float | °C. | Atmocube monitor, onboard temperature sensor. |
| `humidity` | Raw (uncalibrated) relative humidity at the sensor. | Float | % RH, 0–100. | Atmocube monitor, onboard humidity sensor. |
| `temperature_calibrated` | Temperature after applying a lab calibration correction. **This is the temperature value used throughout the manuscript's analysis**, not the raw `temperature` column. | Float | °C. | Derived: raw `temperature` + device-specific calibration offset (established by co-locating devices before deployment). |
| `humidity_calibrated` | Relative humidity after applying a lab calibration correction. **Used throughout the analysis** in place of raw `humidity`. | Float | % RH, 0–100. | Derived, same calibration procedure as `temperature_calibrated`. |
| `abs_humidity` | On-device estimate of absolute humidity (mass of water vapor per unit volume of air). **Not used in any analysis** — the manuscript instead derives its own absolute humidity from `temperature_calibrated` and `humidity_calibrated` (see the analysis notebook, `abs_humidity_c`, which is not stored in this raw file). Retained here only because it ships natively in the Atmocube export. | Float | g/m³ (approx.). | Atmocube monitor's onboard firmware calculation. |
| `co2` | Carbon dioxide concentration. Elevated indoor CO₂ typically reflects poor ventilation / occupant density. | Float | ppm (parts per million). | Atmocube monitor, onboard NDIR CO₂ sensor. |
| `co2_corrected` | *(Dormitory B file only.)* CO₂ concentration after a sensor drift/calibration correction, parallel to `temperature_calibrated`/`humidity_calibrated`. | Float | ppm. | Derived correction applied to raw `co2`. |
| `pm1` | Particulate matter concentration, particles ≤ 1 µm in diameter. | Float | µg/m³. | Atmocube monitor, onboard laser particle sensor. |
| `pm25` | Particulate matter concentration, particles ≤ 2.5 µm in diameter (PM₂.₅ — the size fraction most linked to respiratory health effects). | Float | µg/m³. | Atmocube monitor, onboard laser particle sensor. |
| `pm4` | Particulate matter concentration, particles ≤ 4 µm in diameter. | Float | µg/m³. | Atmocube monitor, onboard laser particle sensor. |
| `pm10` | Particulate matter concentration, particles ≤ 10 µm in diameter (PM₁₀). | Float | µg/m³. | Atmocube monitor, onboard laser particle sensor. |
| `voc` | Total volatile organic compound concentration. | Float | ppm (approx.). | Atmocube monitor, onboard VOC gas sensor. |
| `vocindex` | A normalized VOC index derived from the sensor's raw VOC signal (Sensirion-style VOC Index convention), easier to interpret across devices than raw `voc`. | Float | 1–500 (100 ≈ typical/baseline conditions; higher = more VOCs than baseline). | Onboard sensor firmware algorithm. |
| `noxindex` | A normalized index for oxides of nitrogen (NOx), analogous in construction to `vocindex`. | Float | Index scale, observed range ~1–9 in this dataset. | Onboard sensor firmware algorithm. |
| `ch2o` | Formaldehyde concentration. | Float | ppm (approx.). | Atmocube monitor, onboard formaldehyde sensor. |
| `iaqi` | Indoor Air Quality Index — a single composite score summarizing overall air quality from the sensor's multiple pollutant readings. | Float | 0–100 (higher = better air quality), per Atmocube's proprietary scale. | Onboard sensor firmware algorithm, combining VOC/CO₂/PM readings. |
| `eiaqi` | An "enhanced"/extended variant of the IAQI composite score (exact formula is proprietary to the device manufacturer). | Float | Observed range 0–60 in this dataset. | Onboard sensor firmware algorithm. |
| `tci` | Thermal Comfort Index — a composite score summarizing thermal comfort conditions from temperature/humidity readings (exact formula is proprietary to the device manufacturer; not the same as any thermal-comfort index computed in the manuscript's own analysis). | Float | Observed range 0–60 in this dataset. | Onboard sensor firmware algorithm. |
| `light` | Ambient light level at the sensor. | Float | Lux. | Atmocube monitor, onboard light sensor. |
| `noise` | Ambient sound level at the sensor. | Float | dB(A) (approx.). | Atmocube monitor, onboard microphone/noise sensor. |
| `pressure` | Barometric (atmospheric) pressure at the sensor. | Float | Pa (pascals). | Atmocube monitor, onboard barometric sensor. |
| `device_name` | Human-assigned label for the physical device, used by the research team for inventory tracking. Retained as a stable, human-readable sensor identifier; the other device/session metadata fields Atmocube's export ships (`device_id`, `device_serial_number`, `id`, `id_participant`, `ip`, `signal`, `timestamp_lambda_ms`, `avti`) have been dropped from the shared files as internal identifiers with no analytical value. | String | e.g. `atmo_28`, `atmo_35`. | Research team's device inventory naming. |
| `product_id` | Product/model identifier of the sensor hardware. Constant across all rows. | String | `ATCUBEBF1` (Atmocube monitor model code). | Device manufacturer. |
| `id_room` | Room (and indoor/outdoor placement) the sensor was installed in. | String | `B205`, `B206`, `B207`, `Outside 1` (Dormitory A file); `A104 Indoor`/`Outdoor`, `A305 Indoor`/`Outdoor`, `A201 Indoor`/`Outdoor`, `B102 Indoor`/`Outdoor`, `B302 Indoor`/`Outdoor` (Dormitory B file). | Study installation log. |

**Row counts:** Dormitory A — 203,685 rows across the 4 control-arm sensors (`B205`, `B206`,
`B207`, `Outside 1`), 26 April–31 May 2025. Dormitory B — 267,504 rows across the 5
control-arm room pairs, split across two date-bounded phases (rooms were swapped between the
control arm and the study's other arm between phases): `A104`/`A305` Indoor+Outdoor,
27 Sep–18 Oct 2025 (phase 1); `A201`/`B102`/`B302` Indoor+Outdoor, 25 Oct–15 Nov 2025
(phase 2).

---

## General notes

- **Missing/blank values:** across all files, a blank cell means the value was not
  collected, not applicable, or not yet computed for that row — it does not necessarily
  indicate an error.
- **Timezone:** all timestamps in this dataset are in Singapore local time (UTC+8), the
  location of both study dormitories.
