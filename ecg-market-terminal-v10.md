# ECG Market Terminal  v1.0

A production-quality desktop application that renders live and historical
Binance USDT-M Futures price action as a real-time ECG waveform, with
a full structural-language engine, pattern-research panel, and deterministic
historical replay.

---

## Features

| Feature | Detail |
|---------|--------|
| **Live feed** | Binance WebSocket – 1m / 5m / 15m / 1h candles |
| **ECG canvas** | Session-relative ECG-style waveform; 60 FPS |
| **Swing detection** | Three-phase, no-repaint, threshold-based detector |
| **Shape grammar** | 13 structural shapes: TOP BOT HH LL LH HL EH EL DT DB FB EXP CMP |
| **Pattern research** | Sliding n-gram (2–5) frequency and follow-through statistics |
| **Replay** | Deterministic historical replay at 1×–100×; jump to any date |
| **Settings** | Persistent JSON config; theme switcher (Dark / Darker / Midnight) |
| **Export** | CSV swings, CSV shapes, CSV patterns, JSON session, TXT report, PNG screenshot |
| **Autosave** | Pattern snapshot every 5 minutes to `~/.emrt/autosave/` |
| **Crash recovery** | `sys.excepthook` writes crash report; notice shown on next start |

---

## Installation

**Requirements:** Python 3.12+ on Windows 11 (or macOS / Linux).

```bash
# 1. Clone or download the repository
git clone <repo-url>
cd ECG_NEW

# 2. Create a virtual environment
python -m venv .venv
.venv\Scripts\activate        # Windows
# or: source .venv/bin/activate  # macOS / Linux

# 3. Install dependencies (two packages only)
pip install -r requirements.txt

# 4. Run in live mode
python main.py

# 5. Run in replay mode
python main.py --replay
```

---

## Usage

### Live mode

```bash
python main.py
```

The terminal connects to Binance USDT-M Futures for the configured symbol
and streams candles in real time.

### Replay mode

```bash
python main.py --replay
```

A replay control bar appears at the bottom.  Use **▶ Play / ⏸ Pause**,
**◄◄ Prev** / **Next ▶▶** to navigate sessions, or type a date in **JUMP TO**
and press **GO**.

---

## Configuration

Settings are stored at `~/.emrt/config.json`.

Open the dialog from **File → Settings…** (Ctrl+,) while the app is running.

| Field | Default | Description |
|-------|---------|-------------|
| `symbol` | `btcusdt` | Binance trading pair (lower-case) |
| `interval` | `1m` | Candle interval |
| `log_level` | `DEBUG` | Logging verbosity |
| `swing_threshold` | `0.003` | Fractional reversal to confirm a swing (0.3%) |
| `session_start_hour_utc` | `0` | UTC hour of the daily session boundary |
| `session_start_minute_utc` | `0` | UTC minute of the daily session boundary |

Most changes require a restart.  Theme changes apply immediately for preview.

---

## Export

From the **File** menu:

| Action | Output |
|--------|--------|
| Export Swings (CSV) | All confirmed swings: price, timestamp, kind, size |
| Export Shapes (CSV) | All structural shapes: kind, session, metrics |
| Export Patterns (CSV) | Pattern statistics: occurrences, win-rate, avg/max/min move |
| Export Session (JSON) | Full session: swings + shapes + patterns |
| Export Report (TXT) | Human-readable session summary |
| Screenshot (PNG) | Full window capture (Ctrl+Shift+S) |

Files default to `~/.emrt/exports/` when no path is chosen.

---

## Themes

Switch from **View → Theme**:

| Name | Description |
|------|-------------|
| Dark | Default dark terminal palette |
| Darker | Extra-dark variant for low-light environments |
| Midnight | Deep blue-black with saturated accent colours |

---

## Testing

```bash
# Run all tests
python -m pytest tests/ -v

# Run a single suite
python -m pytest tests/test_swing_detector.py -v
```

| Test file | What is covered |
|-----------|-----------------|
| `test_swing_detector.py` | SwingDetector: phases, no-repaint guarantee, reset |
| `test_session_manager.py` | `session_boundary()`, `SessionManager` async handlers |
| `test_shape_grammar.py` | All 13 shape kinds, meta-shapes, replay-reset isolation |
| `test_pattern_engine.py` | N-gram windowing, outcome resolution, Welford statistics |
| `test_replay.py` | `kline_to_msg()`, `VALID_SPEEDS`, `ReplayEngine` import |

