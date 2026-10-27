# Reddit Volume Forecast

Can Reddit discussion help a retail brokerage anticipate which stocks will see unusually high trading volume, before the market opens?

**Authors:** Luc Grenier, Alex Toth
**Course:** INFO 4360 Complex Data Analytics, University of Denver
**Status:** Phase 2 (exploration) complete; Phase 3 (modelling) next

## Problem

A brokerage's risk and market-intelligence teams often notice Reddit-driven attention only after trading and support requests have already surged. We test whether ticker mentions and sentiment in r/wallstreetbets comments posted before the open can help flag tickers likely to have a volume spike that session, producing a pre-market watchlist.

## Data

| Item | Description |
|---|---|
| Text | 1,001,160 r/wallstreetbets comments from August 2021 ([SocialGrep on Hugging Face](https://huggingface.co/datasets/SocialGrep/reddit-wallstreetbets-aug-2021)); 813,935 usable after removing deleted/removed comments. Loaded directly from a pinned URL in the notebook, nothing to download manually. |
| Volume | Daily trading volume from Yahoo Finance via `yfinance` (July–August 2021) |
| Unit of analysis | One row per ticker-session: 17 tickers × 22 trading sessions = 374 rows |
| Target | `spike_1.5`: session volume at least 1.5× the ticker's average over the previous 20 trading days (50 positive cases) |
| Session rule | Each comment counts toward the next 9:30 a.m. New York market open after it was posted |
| Ticker matching | Uppercase symbols (e.g. `GME`) or `$`-prefixed symbols in any case (e.g. `$gme`) |

HOOD is excluded (listed July 29, 2021, so no 20-day baseline). WISH volume is retrieved under its current symbol, LOGC.

An earlier exploration of a Kaggle dataset of WSB *posts* (January–April 2021) is kept in `notebooks/01b_volume_target.ipynb` with its data in `data/raw/`. We moved to comments because posts were too thin (median 18 words, more than half title-only) and dominated by GME.

## Repository structure

```
notebooks/
  CDA_PhaseII_AlexToth_LucGrenier.ipynb   # Phase 2 submission: exploration, text mining, Phase 3 plan
  01b_volume_target.ipynb                 # earlier exploration on 2021 posts (superseded)
  CDA_Project_PhaseII_Requirements.ipynb  # assignment brief
data/
  raw/reddit_wsb.csv                      # Kaggle posts used by 01b only
  raw/RawDataREADME.txt                   # source notes for that file
requirements.txt
LICENSE                                   # MIT (applies to our code, not the datasets)
```

## How to run

1. Python 3.11 or later. Install dependencies:
   ```
   pip install -r requirements.txt
   ```
2. **Optional, for the Claude pilot in Section 3 only:** create a file named `.env` in the project root containing
   ```
   ANTHROPIC_API_KEY=your-key-here
   ```
   `.env` is gitignored. Never commit a key.
3. Open `notebooks/CDA_PhaseII_AlexToth_LucGrenier.ipynb` and choose **Restart & Run All**. It needs an internet connection (Hugging Face and Yahoo Finance) and takes a few minutes, mostly loading the comments.

## Phase 2 findings (summary)

- **Attention rises before spikes.** Pre-open mentions before spike sessions run at about 2.3× each ticker's typical level, versus about 1.0× before normal sessions.
- **VADER sentiment does not.** Mean sentiment and intensity are nearly identical before spike and normal sessions, and VADER's lexicon is missing 18 of 20 common WSB trading terms (including "calls," "puts," "earnings," and "short").
- **Earnings matter.** "Earnings" appears almost twice as often before spikes, and the tickers that spiked most reported earnings in August, so Phase 3 includes an earnings control.

## Phase 3 plan (summary)

Classification (Path A) with logistic regression and a gradient-boosting comparison, using a small feature set: volume momentum, mention velocity, one sentiment feature, one emotion-intensity feature, and an earnings flag. Claude labels comment-level stance, emotion, intensity, and catalyst, and we compare VADER, Claude polarity, and Claude emotion as features against a volume-only baseline. Evaluation uses time-ordered (expanding-window) validation and a morning watchlist check. Details are in Section 3 of the notebook.

## License

Code is released under the MIT License. The datasets remain subject to their original sources' terms.
