import os
from flask import Flask, request, jsonify
from datetime import datetime
from dotenv import load_dotenv
import logging

# Load env vars
load_dotenv()

# Setup Flask app
app = Flask(__name__)

# Setup logging
LOG_DIR = "logs"
os.makedirs(LOG_DIR, exist_ok=True)
log_filename = datetime.now().strftime("%Y-%m-%d") + ".log"
logging.basicConfig(
    filename=os.path.join(LOG_DIR, log_filename),
    level=logging.INFO,
    format="%(asctime)s - %(message)s"
)

# Util to log and print
def log_event(message):
    print(message)
    logging.info(message)

# Helper: Parse TradingView's 4-in-1 alert string
def parse_alert(data):
    try:
        payload = data.get("content") or data
        if isinstance(payload, str):
            payload = payload.strip()

        parts = payload.split(";")
        alert_data = {}

        for part in parts:
            if "=" in part:
                key, value = part.strip().split("=", 1)
                alert_data[key.strip().lower()] = value.strip()

        return alert_data
    except Exception as e:
        log_event(f"❌ Failed to parse alert: {e}")
        return None

# Main webhook route
@app.route("/webhook", methods=["POST"])
def webhook():
    try:
        data = request.get_json(force=True)
        log_event(f"📩 Received raw alert: {data}")

        alert = parse_alert(data)
        if not alert:
            return jsonify({"error": "Invalid alert format"}), 400

        # Log parsed data
        log_event(f"✅ Parsed alert: {alert}")

        # Example action (you can replace this)
        symbol = alert.get("symbol")
        side = alert.get("side")
        tf = alert.get("tf")
        signal = alert.get("signal")
        log_event(f"⚡ Action: {side} signal on {symbol} ({tf}) - {signal}")

        return jsonify({"status": "ok"}), 200

    except Exception as e:
        log_event(f"❌ Webhook error: {e}")
        return jsonify({"error": str(e)}), 500

# Health check (optional)
@app.route("/", methods=["GET"])
def root():
    return "TradingView Webhook Receiver Online ✅", 200

# Run app
if __name__ == "__main__":
    port = int(os.getenv("PORT", 5000))
    app.run(host="0.0.0.0", port=port)

