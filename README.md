# Reddit Volume Forecast

Predicts which watched stock tickers are likely to experience abnormal next-day trading volume, so a retail brokerage can prepare before markets open.

**Authors:** Luc Grenier, Alex Toth
**Course:** INFO 4360 — Complex Data Analytics, University of Denver

## Problem

A retail brokerage's market intelligence and risk teams need advance notice of stocks likely to experience unusually high trading volume. Today, they may only recognize that a stock has gained attention on Reddit after trading activity and customer support requests begin to rise. This can strain support staff, create unexpected margin exposure, and delay risk decisions.

This project tests whether changes in Reddit ticker mentions and sentiment can help predict volume spikes on the next trading day. The resulting pre-market watchlist would help these teams adjust staffing, prioritize monitoring, and review risk controls before markets open.

## Data

| Item | Description |
|---|---|
| Sources | Reddit API through PRAW for text; Yahoo Finance through `yfinance` for trading volume |
| Collection | Comments from Daily Discussion threads in `r/wallstreetbets` and `r/stocks` across about 90 trading days — an estimated 80,000–120,000 comments before filtering |
| Unit of analysis | One row per ticker-day |
| Expected size | About 900 rows: 10 tickers across about 90 trading days, including days with zero mentions |
| Tickers | AAPL, MSFT, NVDA, TSLA, GME, PLTR, AMD, COIN, HOOD, and SOFI |
| Text field | `comment_text`: all comments mentioning a ticker on a given day, combined into one field |
| Key columns | `ticker`, `date`, `comment_text`, `n_mentions`, `subreddit_mix`, `prev_day_volume`, and `volume_20d_avg` |
| Target | `volume_spike`: whether next-day volume is at least 2 times the ticker's trailing 20-day average |
| Mention rule | Case-sensitive uppercase ticker match with a word boundary; `$` is optional |
| Zero-mention days | Retained because the watchlist must score every ticker each day, and no discussion may also be useful information |

The target is normalized by ticker so that a spike has a similar meaning for stocks with very different normal trading volumes. The uppercase mention rule reduces false matches for tickers such as COIN and HOOD, but it may miss lowercase mentions.

A small mock dataset is included in `data/sample_ticker_days.csv` to show the planned schema, including a zero-mention day.

## Approach

This is a **Path A classification project**. The initial target is next-day volume at least 2 times the trailing 20-day average. The threshold will be lowered to 1.5 times if the 2-times definition creates too few positive cases.

Planned features:

- TF-IDF unigrams and bigrams from Reddit comments
- Average VADER sentiment and the share of negative comments
- Mention velocity compared with the ticker's trailing 20-day average
- Uppercase-character ratio and exclamation-mark density — VADER already accounts for both, but its final score hides their actual magnitude, so the raw ratios are kept as separate features
- Controls: previous-day volume divided by its 20-day average, subreddit mix, an `earnings_tomorrow` flag, and day of the week

Working hypothesis: mention velocity will be more useful than sentiment, because attention matters more than mood.

Logistic regression is the primary model — it works well on a smaller dataset and its coefficients are interpretable. A gradient-boosting model will be compared against it using feature importance or SHAP values.

## Tools

The main pipeline is classical NLP: regex for ticker matching, TF-IDF and VADER for text features, and logistic regression for prediction. The target comes directly from `yfinance` volume data, so no manual outcome labeling is required.

VADER is a general-purpose lexicon, and its 7,506 entries do not include common trading vocabulary — `bullish`, `bearish`, `bagholding`, `tendies`, `moon`, `puts`, `calls`, and `squeeze` are all absent. To cover that gap, the Claude API will label roughly 500 comments for trading-specific sentiment, and a lighter classifier will be trained on those labels rather than calling the API per comment at inference time.

The two sentiment measures are kept as separate features: VADER for general emotion, the trained classifier for trading sentiment. Estimated labeling cost using Claude Haiku is about $0.10, assuming roughly 50,000 input tokens and 10,000 output tokens.

## Verification

Volume momentum alone could drive the prediction without any Reddit data. A volume-only baseline will be compared against a volume-plus-text model; if the second does not improve on the first, the Reddit features are not adding value.

Earnings are included as a control so that earnings-driven volume is not incorrectly attributed to Reddit activity. The model will also be run without that feature to measure how much of the prediction it carries. If earnings dominate, the feature is likely acting as a proxy — predictive, but not evidence of the relationship under study.

## Status

**Phase 1 — complete.** Proposal submitted, mock schema added, data collection in progress.

## Setup

Runnable collection and modeling scripts will be added under `src/` in a later phase. The planned local setup:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install pandas praw yfinance nltk scikit-learn vaderSentiment anthropic
```

Reddit API credentials and the Anthropic API key are stored in a local `.env` file and are not committed. Once the scripts are added, this section will list the exact collection, feature-building, training, and evaluation commands.
