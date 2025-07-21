# Telegram-Signal-Bot
Signal
import requests
import time
from datetime import datetime
import pytz
import ccxt

# === CONFIGURATION ===
TOKEN = "7985476369:AAGGsKPrKDxVssHOOP3Wu_qqwV-h-s6XCSw"
CHANNEL_ID = "-1005860926205"
TIMEZONE = "Asia/Kolkata"
INTERVAL = '1h'
COINS = ["BTC/USDT", "ETH/USDT", "SOL/USDT", "BNB/USDT", "ADA/USDT", "XRP/USDT", "AVAX/USDT", "DOGE/USDT", "TON/USDT", "LINK/USDT", "ZORA/USDT"]

exchange = ccxt.binance()

def get_ohlcv(symbol):
    try:
        ohlcv = exchange.fetch_ohlcv(symbol, INTERVAL)
        return ohlcv[-2:]  # Previous and current candles
    except:
        return None

def get_macd(close_prices):
    ema12 = sum(close_prices[-12:]) / 12
    ema26 = sum(close_prices[-26:]) / 26
    macd_line = ema12 - ema26
    signal_line = sum(close_prices[-9:]) / 9
    return macd_line, signal_line

def get_rsi(closes, period=14):
    if len(closes) < period + 1:
        return None
    gains = []
    losses = []
    for i in range(-period, -1):
        change = closes[i + 1] - closes[i]
        if change >= 0:
            gains.append(change)
        else:
            losses.append(abs(change))
    average_gain = sum(gains) / period
    average_loss = sum(losses) / period
    if average_loss == 0:
        return 100
    rs = average_gain / average_loss
    return 100 - (100 / (1 + rs))

def send_telegram_message(message):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    payload = {
        "chat_id": CHANNEL_ID,
        "text": message,
        "parse_mode": "HTML"
    }
    requests.post(url, json=payload)

def check_signal(symbol):
    data = get_ohlcv(symbol)
    if not data or len(data) < 2:
        return

    closes = [x[4] for x in exchange.fetch_ohlcv(symbol, INTERVAL, limit=100)]
    macd, signal = get_macd(closes)
    rsi = get_rsi(closes)

    current_price = closes[-1]
    timestamp = datetime.now(pytz.timezone(TIMEZONE)).strftime("%Y-%m-%d %H:%M:%S")

    if macd > signal and rsi and rsi < 40:
        sl = round(current_price * 0.97, 6)
        tp1 = round(current_price * 1.03, 6)
        tp2 = round(current_price * 1.06, 6)
        message = f"📈 <b>Buy Signal - {symbol}</b>\n🕐 {timestamp}\n💰 Price: {current_price}\n🧠 MACD Bullish + RSI: {round(rsi,2)}\n🛑 SL: {sl}\n🎯 TP1: {tp1}, TP2: {tp2}"
        send_telegram_message(message)

    elif macd < signal and rsi and rsi > 60:
        sl = round(current_price * 1.03, 6)
        tp1 = round(current_price * 0.97, 6)
        tp2 = round(current_price * 0.94, 6)
        message = f"📉 <b>Sell Signal - {symbol}</b>\n🕐 {timestamp}\n💰 Price: {current_price}\n🧠 MACD Bearish + RSI: {round(rsi,2)}\n🛑 SL: {sl}\n🎯 TP1: {tp1}, TP2: {tp2}"
        send_telegram_message(message)

def run_bot():
    while True:
        for symbol in COINS:
            check_signal(symbol)
        time.sleep(3600)

if __name__ == "__main__":
    run_bot()
    requests
ccxt
pytz
