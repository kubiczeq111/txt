# txt[txt.py](https://github.com/user-attachments/files/22665138/txt.py)
txt
from flask import Flask, send_file, request, abort
import requests
import os
from datetime import datetime

app = Flask(__name__)

# <-- Wklej tutaj swój Discord webhook URL -->
WEBHOOK_URL = "https://discord.com/api/webhooks/https://discord.com/api/webhooks/1423361413484773546/-nVQW73tV5bujYmgaPVPIXQMh9KXNyPTzvA7fRWoToaCQV4Qj44sM3FB1jx_C725I8ou" \
"

ZIP_NAME = "texturepack.zip"
LOG_FILE = "pobrane_logi.txt"

def send_discord_message(content: str):
    try:
        requests.post(WEBHOOK_URL, json={"content": content}, timeout=5)
    except Exception as e:
        # Nie przerywamy pobierania jeśli webhook nie działa
        print("Błąd wysyłania webhooka:", e)

@app.route("/download")
def download_texture_pack():
    # Sprawdź czy plik istnieje
    if not os.path.isfile(ZIP_NAME):
        abort(404, description="Brak pliku texturepack.zip w katalogu serwera.")

    user = request.args.get("user", "Nieznany_uzytkownik")
    ip = request.remote_addr or "brak_IP"
    timestamp = datetime.utcnow().isoformat() + "Z"

    # 1) Wyślij powiadomienie na Discord (bez żadnych tokenów)
    message = f"🎮 **{user}** pobrał texture packa!\n• IP: `{ip}`\n• Czas (UTC): {timestamp}"
    send_discord_message(message)

    # 2) Zapisz wpis do pliku .txt
    try:
        with open(LOG_FILE, "a", encoding="utf-8") as f:
            f.write(f"{timestamp}\t{user}\t{ip}\n")
    except Exception as e:
        print("Błąd zapisu do pliku logu:", e)

    # 3) Zwróć plik .zip do pobrania
    return send_file(ZIP_NAME, as_attachment=True)

if __name__ == "__main__":
    # Ustaw host='0.0.0.0' jeśli chcesz wystawić lokalnie na innych interfejsach
    app.run(port=5000, debug=False)
