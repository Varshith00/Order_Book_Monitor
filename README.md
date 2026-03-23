# Real-Time Order Book Monitoring and Multi-Exchange Sandbox Trading System

Production-style Python prototype for real-time order book analytics with exchange-agnostic signal generation and sandbox execution on Binance Testnet.

## Architecture Overview

- **Data Layer**
  - `exchange/base_exchange.py`
  - `exchange/binance_ws_client.py`
- **Processing Layer**
  - `orderbook/orderbook.py`
  - `orderbook/orderbook_manager.py`
- **Analysis Layer**
  - `analysis/liquidity_detector.py`
  - `analysis/imbalance_detector.py`
- **Strategy Layer** (exchange-agnostic)
  - `strategy/signal_engine.py`
- **Execution Layer** (interface-driven)
  - `execution/base_execution_engine.py`
  - `execution/binance_executor.py`
  - `execution/execution_manager.py`
- **Domain Models**
  - `models/order.py`
  - `models/trade.py`
  - `models/position.py`
- **Infrastructure Layer**
  - `config/settings.py`
  - `infrastructure/logger.py`
  - `infrastructure/exceptions.py`

## Core Design Principles

- SOLID and dependency inversion via abstract execution and exchange interfaces
- Strategy logic remains exchange-agnostic and outputs `TradingSignal(symbol, side, quantity)`
- Config-driven selection of active exchanges and symbols
- Async orchestration with `asyncio` for streaming + execution routing
- Full type hints, dataclasses, and centralized structured logging

## Project Structure

```text
orderbook_trading_system/
│
├── main.py
├── requirements.txt
├── README.md
│
├── config/
│   └── settings.py
│
├── infrastructure/
│   ├── logger.py
│   └── exceptions.py
│
├── exchange/
│   ├── base_exchange.py
│   └── binance_ws_client.py
│
├── orderbook/
│   ├── orderbook.py
│   └── orderbook_manager.py
│
├── analysis/
│   ├── liquidity_detector.py
│   └── imbalance_detector.py
│
├── strategy/
│   └── signal_engine.py
│
├── execution/
│   ├── base_execution_engine.py
│   ├── binance_executor.py
│   ├── execution_manager.py
│   └── paper_trader.py
│
├── models/
│   ├── trade.py
│   ├── order.py
│   └── position.py
│
└── utils/
    └── helpers.py
```

## Environment Configuration

Configuration is loaded from environment variables using `python-dotenv`.

Create a `.env` file in the project root:

```env
# Runtime mode
TRADING_MODE=sandbox
LOG_LEVEL=INFO

# Strategy + order book
SYMBOL=BTCUSDT
DEPTH_LEVEL=20
IMBALANCE_THRESHOLD=1.5
LIQUIDITY_WALL_THRESHOLD=50
TRADE_SIZE=0.001

# Market data
WEBSOCKET_URL=wss://stream.binance.com:9443/ws

# Execution routing
EXCHANGES=binance_testnet
BINANCE_SYMBOL=BTCUSDT

# Risk controls
MAX_POSITION_SIZE=1.0
MAX_TRADES_PER_MINUTE=10

# Binance testnet credentials
BINANCE_API_KEY=your_binance_testnet_key
BINANCE_SECRET_KEY=your_binance_testnet_secret

```

## Exchange Account Setup

### Binance Testnet

1. Open Binance Spot Testnet portal: https://testnet.binance.vision
2. Create API key and secret.
3. Add them to `.env` as `BINANCE_API_KEY` and `BINANCE_SECRET_KEY`.

## Signal to Execution Flow

1. `main.py` fetches one Binance REST depth snapshot.
2. `OrderBookManager` applies snapshot levels to local order book.
3. `SignalEngine` emits `TradingSignal(symbol, side, quantity)`.
4. `ExecutionManager` routes signal to configured execution adapters.
5. Risk checks are applied:
   - max position size
   - max trades per minute
   - account balance check
6. Trade result is returned as JSON-serializable Lambda output.

```text
[EXECUTION] Exchange: Binance Testnet Symbol: BTCUSDT Side: BUY Quantity: 0.010000 Price: 68320.0000
```

## Installation

```bash
pip install -r requirements.txt
```

## Run

```bash
python main.py
```

## Dependencies

- `requests`
- `certifi`
- `python-dotenv`
- `typing_extensions`
