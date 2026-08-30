# Panama Electricity Demand Forecasting

A project that predicts a full week of electricity demand for Panama's national grid, and checks the result against what the utility's own real forecasting process actually achieved — built to demonstrate Python and applied data science skills for a data scientist role at Black Hills Energy.

**Bottom line:** a machine learning model (XGBoost) averaged **3.90% error** across 14 test weeks spread over 2019–2020, beating the utility's own real, published forecast (**4.95% error**) on 13 of those 14 weeks.

## Why this project

I'm an accountant looking to move into data science, and previously applied for a data scientist role at Black Hills Energy - the feedback was a lack of demonstrated Python experience. This project is a direct answer to that: real utility data, a real forecasting problem, and a way of testing the results that's built to hold up under scrutiny rather than just produce a good-looking number.

The accounting background shows up less in the code itself and more in the habits around it: double-checking a calculated number against a trusted source before relying on it, being upfront about something that *didn't* work as well as hoped instead of only reporting the wins, and describing model accuracy in terms someone doing operational planning would actually use.

## The data

[Short-term electricity load forecasting (Panama)](https://www.kaggle.com/datasets/ernestojaguilar/shortterm-electricity-load-forecasting-panama) — Aguilar Madrid, 2021, via Kaggle/Mendeley.

- **48,048 hourly readings**, January 3, 2015 through June 27, 2020. Every hour is present — no gaps, no duplicates.
- **What we're predicting:** `nat_demand` — Panama's national electricity demand, in megawatt-hours (MWh).
- **Weather:** temperature, humidity, rainfall, and wind for three cities.
- **Calendar:** which days are holidays (with a name for each one) and which are school days vs. vacation.
- **The real benchmark:** the grid operator's actual weekly forecasts, published in advance — this is what makes the comparison meaningful, since it's not a made-up baseline.
- **Test weeks:** the dataset's own author designated 14 specific weeks for testing (explained more below), which this project reuses as-is rather than inventing a different split.

## What's in this project

| File | What it covers |
| `panama_load_eda.ipynb` | Getting to know the data: checking it's clean, spotting anything unusual, and understanding the patterns a model will need to learn |
| `panama_load_part2.ipynb` | Building the model inputs the right way, testing five different forecasting approaches, and comparing all of them to the utility's real forecast |

Both notebooks were fully run before being saved, so you can see every chart and result without needing to run anything yourself — though they're built to be rerun in Google Colab if you'd like to (see **How to Run** below).

## How this was built, and why it can be trusted

**A real anomaly, found and explained, not just deleted.** Early exploration turned up a sharp, single-day collapse in demand on January 20, 2019. That turned out to be real: a nationwide power outage was reported in Panama that day, three days before Pope Francis's visit for World Youth Day. Rather than deleting that data point or ignoring it, it's carried forward through the project as a simple marker so the model can recognize it as a one-off event.

**The model inputs are built to avoid a common, easy-to-miss mistake.** The goal is to predict a full week of demand at once, not one hour at a time. That rules out using "yesterday's demand" as an input — for later days in the forecast week, "yesterday" would actually fall inside the week we're trying to predict, meaning we wouldn't really know that number yet. Using it anyway would quietly let the model see part of the answer in advance (a mistake called **data leakage**), making it look more accurate than it really is. The fix: only use demand from 2, 3, and 4 weeks back, which is always safely in the past no matter which day of the forecast week you're looking at.

**Every number was tested the way it would really be used.** Rather than testing on just one convenient stretch of data, every model here was tested across all 14 weeks the dataset's original author set aside for exactly this purpose, spread across more than a year. Pulling the exact boundaries of those 14 weeks out of the author's own files (rather than guessing) revealed a consistent 73-hour gap between the end of each training period and the start of its test week — which turns out to be exactly the lead time a real weekly forecast needs, since the utility has to publish it a few days early. The testing here respects that same real-world constraint.

**A mistake was caught and corrected along the way.** An early version of the simple baseline model used "1 week ago" as an input, which — as explained above — isn't actually safe for a full week-ahead forecast. It was caught, fixed, and rerun; the more important model (XGBoost) barely changed as a result, which was itself a useful confirmation that the finding wasn't an accident of the mistake. Worth keeping in mind for a conversation about this project: **finding and fixing that kind of issue is the point, not something to hide.**

## Results

| Approach | Average error across 14 weeks |
| **XGBoost** | **3.90%** |
| Official utility forecast | 4.95% |
| Simple 3-week average | 5.35% |
| Simple 2-week lookup | 5.85% |
| Prophet (Meta's forecasting tool) | 6.31% |

*("Error" here means MAPE — on average, what percent off was the forecast from what actually happened.)*

Two honest results worth calling out rather than glossing over:

- **Prophet, a tool built specifically for this kind of forecasting, actually did the worst.** Its default assumptions about repeating patterns didn't fit this particular data as well as either a plain lookup or a model built from carefully chosen inputs. Not every tool wins on every dataset — and being able to say that plainly is worth more in an interview than pretending everything worked perfectly.
- **XGBoost's edge over the official forecast is biggest in May 2020** (5.66% vs. 9.06% error) — right in the middle of Panama's COVID-19 disruption to normal demand patterns. A model that gets rebuilt from scratch each week on recent data adjusted to that shift faster than the official process likely could.

## What this means in practice

A model isn't actually useful until someone can act on the number, not just admire it. A few ways this translates:

- **Average MWh off**, not just percent, is the number that matters operationally — it's closer to what a dispatcher would actually plan reserves around. (Both notebooks report this alongside the percentage.)
- **Forecasting too low** risks a real shortfall, covered by reserve power or short-notice imports — usually at a premium price.
- **Forecasting too high** means capacity held in reserve that didn't need to be — a real cost, just one that shows up as lost efficiency rather than an obvious line-item overrun.
- A forecast that's **more consistent** week to week, even at a similar average error, reduces how big a safety margin planners need to build in — which has value on its own, separate from the headline error number.

## How to run

1. Download the [Kaggle dataset](https://www.kaggle.com/datasets/ernestojaguilar/shortterm-electricity-load-forecasting-panama) (`continuous dataset.csv`, `weekly pre-dispatch forecast.csv`, `train_dataframes.xlsx`, `test_dataframes.xlsx`).
2. Open a notebook in [Google Colab](https://colab.research.google.com) (File → Upload notebook).
3. Upload the needed data files into the same session as the notebook.
4. Runtime → Run all. Part 2 refits several models across all 14 test weeks and takes a few minutes — that's expected, not a hang.

**What it needs installed:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `statsmodels`, `scikit-learn`, `prophet`, `xgboost` — all installable with `pip`.

## Potential imrpovements

- Confidence ranges instead of single-number forecasts, so a planner knows not just the expected demand but how much it could reasonably vary.
- Bringing in the weather data that wasn't used in the final model (humidity, rainfall, and two of the three weather stations were set aside early on for simplicity — worth revisiting).

## Credit

Dataset by Ernesto Aguilar Madrid (2021), [Short-term electricity load forecasting (Panama case study)](https://www.kaggle.com/datasets/ernestojaguilar/shortterm-electricity-load-forecasting-panama), Kaggle / Mendeley Data.
