# TIC QUANT - MARKET DATA ETL PIPELINE

## Abstract
This repository houses the Extract, Transform, and Load (ETL) engine for the Técnico Investment Club's Strategy Application team. The ultimate goal of this system is to act as the central system for our market data. 

The final architecture will be able to:
* Read a master configuration file defining multiple triplets of (Asset, Timeframe, Broker) across different asset classes.
* Download historical and live data from various sources (Alpaca, Binance, IBKR) using a modular design.
* Normalize raw API data into strict internal objects.
* Store everything persistently in a PostgreSQL database with data retention policies.
* Act as a gRPC server, listening to the team's various strategies and asynchronously sending them the required candles and lookbacks to their respective VMs.

---

## Phase 1: The MVP (Minimum Viable Product)
To establish a solid foundation, our first development milestone is a strictly functional MVP. **For this initial phase, the ETL will exclusively fetch CRYPTO data.**

* **Scope:** 10 predefined cryptocurrencies, extracted on an hourly timeframe, using the Alpaca API.
* **Warmup Logic:** On startup, the system queries the database to find the last stored candle for each asset and calculates the necessary lookback (up to 200 bars) to patch any missing history.
* **Synchronization:** The extraction loop does not use arbitrary time intervals. The system wakes up strictly at the top of the hour (e.g., 11:01) to fetch the perfect, fully consolidated candle of the previous hour (10:00:00 to 10:59:59). This eliminates look-ahead bias and mutable data issues.
* **Mock Exporter:** Instead of streaming data via gRPC to a strategy, the MVP concludes its cycle by dumping the final 200 candles of each crypto asset into a flat file as a proof of concept.

---

## Architecture & Directory Structure
The codebase strictly rejects loose scripts in favor of Object-Oriented Programming (OOP) and Dependency Injection.

* **`core/`**: Where Models and Configurations live. Contains `models.py` (with the `Candle` dataclass that travels throughout the system, banning raw dictionaries) and `config.py` (to statically load the crypto list on startup).
* **`extractors/`**: Isolates external HTTP communication. Contains the abstract base class (`broker.py`) enforcing the `fetch_candles` method, and child classes (e.g., `alpaca_broker.py`).
* **`storage/`**: The only place in the repository allowed to contain SQL code. Contains `postgres_db.py` with the `PostgresRepository` class.
* **`orchestrator/`**: The brain of the system. Contains `etl_engine.py`, tying the extractors and database together.

---

## Team Engineering Standards
* **Branch Protection & PRs:** The `main` branch is protected. Direct pushes are blocked. All code must be developed in a separate branch (e.g., `feature/alpaca-fetcher`) and submitted via Pull Request.
* **Codeowners:** Merging is blocked until explicit review and approval from department leads, enforced by the `CODEOWNERS` file.
* **Strong Typing:** Use native Python Type Hints in all functions. If a function returns a list of bars, the signature must be `-> List[Candle]`.

---

## Local Setup
1. Ensure Docker and Docker Compose are installed on your machine.
2. Create an `.env` file at the root (never commit this to GitHub) containing `ALPACA_API_KEY` and `ALPACA_API_SECRET`.
3. Run `docker-compose up -d` to spin up both the PostgreSQL database and the Python ETL container simultaneously.
4. Hourly orchestration logs will be visible in the main container's terminal output.