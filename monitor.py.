
import json
import os
import urllib.parse
import urllib.request
from pathlib import Path

TOKEN = os.environ["TELEGRAM_BOT_TOKEN"]
CHAT_ID = os.environ["TELEGRAM_CHAT_ID"]
SEEN_FILE = Path("seen.json")
API_URL = (
    "https://www.gamerpower.com/api/filter?"
    "platform=steam.epic-games-store.gog&type=game"
)

def get_json(url):
    request = urllib.request.Request(
        url, headers={"User-Agent": "FreeGamesMonitor/1.0"}
    )
    with urllib.request.urlopen(request, timeout=30) as response:
        return json.loads(response.read().decode("utf-8"))

def send_telegram(message):
    url = f"https://api.telegram.org/bot{TOKEN}/sendMessage"
    payload = urllib.parse.urlencode({
        "chat_id": CHAT_ID,
        "text": message
    }).encode()
    request = urllib.request.Request(url, data=payload)
    with urllib.request.urlopen(request, timeout=30) as response:
        result = json.loads(response.read().decode("utf-8"))
    if not result.get("ok"):
        raise RuntimeError("Telegram rejected the message")

data = get_json(API_URL)

if isinstance(data, dict):
    games = data.get("giveaways", [])
else:
    games = data

if not isinstance(games, list):
    raise RuntimeError("Unexpected giveaway API response")

seen = set()
if SEEN_FILE.exists():
    seen = set(json.loads(SEEN_FILE.read_text()))

new_games = [
    game for game in games
    if game.get("id") is not None
    and str(game["id"]) not in seen
    and game.get("status", "Active").lower() == "active"
]

# Send one digest instead of spamming Telegram.
batch = new_games[:15]

if batch:
    lines = ["🎮 NEW FREE PC GAME GIVEAWAYS", ""]
    for game in batch:
        title = game.get("title", "Untitled game")
        platforms = game.get("platforms", "PC")
        end_date = game.get("end_date", "Check offer page")
        link = (
            game.get("open_giveaway_url")
            or game.get("open_giveaway")
            or game.get("gamerpower_url")
            or "https://www.gamerpower.com/"
        )
        lines.extend([
            f"🎁 {title}",
            f"Platform: {platforms}",
            f"Ends: {end_date}",
            f"Claim/check: {link}",
            ""
        ])

    lines.append("Giveaway data: https://www.gamerpower.com/")
    send_telegram("\n".join(lines))

    seen.update(str(game["id"]) for game in batch)
    SEEN_FILE.write_text(json.dumps(sorted(seen), indent=2))
elif os.environ.get("GITHUB_EVENT_NAME") == "workflow_dispatch":
    send_telegram(
        "✅ Free Games Monitor test succeeded. "
        "No new full-game giveaways were found."
    )

print(f"Active offers checked: {len(games)}")
print(f"New offers sent: {len(batch)}")
