# Institutional Mutual Fund Folio Scanner & Projection Engine

## Overview
A comprehensive, institutional-grade quantitative system for analyzing the Indian Mutual Fund ecosystem. This engine scans underlying folios, maps them across three structural time horizons, and projects forward alpha using a dynamic simulation overlay.

## Architecture

### Backend (Python)
- **data_models.py**: Core data structures for Stocks, Holdings, and Mutual Funds.
- **core_metrics.py**: Calculation logic for the 5 Core Institutional Metrics.
- **engine.py**: The main orchestrator (`FolioScanner` and `SimulationEngine`).

### Frontend (HTML5/JS)
- **dashboard/index.html**: A self-contained, interactive dark-mode dashboard using Tailwind CSS and Chart.js.

## Core Institutional Metrics

### 1. Days to Liquidate (DtL)
- **Formula**: `DtL_target = (AUM * Target%) / (Weighted_Avg_ADTV * 0.01)`
- **Threshold**: 1% Market Impact Cost
- **Targets**: 10%, 25%, 50% of AUM

### 2. Velocity Cash Drag
- **Formula**: `VCD = ((Avg_Inflow *  12 / AUM) * (Float_Days / 365) * 4.5%)`
- **Assumption**: 4.5 days average float (T+2 + TREPS)

### 3. Active Share
- **Formula**: `AS = 0.5 * sum(|W_fund - W_benchmark|)`
- **Flag**: < 60% = 'Closet Indexer'

### 4. Portfolio Turnover Ratio (PTR)
- **Formula**: Derived from category-specific benchmarks (Large: 20%, Mid: 45%, Small: 90%).

### 5. Rolling Alpha
- **Formula**: `Rolling_Alpha = Mean(Fund_Returns_Window) - Mean(Benchmark_Returns_Window)`
- **Window**: 12 months, stepped monthly.

## Time-Horizon Categorization
1. **Short-Term (<30-60 Days)**: High Velocity Cash Drag + Liquid (DtL 50% < 30 days).
2. **Mid-Term (60 Days - 1 Year)**: Capacity constrained (DtL 50% > 100 days).
3. **Long-Term (>1 Year)**: High Active Share (>70%), Master stock-pickers (Contra, Small, Flexi).

## Dynamic Simulation Engine

### Inputs
- `capital_size`: Capital deployment amount (₹5 Lakhs to ₹20 Crores).
- `start_date`: Analysis start date.
- `end_date`: Analysis end date.

### Logic
1. **Liquidity & Slippage**: `Slippage = min(2.5%, log(1 + Capital/ADTV) * 0.3)`
2. **Historical Flow**: Adjusts base alpha by `+0.2%` monthly (DII sentiment).
3. **Forward Projection**: `Projected_Return = (1 + (Bench_ROI + Adjusted_Alpha)) ^ Months - 1`

## Usage

### 1. Run the Backend Engine
```bash
cd "C:\IN MF START"
python engine.py
```
This will generate `dashboard/fund_data.json`.

### 2. Launch the Dashboard
Open `dashboard/index.html` in any modern web browser. The dashboard is self-contained and does not require a server (though a local server is recommended for fetching JSON).

## Files Generated
- `data_models.py`
- `core_metrics.py`
- `engine.py`
- `dashboard/index.html`
- `dashboard/fund_data.json` (generated after running engine.py)

## Disclaimer
This is a simulation engine using mock data for demonstration purposes. The slippage models and alpha projections are approximations and should not be used as sole guidance for actual capital deployment without further validation against live market microstructure data.
