# 📄 Yahoo Email Attachment → Telegram Bot  
This project automatically downloads attachments from a Yahoo Mail inbox (within the last 24 hours) and sends them to a Telegram chat via a bot.  

---

## 🚀 Features  
- Connects to Yahoo Mail via **IMAP**  
- Downloads PDF attachments to a local or cloud storage folder  
- Sends the downloaded file to a **Telegram chat**  
- Uses `.env` for credentials (security best practices)  
- Can run locally or be hosted online (various options discussed)  

---

## 📂 Project Structure  

```
YahooBot/
│
├── download_attachment.py   # Main script
├── .env                     # Environment variables (never commit to GitHub)
├── requirements.txt         # Python dependencies
└── README.md                # Documentation
```

---

## ⚙️ Requirements  

- Python 3.8+  
- Yahoo Mail account with **App Password** enabled  
- Telegram Bot Token (via [BotFather](https://core.telegram.org/bots#botfather))  
- Your Telegram Chat ID (from `getUpdates` API)  

---

## 📦 Installation & Setup  

### 1️⃣ Clone the repo & install dependencies  
```bash
git clone https://github.com/yourusername/YahooBot.git
cd YahooBot
pip install -r requirements.txt
```

---

### 2️⃣ Create `.env` file for secrets  

`.env`  
```env
YAHOO_USER=your_yahoo_email
YAHOO_PASS=your_yahoo_app_password
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
TELEGRAM_CHAT_ID=your_chat_id
IMAP_SERVER=imap.mail.yahoo.com
IMAP_PORT=993
```

---

### 3️⃣ Main script (download & send attachments)  

```python
import imaplib, email, os, requests
from datetime import datetime, timedelta
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

YAHOO_USER = os.getenv("YAHOO_USER")
YAHOO_PASS = os.getenv("YAHOO_PASS")
IMAP_SERVER = os.getenv("IMAP_SERVER")
IMAP_PORT = int(os.getenv("IMAP_PORT", 993))
TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
TELEGRAM_CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

# Connect to Yahoo IMAP
mail = imaplib.IMAP4_SSL(IMAP_SERVER, IMAP_PORT)
mail.login(YAHOO_USER, YAHOO_PASS)
mail.select("inbox")

# Search for last 24h emails
date_24h_ago = (datetime.now() - timedelta(days=1)).strftime("%d-%b-%Y")
status, data = mail.search(None, f'(SINCE "{date_24h_ago}")')

# Create downloads folder
os.makedirs("downloads", exist_ok=True)

# Process each email
for num in data[0].split():
    status, msg_data = mail.fetch(num, "(RFC822)")
    msg = email.message_from_bytes(msg_data[0][1])

    for part in msg.walk():
        if part.get_content_maintype() == "multipart":
            continue
        if part.get("Content-Disposition") is None:
            continue

        filename = part.get_filename()
        if filename and filename.lower().endswith(".pdf"):
            filepath = os.path.join("downloads", filename)
            with open(filepath, "wb") as f:
                f.write(part.get_payload(decode=True))

            # Send to Telegram
            with open(filepath, "rb") as f:
                requests.post(
                    f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendDocument",
                    data={"chat_id": TELEGRAM_CHAT_ID},
                    files={"document": f},
                )

mail.logout()
```

---

### 4️⃣ Run locally  
```bash
python download_attachment.py
```

---

## 🌍 Hosting Options  

| Platform           | Free Tier Limitations | IMAP Access | Notes |
|--------------------|----------------------|-------------|-------|
| **PythonAnywhere** | No external IMAP on free tier | ❌ | Requires paid plan |
| **Railway.app**    | 500 hrs/month free    | ✅ | Easy deploy |
| **Replit**         | Always-on requires paid | ✅ | Good for dev/test |
| **Render.com**     | 750 hrs/month free    | ✅ | Similar to Railway |
| **Google Colab**   | Session-based         | ✅ | Not 24/7, needs scheduling |
| **AWS Lambda**     | 1M req/month free     | ✅ | Serverless setup |

---

## 🔄 Alternate Design (Push Instead of Pull)  

If IMAP is blocked (e.g., PythonAnywhere free tier):
1. Create a Yahoo Mail **forwarding rule** to Gmail.  
2. Use the **Gmail API** to pull attachments (works on HTTPS, allowed everywhere).  
3. Send to Telegram as before.

---

## 🔐 Security Best Practices  
- **Never** commit `.env` to GitHub  
- Use **App Passwords** for Yahoo (not your main password)  
- Restrict bot permissions in Telegram BotFather  
- Rotate credentials regularly  

---

## 🛠 Dependencies  
`requirements.txt`  
```
python-dotenv
requests
```

Install with:
```bash
pip install -r requirements.txt
```

---

## 📬 Telegram Bot Setup  

1. Search **BotFather** on Telegram  
2. Use `/newbot` to create bot → get **BOT_TOKEN**  
3. Start your bot, send any message to it  
4. Visit:
```
https://api.telegram.org/bot<BOT_TOKEN>/getUpdates
```
Find `"chat":{"id":...}` — that’s your **CHAT_ID**  

---

## 📌 Future Improvements  
- Support multiple file types  
- Use Google Drive / Dropbox for storage instead of local  
- Add logging for sent files  
- Add scheduling (e.g., run every hour)  
