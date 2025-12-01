# 🧹 Smart Mailbox Sanitizer — AI-Driven Gmail Management System
## 🎯 Capstone / Major Project Track  
**AI & Automation Systems**  
*(Also suitable for Cloud Computing / Machine Learning / Software Development)*  

---

## 📌 Problem Statement  
Email inboxes today are overloaded with:  
- Spam messages  
- Promotional ads  
- Endless newsletters  
- Social notifications  

Manually sorting them is slow, repetitive, and highly inefficient.  
Hence, an **intelligent automated system** is required that can clean and organize the mailbox without manual effort.

---

## 🎯 Project Objectives  
- Categorize emails using AI (Spam / Newsletter / Promotions / Social / Important)  
- Automatically delete spam & irrelevant promotions  
- Auto-unsubscribe from unwanted newsletters  
- Archive low-priority social emails  
- Securely keep important emails untouched  
- Execute automatically every day using cloud-based scheduling  

---

## 💡 Proposed Solution  
**Smart Mailbox Sanitizer** is an AI-powered Gmail automation tool built using:  

- **Google Gemini API** → Smart email classification  
- **Gmail API** → Read, organize, delete & modify mail  
- **Kaggle Scheduler** → Daily automated execution  

The system fetches emails, classifies them using AI models, applies actions automatically, and maintains a clean inbox continuously.

---

## ⚠️ IMPORTANT — RUN KARNE SE PEHLE ZARURI SETUP

---

### ✅ 1. Google Cloud Project Setup  
1. Open: https://console.cloud.google.com  
2. New Project create karo → Project Name: **SmartMailboxSanitizer**  
3. Gmail API enable karo:  
   - Go to **API & Services → Library**  
   - Search **Gmail API**  
   - Click **Enable**

---

### ✅ 2. OAuth Consent Screen Configuration (VERY IMPORTANT)  
1. Go to **API & Services → OAuth Consent Screen**  
2. User Type → **External**  
3. App Name → **Smart Mailbox Sanitizer**  
4. Developer Contact Email → Apna Gmail  
5. Scroll down → **Test Users** section kholna  
6. **Add User** → Apna Gmail add karo  
7. Save

⚠️ Agar aap apne Gmail ko **Test User** me add nahi karoge, to OAuth permission screen error dega.

---

### ✅ 3. OAuth Client ID (credentials.json) Banana  
1. Go to **API & Services → Credentials**  
2. Click **Create Credentials → OAuth Client ID**  
3. Application Type → **Desktop App**  
4. Generate hone ke baad → **credentials.json** download karo  
5. Kaggle Notebook me → **Add → Upload File** ka use karke  
   `credentials.json` upload kar dena

---

### ✅ Notes  
- Kaggle environment har session ke baad reset hota hai, isliye `credentials.json` hamesha upload rehna chahiye.  
- Credentials ke bina Gmail API connect nahi hoga.

# ⚠️ Manual Setup (Required Before Running)

Before using the Smart Gmail Inbox Automation, you need to complete a few setup steps:

---

### 1️⃣ Google Cloud Setup
- Create a **new Google Cloud Project**.
- **Enable the Gmail API** for your project.

---

### 2️⃣ OAuth Consent Screen
- **App Name:** Inbox Cleaner  
- **User Type:** External  
- Add your Gmail account under **Test Users** (⚠️ Important!).

---

### 3️⃣ OAuth Credentials
- Create a **Desktop App OAuth Client**.
- **Download `credentials.json`**.
- **Upload `credentials.json`** to your Kaggle environment.

---

Once these steps are done, you’re ready to run the automation! 🚀

```python
import os
import json
import urllib.parse
import requests
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build   # ← FIX
```

```python
import os
print(os.listdir('/kaggle/input/'))
```

  OUTPUT GENERATED ✅ 

```python
# STEP 1 — IMPORTS & GLOBAL VARIABLES
# ================================
import os
import json
import urllib.parse
import requests
from google.oauth2.credentials import Credentials
from googleapiclient.discovery import build   # Used to create Gmail API service object
from oauth2client import client as credentials

# Define global constants
TOKEN_PATH = "token.json"                # Token will be saved here after OAuth
CREDENTIALS_PATH = "/kaggle/input/mailbox-sanitizer/credentials.json"   # Upload your credentials.json here
SCOPES = ["https://www.googleapis.com/auth/gmail.modify"]  # Gmail API scope


# ================================
# STEP 2 — MANUAL OAUTH FUNCTION
# ================================
def get_gmail_service():

    creds = None   # Placeholder for credentials

    # STEP 1 — Load existing OAuth token if available
    if os.path.exists(TOKEN_PATH):
        creds = Credentials.from_authorized_user_file(TOKEN_PATH, SCOPES)

    # If no valid credentials found → perform manual OAuth
    if not creds or not creds.valid:

        # STEP 2 — Load OAuth client details from credentials.json
        
        with open(CREDENTIALS_PATH, "r") as f:
            data = json.load(f)["installed"]

        client_id = data["client_id"]
        redirect_uri = data["redirect_uris"][0]   # Must match Google Cloud Console
        token_uri = data["token_uri"]
        client_secret = data["client_secret"]

        # STEP 3 — Manually build Google OAuth URL
        params = {
            "response_type": "code",
            "client_id": client_id,
            "redirect_uri": redirect_uri,
            "scope": " ".join(SCOPES),
            "access_type": "offline",
            "prompt": "consent"
        }

        auth_url = "https://accounts.google.com/o/oauth2/v2/auth?" + urllib.parse.urlencode(params)

        # Show URL to user
        print("🔗 Open this URL:")
        print(auth_url)

        print("\nAfter login Google redirects to: http://localhost/?code=XXXX")
        code = input("\nPaste authorization code: ").strip()   # User pastes code from redirect URL

        # STEP 4 — Exchange authorization code for access token
        token_data = {
            "code": code,
            "client_id": client_id,
            "client_secret": client_secret,
            "redirect_uri": redirect_uri,
            "grant_type": "authorization_code"
        }

        resp = requests.post(token_uri, data=token_data)
        token_json = resp.json()

        # OAuth failed check
        if "access_token" not in token_json:
            print("❌ Token exchange failed:", token_json)
            raise Exception("OAuth Failed")

        # STEP 5 — Create Credentials object manually
        creds = Credentials(
            token_json["access_token"],
            refresh_token=token_json.get("refresh_token"),
            token_uri=token_uri,
            client_id=client_id,
            client_secret=client_secret,
            scopes=SCOPES
        )

        # Save token for future runs
        with open(TOKEN_PATH, "w") as f:
            f.write(creds.to_json())

        print("✅ OAuth Success!")

    # STEP 6 — Return Gmail API service using authenticated creds
    return build("gmail", "v1", credentials=creds)


# ================================
# STEP 3 — INITIALIZE SERVICE
# ================================
service = get_gmail_service()
print("📬 Gmail service initialized successfully!")
```

  OUTPUT GENERATED ✅ AND FOLLOW INSTRUCTION ON OUTPUT TO GET A CODE

