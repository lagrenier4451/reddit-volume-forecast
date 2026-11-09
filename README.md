# Reddit Volume Forecast

Can pre-market chatter on r/wallstreetbets help a brokerage flag which stocks will trade unusually heavily that day?

**Team:** Luc Grenier, Alex Toth
**Course:** INFO 4360 Complex Data Analytics, University of Denver
**Status:** Complete (Phase III)

## Problem

A retail brokerage's risk and support teams often notice Reddit-driven trading surges only after they start. We test whether Reddit attention and sentiment in comments posted before the 9:30 a.m. open help predict which tickers will have a volume spike that session, beyond what yesterday's volume and earnings news already show. The intended output is a morning watchlist of the three tickers most likely to spike.

## Main results

All models were evaluated the way they would be used: trained on earlier sessions and tested one morning at a time on Aug 18–31, 2021 (10 mornings, 170 ticker-sessions, 26 spikes). The metrics and feature sets were fixed before any model was run.

| Model | Features | Precision@3 | Avg. precision |
|---|---|---|---|
| Random | — | 15.3% | 0.153 |
| M1 | Yesterday's volume | 60% | 0.730 |
| M2 | + Earnings flag | 60% | 0.741 |
| M3 | + Reddit attention (mention rate, velocity) | 60% | 0.750 |
| M4 | + VADER sentiment | 60% | 0.726 |
| M5 | + Claude stance and intensity | 60% | 0.736 |
| Best possible | — | 73.3% | 1.000 |

- **Yesterday's volume does most of the work.** It alone gets 60% of watchlist picks right, against a 73% ceiling.
- **Reddit attention adds a little.** Average precision rises from 0.741 to 0.750, with no gain on the watchlist. (F1 at a 0.5 threshold, a secondary measure, rises from 0.56 to 0.64.)
- **Sentiment adds nothing.** Neither VADER nor Claude labels improved results.
- **The watchlist is concentrated.** CRWD and SPY take 14 of 30 picks; the models catch all large-cap spikes but only 6 of 12 meme-stock spikes.

Full analysis, key findings, business implications, limitations and the AI usage log are in the Phase III notebook.

## Data

| Item | Description |
|---|---|
| Text | 1,001,160 r/wallstreetbets comments from August 2021 ([SocialGrep on Hugging Face](https://huggingface.co/datasets/SocialGrep/reddit-wallstreetbets-aug-2021)); 813,935 usable after removing deleted and removed comments. Loaded in the notebook from a pinned URL; nothing to download manually. |
| Volume | Daily trading volume from Yahoo Finance via `yfinance` (July–August 2021) |
| Unit of analysis | One row per ticker per trading session: 17 tickers × 22 sessions = 374 rows; 306 in the modelling sample |
| Target | Volume at least 1.5× the ticker's average over the previous 20 trading days (50 spikes) |
| Session rule | Each comment counts toward the next 9:30 a.m. New York open after it was posted |
| Claude labels | Saved in `notebooks/labeling_results/` (53,842 comment–ticker pairs), so reruns need no API calls |

## API usage

Claude Haiku 5.5 labelled about 54,000 comment–ticker pairs, mostly through the Batch API: about 86.6M input and 18.0M output tokens, **about $8.86 in total** ($8.81 for the batch run). Details are in the notebook's API usage table.

## How to run

1. Python 3.11 or later. From the project folder:
   ```
   pip install -r requirements.txt
   ```
2. Open `notebooks/CDA_PhaseIII_AlexToth_LucGrenier.ipynb` and choose **Restart** then **Run All**. It needs an internet connection (Hugging Face and Yahoo Finance) and takes a few minutes to load the comments.
3. **No API key or spending is needed to reproduce the results.** All Claude labels load from `notebooks/labeling_results/`, and `ALLOW_API_CALLS = False` blocks any new paid calls.
4. Only to make new Claude calls: put `ANTHROPIC_API_KEY=your-key` in a `.env` file in the `notebooks/` folder and set `ALLOW_API_CALLS = True`. `.env` is gitignored; never commit a key.

HTML exports of the notebooks are in `notebooks/` for reading without running anything.

## Repository structure

```
notebooks/
  CDA_PhaseIII_AlexToth_LucGrenier.ipynb   # final report (Phase III)
  CDA_PhaseIII_AlexToth_LucGrenier.html    # HTML export
  CDA_PhaseII_AlexToth_LucGrenier.ipynb    # Phase II exploration
  labeling_results/                        # saved Claude labels, verification sheet, batch IDs
  01b_volume_target.ipynb                  # earlier exploration on 2021 Kaggle posts (superseded)
data/raw/                                  # Kaggle posts used only by 01b
requirements.txt
LICENSE                                    # MIT (applies to our code, not the datasets)
```

## Limitations

One month of data from the 2021 meme-stock period; 10 test mornings; one earnings session in the test period; ticker matching misses lowercase mentions and company names. Results are exploratory and make no claim of statistical significance. A volume-spike flag says nothing about price direction and is meant to prompt human review, not trading.

## License

Code is released under the MIT License. The datasets remain subject to their original sources' terms.
