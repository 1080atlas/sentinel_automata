# Sentinel Automata

This repository contains an MVP backend for a fully AI‑driven trading fund. It uses FastAPI, SQLite, and simple rule-based logic to backtest strategies, evaluate performance, and store results.

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

This backend powers an AI‑driven trading pipeline:

1. **Strategy Generation** (via n8n/OpenAI) produces Python functions `generate_signals(df)`.
2. **Backtesting**: POST code to `/run_backtest`, which:

   * Loads BTC‑USD hourly data.
   * Executes `generate_signals`.
   * Computes performance & risk metrics.
   * Applies a rule‑based verdict.
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

   This creates `price_data.db` with 90 days of BTC‑USD hourly bars.

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

### POST `/run_backtest`

* **Request JSON**:

  ```json
  { "strategy_code": "<Python code defining generate_signals(df)>" }
  ```
* **Response JSON**:

  ```json
  {
    "status": "success|duplicate|error",
    "message": "...",
    "performance": { ... },
    "verdict": "excellent|good|average|poor|reject",
    "preview": { ... },
    "code_hash": "..."
  }
  ```

### POST `/update_verdict`

* **Request JSON**:

  ```json
  { "code_hash": "...", "verdict": "good" }
  ```
* **Response JSON**:

  ```json
  { "status": "success", "message": "Verdict updated" }
  ```

---

## Script Overviews

### `main.py`

```python
"""
main.py: Initializes DB and defines API endpoints:
- GET  /
- POST /run_backtest
- POST /update_verdict
"""
```

### `strategy_engine.py`

```python
"""
strategy_engine.py: Backtest pipeline.
Defines run_strategy_pipeline():
- Loads price data
- Executes user strategy code
- Calculates performance & risk metrics
- Applies rule-based verdict
- Saves to SQLite
"""
```

### `strategy_verdict.py`

```python
"""
strategy_verdict.py: Rule-based performance classifier.
Defines evaluate_performance() → verdict string.
"""
```

### `strategy_db.py`

```python
"""
strategy_db.py: SQLite helpers for strategy storage.
Creates/manages the `strategies` table.
"""
```

### `utils.py`

```python
"""
utils.py: Utility functions.
Provides hash_signals() for duplicate detection.
"""
```

### `inspect_db.py`

```python
"""
inspect_db.py: CLI tool to print the latest strategies.
Helps verify stored results.
"""
```

---

## Running the Server

```bash
uvicorn main:app --reload --host 0.0.0.0 --port ####
```

---

## Example Workflow

1. Use n8n/OpenAI to generate 3–5 strategy codes.
2. Fan out POSTs to `/run_backtest` for each.
3. Merge results and trigger alerts for “excellent” verdicts.
4. Build a simple Streamlit dashboard reading `strategy_results.db`.

---

## Next Steps & Extensions

* Add Sharpe & Calmar ratios in backtest.
* Implement paper-trading simulator.
* Build a Streamlit or React dashboard.
* Automate daily summaries via n8n + OpenAI.
* Extend to multiple assets.

---