---

## Project Structure

```
ECG_NEW/
├── core/
│   ├── config_manager.py   Load/save ~/.emrt/config.json
│   ├── constants.py        Event names, default values
│   ├── crash_recovery.py   sys.excepthook crash handler
│   ├── event_bus.py        Async pub/sub dispatcher
│   └── logger.py           RotatingFileHandler + console
├── data/
│   ├── binance_ws.py       Binance WebSocket client with reconnect
│   └── candle_builder.py   Raw kline dict → Candle dataclass
├── engine/
│   ├── confirmation_engine.py  Swing detection pipeline
│   ├── session_manager.py      Daily session boundaries
│   ├── shape_database.py       Circular-buffer (deque) shape store
│   ├── shape_grammar.py        13-kind structural classifier
│   └── swing_detector.py       Three-phase no-repaint swing detector
├── export/
│   └── exporter.py         CSV / JSON / TXT / PNG export
├── models/
│   ├── candle.py           Frozen Candle dataclass
│   ├── session.py          SessionInfo dataclass
│   ├── shape.py            Shape dataclass + ShapeKind enum
│   ├── signal.py           BUY/SELL signal dataclass
│   └── swing.py            Swing dataclass + SwingKind enum
├── render/
│   ├── dashboard.py        Right-side status panel
│   ├── ecg_canvas.py       Main ECG waveform canvas (60 FPS)
│   ├── main_window.py      QMainWindow + menu bar + autosave
│   ├── replay_bar.py       Replay control bar widget
│   ├── research_panel.py   Pattern statistics panel
│   ├── settings_dialog.py  Settings editor dialog
│   ├── theme.py            Theme colour constants (QColor)
│   └── theme_manager.py    Runtime theme switching
├── replay/
│   ├── fetcher.py          Binance REST kline fetcher (urllib, no extra deps)
│   └── replay_engine.py    Deterministic replay engine
├── research/
│   ├── pattern_engine.py   Sliding n-gram pattern detector
│   └── statistics.py       O(1) Welford online pattern statistics
├── tests/
│   ├── test_pattern_engine.py
│   ├── test_replay.py
│   ├── test_session_manager.py
│   ├── test_shape_grammar.py
│   └── test_swing_detector.py
├── config.py               Frozen Config dataclass
├── main.py                 Entry point
└── requirements.txt        PySide6>=6.6.0, websockets>=12.0
```

---

## Architecture

```
Qt main thread                    asyncio thread (daemon)
─────────────────────             ───────────────────────────────
MainWindow._drain_queue()  ←───── SimpleQueue ←── EventBus
  │                                               │
  ├── EcgCanvas                                  ├── CandleBuilder
  ├── Dashboard                                  ├── SessionManager
  ├── ResearchPanel                              ├── ConfirmationEngine
  └── ReplayBar                                  │     └── SwingDetector
                                                 ├── ShapeGrammar
                                                 ├── ShapeDatabase
                                                 ├── PatternEngine
                                                 └── ReplayEngine / BinanceWebSocket
```

- Qt event loop owns the **main thread** (required by PySide6 on all platforms).
- The asyncio backend runs in a **daemon thread** with its own event loop.
- `queue.SimpleQueue` is the only shared state between threads — thread-safe, lock-free.
- All business logic runs in the asyncio thread; the Qt thread is pure rendering.
- No growing lists anywhere: `deque(maxlen=…)` for all buffers, Welford for statistics.

---

## Data Directory

| Path | Contents |
|------|----------|
| `~/.emrt/config.json` | Persistent settings |
| `~/.emrt/crash_report.json` | Last crash report (deleted on next start) |
| `~/.emrt/autosave/` | Rolling pattern autosave (last 3 files kept) |
| `~/.emrt/exports/` | Default export destination |
| `logs/emrt.log` | Rotating log — 10 MB per file, 5 backups |

---

## Reconnect Strategy

The live WebSocket uses full-jitter exponential backoff:

```
delay = clamp(BASE × MULTIPLIER^(attempt-1), 0, MAX) ± jitter
```

| Constant | Value |
|----------|-------|
| Base | 1.0 s |
| Multiplier | 2.0 |
| Max | 60.0 s |
| Jitter | ±10% |

---

## License

MIT
