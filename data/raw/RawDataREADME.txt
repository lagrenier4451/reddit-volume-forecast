# Data Source

## reddit_wsb.csv

Source: Gabriel Preda, “Reddit WallStreetBets Posts,” Kaggle.
https://www.kaggle.com/datasets/gpreda/reddit-wallstreetsbets-posts

Downloaded: October 16, 2026.

The source describes collecting posts from r/wallstreetbets using PRAW,
the Python Reddit API Wrapper. We downloaded the existing dataset;
we did not collect these posts ourselves.

The downloaded file contains 53,187 posts and eight columns:
title, score, id, url, comms_num, created, body, and timestamp.

Most records cover January 28–August 16, 2021, with one earlier
record from September 2020. Some dates are absent, and many posts
have no body text. All posts have a title.

Each row represents a post, not an individual comment.
The comms_num field gives the comment count, not comment text.

The original CSV is stored at data/raw/reddit_wsb.csv without modification.
Cleaning and analysis are documented in the project notebooks.

The repository's MIT license applies to our code. This externally
sourced dataset remains subject to its original rights and terms.