# sentinel_automata

AI-Driven Trading Backend Documentation

This document describes the components, setup, and usage of the FastAPI-based, AI-driven trading backend. It is written for users with beginner-to-intermediate programming experience.

Table of Contents

Overview

System Requirements

Installation & Setup

Database Initialization

API Endpoints

Script Overviews

main.py

strategy_engine.py

strategy_verdict.py

strategy_db.py

utils.py

inspect_db.py (optional)

Running the Server

Example Workflow

Next Steps & Extensions

Overview

You have built an MVP backend for an AI-driven trading fund. The key flow is:

Strategy Generation (external or via n8n) produces Python code defining generate_signals(df).

Backtesting: The code is sent to /run_backtest, which loads hourly BTC-USD data, executes the strategy, computes performance metrics, applies a rule-based verdict, and saves everything to SQLite.

Review & Alerts: Results (performance + verdict) can be fetched, displayed in dashboards, or trigger alerts.

All storage is in strategy_results.db and price_data.db.

System Requirements

Python 3.11+

pip

SQLite (built into Python)

yfinance, pandas, fastapi, uvicorn

Install dependencies:

pip install fastapi uvicorn pandas yfinance

Installation & Setup

Clone the repo to your local machine or cloud IDE.

Create a virtual environment (optional but recommended):

python -m venv venv
source venv/bin/activate  # Linux/Mac
venv\Scripts\activate   # Windows

Install Python packages:

pip install -r requirements.txt

Initialize the price database by running your fetcher script:

python price_fetcher.py

This creates price_data.db and downloads 90 days of hourly BTC-USD bars.

Initialize the strategy database is handled automatically when the FastAPI server starts.

Database Initialization

price_data.db: stores raw price bars in table btc_usd_hourly.

strategy_results.db: stores backtest results in table strategies (created by init_strategy_db()).

To reset price DB:

python price_fetcher.py   # deletes and re-creates price_data.db

API Endpoints

GET /

Simple health check. Returns:

{ "msg": "FastAPI is working" }

POST /run_backtest

Input: JSON with key strategy_code, a string containing Python code defining:



def generate_signals(df):
# df is a pandas DataFrame with BTC hourly bars
# must return a pd.Series of integer signals (-1,0,1)

- **Process**:
  1. Loads price data.
  2. Runs the code safely with `exec()`.
  3. Validates signals, calculates performance, verdict, and saves.
- **Output**: JSON with:
  - `status`: "success", "duplicate", or "error"
  - `message`: confirmation or error text
  - `performance`: cumulative return, drawdown, trades, win rate, avg trade
  - `verdict`: "excellent", "good", "average", "poor", or "reject"
  - `preview`: first 10 signal values
  - `code_hash`: unique identifier for strategy code

### POST `/update_verdict`

- **Input**: JSON with `code_hash` and manual `verdict`.
- **Process**: Overrides the stored `verdict` in SQLite (for manual review).
- **Output**: Confirmation JSON.

---

## Script Overviews

### `main.py`

- **Purpose**: Entry point for FastAPI. Defines endpoints and error handling.
- **Key functions**:
  - `root()`: health check.
  - `run_backtest()`: calls `run_strategy_pipeline()`.
  - `update_verdict_handler()`: calls DB update for manual overrides.
- **Automatic tasks**: calls `init_strategy_db()` at startup.

Paste this at top of `main.py`:
```python
"""
main.py: Entry point for the FastAPI backend.
Initializes the databases and exposes two endpoints:
- GET /
- POST /run_backtest
- POST /update_verdict
"""

strategy_engine.py

Purpose: Implements run_strategy_pipeline(), the backtest driver.

Workflow:

Load price data from SQLite.

Execute user-provided generate_signals(df) code.

Validate signals, count and hash duplicates.

Calculate performance metrics.

Call evaluate_performance() for verdict.

Save strategy record to DB.

Returns: A dict for the API response.

Paste this at top of strategy_engine.py:

"""
strategy_engine.py: Core backtest pipeline.
Defines run_strategy_pipeline(), which:
- Loads price data
- Executes user strategy code
- Calculates performance and verdict
- Saves results to SQLite
"""

strategy_verdict.py

Purpose: Classify strategies by performance using simple rules.

Function: evaluate_performance(performance: dict) -> str

Rules (tuned for 90-day hourly BTC data):

Reject if <30 trades.

Excellent: ≥20% return, <15% drawdown, and (win_rate ≥55% OR (win_rate ≥50% & avg_trade >1.5%)).

Good: ≥15% return, <15% drawdown, win_rate ≥52%, avg_trade >0.5%.

Average: ≥5% return & <20% drawdown.

Poor: <0% return OR >30% drawdown.

Otherwise reject.

Paste this at top of strategy_verdict.py:

"""
strategy_verdict.py: Rule-based performance classifier.
Defines evaluate_performance() to return verdict strings.
"""

strategy_db.py

Purpose: SQLite helper functions for storing and updating backtests.

Key functions:

init_strategy_db()

get_hash()

signal_exists() / code_exists()

save_strategy()

update_verdict() / update_verdict_by_signal_hash()

Schema: Table strategies with columns:
id, code_hash, strategy_code, signal_counts, performance, signal_hash, verdict, created_at.

Paste this at top of strategy_db.py:

"""
strategy_db.py: SQLite helpers for strategy storage.
Creates and manipulates the `strategies` table.
"""

utils.py

Purpose: Utility functions for hashing signals for deduplication.

Key function:

hash_signals(series: pd.Series) -> str

Paste this at top of utils.py:

"""
utils.py: Utility functions.
Provides hash_signals() to detect duplicate signal series.
"""

inspect_db.py (optional)

Purpose: Command-line script to view the latest saved strategies.

Usage: python inspect_db.py prints the last 5 strategies with performance and verdict.

Paste this at top of inspect_db.py:

"""
inspect_db.py: CLI tool to print the latest strategies.
Helps debug and confirm stored results.
"""

Running the Server

Ensure price_data.db exists (run your fetcher).

Start FastAPI:

uvicorn main:app --reload --host 0.0.0.0 --port 3000

Call endpoints as needed from n8n or curl.

Example Workflow

Generate new strategies via n8n + OpenAI.

Fan out into 5–10 POSTs to /run_backtest.

Collect responses with performance & verdict.

Slack alert for “excellent” or high-Sharpe strategies.

Monitor in a simple Streamlit dashboard.

Next Steps & Extensions

Add Sharpe & Calmar ratios in strategy_engine.py.

Implement paper-trading simulator and order-book.

Build a lightweight Streamlit dashboard.

Automate daily summary emails via n8n + OpenAI.

Expand to multi-asset support.
