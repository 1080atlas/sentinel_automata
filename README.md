# sentinel_automata

# AI-Driven Trading Backend

This repository contains an MVP backend for a fully AI-driven trading fund. It uses FastAPI, SQLite, and simple rule-based logic to backtest strategies, evaluate performance, and store results.

---

## Table of Contents

* [Overview](#overview)
* [System Requirements](#system-requirements)
* [Installation & Setup](#installation--setup)
* [Database Initialization](#database-initialization)
* [API Endpoints](#api-endpoints)
* [Script Overviews](#script-overviews)

  * [`main.py`](#mainpy)
  * [`strategy_engine.py`](#strategy_enginepy)
  * [`strategy_verdict.py`](#strategy_verdictpy)
  * [`strategy_db.py`](#strategy_dbpy)
  * [`utils.py`](#utilspy)
  * [`inspect_db.py`](#inspect_dbpy)
* [Running the Server](#running-the-server)
* [Example Workflow](#example-workflow)
* [Next Steps & Extensions](#next-steps--extensions)

---

## Overview

This backend powers an AI-driven trading pipeline:

1. **Strategy Generation** (via n8n/OpenAI) produces Python functions `generate_signals(df)`.
2. **Backtesting**: POST code to `/run_backtest`, which:

   * Loads BTC-USD hourly data.
   * Executes `generate_signals`.
   * Computes performance & risk metrics.
   * Applies a rule-based verdict.
   * Saves results in SQLite.
3. **Alerts & Dashboards** can be built on top of the stored results.

---

## System Requirements

* Python 3.11+
* pip
* SQLite (bundled with Python)
* Python packages: `fastapi`, `uvicorn`, `pandas`, `yfinance`

```bash
pip install fastapi uvicorn pandas yfinance
```

---

## Installation & Setup

1. **Clone** this repo.
2. **(Optional)** Create and activate a virtual environment:

   ```bash
   python -m venv venv
   source venv/bin/activate    # macOS/Linux
   venv\Scripts\activate     # Windows
   ```
3. **Install** dependencies:

   ```bash
   pip install -r requirements.txt
   ```
4. **Fetch price data**:

   ```bash
   python price_fetcher.py
   ```

   This creates `price_data.db` with 90 days of BTC-USD hourly bars.

---

## Database Initialization

* **price\_data.db**: Raw price bars (`btc_usd_hourly` table).
* **strategy\_results.db**: Strategy backtest results (`strategies` table).

`strategy_results.db` is created automatically when FastAPI starts.

---

## API Endpoints

### GET `/`

Health check:

```json
{ "msg": "FastAPI is working" }
```

### POST \`/run\_backtest
