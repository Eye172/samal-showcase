# SAMAL · engineering and evaluation

SAMAL combines a wind-power forecasting model with a repeatable agent workflow and an operator interface. This document retains the detailed data preparation, model comparison and historical replay notes behind the [project case study](../README.md).

## Reading the timeline

A **forecast issue** is one run produced at a specific time. A **target hour** is a future hour that the run predicts. Several issues can predict the same target hour as newer weather information arrives. SAMAL keeps those revisions separately, compares them on shared hours and records the weather run and model used for each issue.

**SCADA** is the turbine's recorded operating data. **NWP** means numerical weather prediction. The model learns the relationship between weather inputs and normalized turbine power; the agent coordinates the steps needed to produce and inspect a forecast.

## The difficult part is the timeline

A forecast can look excellent if it accidentally uses weather that was published later or measurements from the period it is meant to predict. SAMAL makes the issue time part of the data model. Every forecast needs to know which weather run was available and which SCADA observations were allowed at that moment.

The project assembled **142,360 and 149,499 SCADA observations** for the two turbines, at approximately ten-minute resolution, covering March 2023 through January 2026. Preparation removes duplicate records and builds hourly coverage before fitting and evaluation.

Archived ECMWF forecasts are retrieved through Open-Meteo Single Runs. Raw weather responses retain hashes and provenance. The replay uses an explicit **run time + 9 hours** availability assumption; that assumption is surfaced rather than treated as a measured publication timestamp.

## Inspect the forecast as an operator

The workspace selects issue date, issue time, horizon, turbine and display timezone. It shows hourly normalized power and summary values for the two halves of a 48-hour forecast. A weather update creates another forecast issue rather than overwriting the previous one.

![The desktop workspace keeps model output and execution context together.](../workspace.png)

*The desktop workspace keeps model output and execution context together.*

Comparing only overlapping target hours avoids a misleading comparison between two differently positioned horizons. The earlier issue remains visible, and CSV export provides a concrete handoff for further analysis.

## The agent orchestrates; the model calculates

~~~mermaid
flowchart LR
 A[Archived weather] --> B[Prepare aligned inputs]
 B --> C[Forecasting model]
 C --> D[Validate timeline and output]
 D --> E[Forecast ledger and CSV]
 F[Tool-using agent] -. orchestrates .-> A
 F -. orchestrates .-> B
 F -. orchestrates .-> C
 E --> G[Compare issues in the workspace]
~~~

The numerical power estimate comes from the forecasting model. The agent coordinates weather retrieval, preparation, forecasting, validation and export. A deterministic mode follows the same tool path without an LLM, making it possible to inspect the pipeline independently of model-provider availability.

![The trace records actions and provenance so a forecast can be inspected beyond its final curve.](../provenance.jpg)

*The trace records actions and provenance so a forecast can be inspected beyond its final curve.*

## Why the simpler model stayed

The evaluation compared learned empirical power curves with a CatBoost baseline. January backtesting covered **1,390 forecast-hour pairs per turbine**, representing **707 unique target hours**. Repeated horizons mean those two counts are not interchangeable.

| January evaluation | Turbine 1 | Turbine 2 |
|---|---:|---:|
| Selected model MAE | 0.190351 | 0.190407 |
| Selected model RMSE | 0.243279 | 0.243430 |
| CatBoost MAE | 0.253398 | 0.254650 |

The values are on the normalized-power scale, not megawatts. Evaluation weights used data before January 1; the later production refit through January 31 is a different model state and is not assigned the same backtest score.

Neural experiments explored MLP and transformer variants on a T4, with 60 checkpoints and 12 CatBoost comparisons in the documented research. Mean improvements were not sufficiently stable in uncertainty checks to justify replacing the selected version. Keeping the simpler model was an experimental conclusion, not an absence of further modelling work.

## Replay is not ground truth

The February replay generated **29 daily issues**, **2,784 ledger rows** and **1,344 selected forecast values**. Observed February power was unavailable in that replay, so these are completeness and pipeline results—not February accuracy metrics.

![Issue history and replay metrics expose what ran and which outputs were retained.](../history.jpg)

*Issue history and replay metrics expose what ran and which outputs were retained.*

<details><summary>Mobile workspace</summary>

![The mobile capture preserves the forecast controls and result inspection on a narrower display.](../mobile.png)

*The mobile capture preserves the forecast controls and result inspection on a narrower display.*

</details>

The implemented result is a forecast workflow whose inputs, issue times and revisions can be examined. Its most important constraint is explicit: all reported accuracy belongs to the documented January evaluation, while later replay demonstrates operational reproducibility.


## Data supplied for the challenge

| File | Data rows | First timestamp | Last timestamp |
|---|---:|---|---|
| Turbine 1 | 142,360 | 11 March 2023, 00:00 | 31 January 2026, 23:50 |
| Turbine 2 | 149,499 | 11 March 2023, 00:00 | 31 January 2026, 23:50 |

The participant CSV files contain an ID, timestamp, average wind speed (m/s), normalized active power and average ambient temperature (°C). These counts and boundary timestamps were checked against the supplied files. The table preserves the source timestamps without assigning an additional timezone.

[Original challenge and rubric](HACKATHON.md) · [Back to SAMAL](../README.md)
