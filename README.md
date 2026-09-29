# reddit-volume-forecast



This project predicts which watched stock tickers are likely to experience abnormal next-day trading volume so a retail brokerage can prepare before markets open.



\## Problem



A retail brokerage's market intelligence and risk teams monitor stocks that may create unusual trading activity and exposure. Today, they may not learn that a ticker has gone viral on Reddit until order volume and support requests are already increasing. This can lead to overloaded support queues, unexpected margin exposure, and slower risk decisions. The planned system will produce a pre-market watchlist so the brokerage can adjust staffing, monitoring, and risk controls before markets open.



\## Data



| Item | Description |

|---|---|

| Sources | Reddit API through PRAW for text; Yahoo Finance through `yfinance` for trading volume |

| Collection | Comments from Daily Discussion threads in `r/wallstreetbets` and `r/stocks` across about 90 trading days |

| Unit of analysis | One row per ticker-day |

| Expected size | About 900 rows: 10 tickers across about 90 trading days |

| Tickers | AAPL, MSFT, NVDA, TSLA, GME, PLTR, AMD, COIN, HOOD, and SOFI |

| Text field | `comment\_text`: all comments mentioning a ticker on a given day, combined into one field |

| Key columns | `ticker`, `date`, `comment\_text`, `n\_mentions`, `subreddit\_mix`, `prev\_day\_volume`, and `volume\_20d\_avg` |

| Target | `volume\_spike`: whether next-day volume is at least 2 times the ticker's trailing 20-day average |

| Mention rule | Case-sensitive uppercase ticker match with a word boundary; `$` is optional |

| Zero-mention days | Retained because the watchlist must score every ticker each day, and no discussion may also be useful information |



The target is normalized by ticker so that a spike has a similar meaning for stocks with very different normal trading volumes. The uppercase mention rule reduces false matches for tickers such as COIN and HOOD, but it may miss lowercase mentions.



A small mock dataset is included in `data/sample\_ticker\_days.csv` to show the planned schema, including a zero-mention day.



\## Approach



This is a \*\*Path A classification project\*\*. The initial target is next-day volume at least 2 times the trailing 20-day average. The threshold will be lowered to 1.5 times if the 2-times definition creates too few positive cases.



Planned features include:



\- TF-IDF unigrams and bigrams from Reddit comments

\- Average VADER sentiment and the share of negative comments

\- Mention velocity compared with the ticker's trailing 20-day average

\- Uppercase-character ratio and exclamation-mark density

\- Previous-day volume momentum, subreddit mix, earnings timing, and day of the week



Logistic regression will be the primary model because its coefficients are easy to interpret. A gradient-boosting model may be used as a comparison. A volume-only baseline will be compared with a volume-plus-text model to test whether Reddit features add useful information instead of simply repeating existing volume momentum.



\## Status



\*\*Phase 1:\*\* Proposal submitted, mock schema added, and data collection in progress.



\## Setup



Runnable collection and modeling scripts will be added under `src/` in a later phase. The planned local setup is:



```powershell

python -m venv .venv

.\\.venv\\Scripts\\Activate.ps1

pip install pandas praw yfinance nltk scikit-learn vaderSentiment

```



Reddit API credentials should be stored in a local `.env` file and must not be committed. Once the scripts are added, this section will be updated with the exact collection, feature-building, training, and evaluation commands.



