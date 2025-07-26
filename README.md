# Moonx47
/quotex-signal-bot
├── app.py
├── requirements.txt
├── frontend/
│    └── index.html
└── README.md
from flask import Flask, jsonify
import pandas as pd
import numpy as np
import ta
import time

app = Flask(__name__)
PAIRS = ["EURUSD", "GBPUSD", "USDJPY", "USDCHF", "AUDUSD", "USDCAD"]

# Dummy data generator (replace with real feeds later)
def get_prices(pair):
    np.random.seed(int(time.time()) % 100000 + hash(pair) % 1000)
    price = np.cumsum(np.random.randn(100)) + 100
    df = pd.DataFrame({'close': price})
    df['open'] = df['close'].shift(1).fillna(method='bfill')
    df['high'] = df['close'] + np.random.rand(100)
    df['low'] = df['close'] - np.random.rand(100)
    df['volume'] = np.random.randint(100, 1000, size=100)
    return df

def compute_signal(df):
    df['rsi'] = ta.momentum.RSIIndicator(df['close'], window=14).rsi()
    macd = ta.trend.MACD(df['close'])
    df['macd'] = macd.macd()
    df['macd_signal'] = macd.macd_signal()
    last = df.iloc[-1]
    if last['rsi'] < 30 and last['macd'] > last['macd_signal']:
        return "BUY"
    if last['rsi'] > 70 and last['macd'] < last['macd_signal']:
        return "SELL"
    return "HOLD"

@app.route('/signals')
def signals():
    out = {}
    for pair in PAIRS:
        df = get_prices(pair)
        out[pair] = {
            "signal": compute_signal(df),
            "rsi": round(df['rsi'].iloc[-1], 2),
            "macd": round(df['macd'].iloc[-1], 4),
            "macd_signal": round(df['macd_signal'].iloc[-1], 4)
        }
    return jsonify(out)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
    flask
pandas
numpy
ta
gunicorn
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Quotex Signal Dashboard</title>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; }
    table { margin: auto; border-collapse: collapse; width: 80%; }
    th, td { padding: 12px; border: 1px solid #ccc; }
    .BUY { color: green; font-weight: bold; }
    .SELL { color: red; font-weight: bold; }
    .HOLD { color: gray; font-weight: normal; }
  </style>
</head>
<body>
  <h1>Quotex Signal Dashboard</h1>
  <table id="signals">
    <tr><th>Pair</th><th>Signal</th><th>RSI</th><th>MACD</th><th>MACD Signal</th></tr>
  </table>
  <p>Updating every minute...</p>
  <script>
    async function load() {
      const res = await fetch('/signals');
      const data = await res.json();
      const tbl = document.getElementById('signals');
      tbl.innerHTML = '<tr><th>Pair</th><th>Signal</th><th>RSI</th><th>MACD</th><th>MACD Signal</th></tr>';
      for (const [pair, info] of Object.entries(data)) {
        const row = tbl.insertRow();
        row.insertCell().innerText = pair;
        const sig = row.insertCell(); sig.innerText = info.signal; sig.className = info.signal;
        row.insertCell().innerText = info.rsi;
        row.insertCell().innerText = info.macd;
        row.insertCell().innerText = info.macd_signal;
      }
    }
    load(); setInterval(load, 60000);
  </script>
</body>
</html>
# Quotex Signal Trading Bot Dashboard

## Overview
This project runs a Flask backend that simulates signal generation (RSI + MACD) for popular currency pairs, and a frontend dashboard to display Buy/Sell/Hold signals.

## ⚙️ Deploy Locally
1. `pip install -r requirements.txt`  
2. `python app.py`  
3. Open `http://localhost:5000/frontend/index.html` in your browser.

## 🚀 Deploy on Render.com
- Push this repo to GitHub.
- Create a new Web Service on Render using the repo.
- Set:
  - **Build command:** `pip install -r requirements.txt`
  - **Start command:** `gunicorn app:app`
- Deploy — access your app live at your Render URL (e.g. `https://your-app.onrender.com`).

## 🧩 Customization Tips
- Replace `get_prices()` with real market data fetchers (Alpha Vantage, Yahoo Finance, or live scraped quotes).
- Add more pairs by editing `PAIRS` list.
- Adjust RSI/MACD thresholds in `compute_signal()`.
