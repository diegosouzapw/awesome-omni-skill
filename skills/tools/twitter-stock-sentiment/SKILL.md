---
name: twitter-stock-sentiment
description: Analyze Twitter/X sentiment for stocks using $cashtags. Track mentions, sentiment scores, influencer activity, and trending discussions for any ticker.
license: MIT
metadata:
  author: Erik Malm
  version: "1.0.0"
allowed-tools: exec
---

# Twitter Stock Sentiment

Analyzes Twitter/X sentiment for stocks using the bird CLI and $cashtag tracking.

## Features

- **Mention Volume:** Track tweet counts for any $TICKER
- **Sentiment Analysis:** Bull/neutral/bear scoring using NLP
- **Influencer Tracking:** Identify high-follower accounts discussing the stock
- **Trending Hashtags:** Associated tags and themes
- **Crisis Detection:** Spikes in negative sentiment
- **7-day Trends:** Volume and sentiment changes

## Usage

```bash
./analyze.sh AAPL
./analyze.sh TSLA --days 30
```

## Output Format

Markdown report with:
- Sentiment score (-1 to +1)
- Mention volume and trend
- Top influencer tweets
- Hashtag analysis
- Notable sentiment shifts

## Requirements

- bird CLI installed (`brew install steipete/tap/bird`)
- Authenticated Twitter/X session

## Installation

1. Clone this repository to your skills folder
2. Install bird CLI: `brew install steipete/tap/bird`
3. Authenticate with Twitter/X: `bird auth`
4. Install Python dependencies: `pip install -r requirements.txt`

## How It Works

1. `analyze.sh` fetches recent tweets mentioning the $TICKER using bird CLI
2. `sentiment.py` performs NLP analysis on tweet text using VADER
3. Results are aggregated and formatted as a Markdown report
4. Influencer tracking identifies high-follower accounts
5. Hashtag extraction reveals trending themes

## Configuration

Edit `analyze.sh` to customize:
- Number of tweets to fetch (default: 100)
- Time window (default: 7 days)
- Sentiment thresholds for crisis detection
