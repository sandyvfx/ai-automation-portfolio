# AEGIS Runtime Stack Runbook — Phase 40.I

## 1. Purpose
This runtime stack is for **PAPER DATA ONLY**. It provides a high-fidelity dashboard and runtime monitoring console using Binance public market data. It is decoupled from any live trading, real capital, or broker routing.

## 2. Required Terminals
To run the full dashboard stack, open three separate PowerShell terminals and execute the following commands in order:

### Terminal 1 — Binance Ingest
Responsible for streaming live market data to the local cache.
```powershell
Set-Location C:\TV_AUTO_ANALYSIS\Chart_Engine
.\aegis_env\Scripts\Activate.ps1
python paper_trading\binance_ingest.py
```

### Terminal 2 — FastAPI Backend
The data orchestrator and REST API provider.
```powershell
Set-Location C:\TV_AUTO_ANALYSIS\Chart_Engine
.\aegis_env\Scripts\Activate.ps1
python -m uvicorn backend.aegis_api:app --host 127.0.0.1 --port 8000 --log-level debug
```

### Terminal 3 — React Frontend
The institutional-grade command center UI.
```powershell
Set-Location C:\TV_AUTO_ANALYSIS\Chart_Engine\frontend
npm run dev
```

## 3. Dashboard URLs
Once all terminals are running, the dashboard is accessible at:
- **Main Entry:** [http://localhost:5173/](http://localhost:5173/)
- **Overview:** [http://localhost:5173/#overview](http://localhost:5173/#overview)
- **Paper Trading:** [http://localhost:5173/#paper-trading](http://localhost:5173/#paper-trading)
- **Configuration:** [http://localhost:5173/#configuration](http://localhost:5173/#configuration)

## 4. Smoke Test Commands
Verify the health of the stack using these PowerShell commands:

```powershell
# Verify Runtime Configuration
Invoke-RestMethod http://127.0.0.1:8000/api/config/runtime | ConvertTo-Json -Depth 5

# Verify Live Paper Feed
Invoke-RestMethod http://127.0.0.1:8000/api/paper/feed | ConvertTo-Json -Depth 5

# Verify Runtime Process Health
Invoke-RestMethod http://127.0.0.1:8000/api/runtime/health | ConvertTo-Json -Depth 5

# Verify L2 Market Depth (Public Binance)
Invoke-RestMethod http://127.0.0.1:8000/api/market/depth | ConvertTo-Json -Depth 5

# Verify Paper Event Ledger
Invoke-RestMethod http://127.0.0.1:8000/api/paper/events | ConvertTo-Json -Depth 5

# Verify Session Summary
Invoke-RestMethod http://127.0.0.1:8000/api/paper/session-summary | ConvertTo-Json -Depth 5
```

## 5. Expected Healthy Outputs
A healthy stack will display the following markers in the smoke test JSON:
- `config_status`: "OK"
- `runtime_mode`: "PAPER_ONLY"
- `status`: "LIVE_FEED_ACTIVE" (when heartbeats are fresh)
- `heartbeat_age_seconds`: <= 60
- `active_threshold_seconds`: 60
- `source_file`: "reports/live_paper_feed.json"
- **Safety Flags (MUST BE TRUE):**
  - `paper_data_only`: true
  - `no_api_keys`: true
  - `no_live_orders`: true
  - `no_broker_routing`: true

## 6. Troubleshooting
- **Backend connection fails:** Restart Terminal 2 (Uvicorn).
- **Port 8000 is blocked:**
  ```powershell
  netstat -ano | findstr :8000
  Get-Process -Id <PID_FROM_NETSTAT>
  Stop-Process -Id <PID> -Force
  ```
- **Feed is stale:** Verify Terminal 1 is running; check for WebSocket errors in the console; restart `binance_ingest.py`.
- **Frontend_status is DOWN:** Verify Terminal 3 is running `npm run dev`.
- **Bid/Ask is 0 or missing:** Inspect `reports/live_paper_feed.json` to see if the ingest process is writing valid data.
- **Dashboard flickers stale:** Check if `heartbeat_age_seconds` is consistently crossing the 60s threshold.

## 7. Shutdown
To safely stop the stack, press `Ctrl+C` in each terminal in reverse order (Frontend -> Backend -> Ingest).

## 8. Forbidden Actions (SAFETY MANDATE)
- **DO NOT** run `paper_trading/run_live_paper_engine.py` yet.
- **DO NOT** integrate `core/confirmation_filter.py` yet.
- **DO NOT** add API keys or authenticated endpoints.
- **DO NOT** place live orders or add real broker credentials.
- **DO NOT** modify frozen model artifacts, replay, benchmark, or core risk logic.

## 9. Validation
Every stack launch must verify `safety_status: "READ_ONLY_AUDIT_COMPLETED"` and zero `determinism_violations` in the dashboard.