```python
def get_messages(service, query="", max_results=1000):
    results = service.users().messages().list(
        userId="me",
        q=query,
        maxResults=max_results
    ).execute()

    return results.get("messages", [])

emails = get_messages(service, "in:draft", 5)

emails
```

  OUTPUT GENERATED ✅ 

```python
import base64

def get_email_text(service, msg_id):
    msg = service.users().messages().get(
        userId="me",
        id=msg_id,
        format="full"
    ).execute()

    payload = msg.get("payload", {})
    
    if "parts" in payload:
        for part in payload["parts"]:
            if part["mimeType"] == "text/plain":
                data = part["body"]["data"]
                return base64.urlsafe_b64decode(data).decode()

    if "body" in payload and "data" in payload["body"]:
        data = payload["body"]["data"]
        return base64.urlsafe_b64decode(data).decode()

    return "(no text found)"

sample_text = get_email_text(service, emails[1]["id"])
print(sample_text)
```
  OUTPUT GENERATED ✅ 

```python
from kaggle_secrets import UserSecretsClient
user_secrets = UserSecretsClient()
GEMINI_API_KEY = user_secrets.get_secret("GOOGLE_API_KEY")
```

```python
import google.generativeai as genai
genai.configure(api_key=GEMINI_API_KEY)
model = genai.GenerativeModel("models/gemini-2.5-flash")
```

```python
from kaggle_secrets import UserSecretsClient
import google.generativeai as genai

# Load your Gemini API key from Kaggle
user_secrets = UserSecretsClient()
GEMINI_API_KEY = user_secrets.get_secret("GOOGLE_API_KEY")

# Configure Gemini client
genai.configure(api_key=GEMINI_API_KEY)

# Use the correct supported model
model = genai.GenerativeModel("models/gemini-2.5-flash")

def classify_email(text):
    prompt = f"""
    Classify this email into exactly one of these categories:
    - spam
    - promotion
    - important
    - update
    - social

    Email content:
    \"\"\"{text}\"\"\"

    Respond with only ONE WORD from the list above.
    """
    response = model.generate_content(prompt)
    return response.text.strip().lower()

# Test
sample_text = "."
print(classify_email(sample_text))
```

  OUTPUT GENERATED ✅ 

```python
def delete_email(service, msg_id):
    service.users().messages().trash(userId="me", id=msg_id).execute()
```

```python
def archive_email(service, msg_id):
    service.users().messages().modify(
        userId="me",
        id=msg_id,
        body={"removeLabelIds": ["INBOX"]}
    ).execute()
```

```python
def extract_unsubscribe_link(text):
    prompt = f"""
    Extract the unsubscribe link from this email.
    If none, respond with 'none'.
    
    Email:
    {text}
    """

    link = model.generate_content(prompt).text.strip()
    return link
```

```python
import requests

def unsubscribe(link):
    if link.startswith("http"):
        try:
            requests.get(link, timeout=5)
        except:
            pass
```

```python
def clean_inbox():
    emails = get_messages(service, "in:spam", max_results=10)

    for e in emails:
        msg_id = e["id"]
        text = get_email_text(service, msg_id)
        category = classify_email(text)

        print("Email:", msg_id, "| Category:", category)

        if category == "spam":
            delete_email(service, msg_id)

        elif category == "newsletter":
            link = extract_unsubscribe_link(text)
            if link != "none":
                unsubscribe(link)
            archive_email(service, msg_id)

        elif category == "promotion":
            delete_email(service, msg_id)

        elif category == "social":
            archive_email(service, msg_id)
```

# 🧪 Smart Gmail Inbox Automation — Features

Our AI-powered Gmail automation system takes the hassle out of managing your inbox. Here's what it can do:

---

### 🔴 Spam Management
- Automatically detects spam emails.
- Deletes unwanted messages so you never see them again.  

### 🟡 Newsletter Automation
- Extracts **unsubscribe links** using Gemini.
- Auto-clicks unsubscribe links to reduce clutter.
- Archives newsletters for easy reference.

### 🟣 Social Email Handling
- Automatically archives social notifications.
- Keeps your inbox focused on what matters most.

### 🟢 Important Email Protection
- Protects important emails from being deleted or archived.
- Ensures you never miss crucial messages.

### 🔄 Fully Automated
- Runs **daily** using the Kaggle Scheduler.
- **No user interaction required** — set it up once and forget it!

---

Transform your Gmail into a **clean, organized, and stress-free inbox** effortlessly! ✨



