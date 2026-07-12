# 🌿 KisanAI — Smart Agricultural Assistant for Indian Farmers

KisanAI is an AI-powered web platform built to help Indian farmers diagnose crop diseases and check irrigation water quality — instantly, in their own language. It combines a trained deep learning model, a conversational Telegram bot, and a modern web dashboard into one connected system.

This project was built as a final year Data Science capstone, with a dual goal: solve a real problem for Indian farmers, and demonstrate applied machine learning, full-stack engineering, and automation at a production-grade level.

---

## What KisanAI Does

**🍃 Leaf Disease Detector**
Upload a photo of a crop leaf — through the website or directly via Telegram — and KisanAI's CNN model identifies the disease in seconds, with a confidence score and treatment advice delivered in both English and Marathi.

**💧 Water Quality Checker**
Enter irrigation water parameters (pH, Dissolved Oxygen, BOD, Turbidity, etc.) and get an instant safety classification, benchmarked against BIS IS:10500 and CPCB standards.

**🔐 Account System**
Sign in with Google, GitHub, or Email/Password (Firebase Authentication). Tools are locked until sign-in, keeping usage tied to real accounts.

**📧 Automated Communication**
New users receive an automatic welcome email and WhatsApp message. Every diagnosis is also emailed as a permanent record, so farmers can track their crop's health history over time.

**🤖 Telegram Bot**
Farmers with no interest in visiting a website can simply message a photo to the KisanAI Telegram bot and receive a diagnosis in Marathi — no app download, no login required.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript (single-file), Three.js, GSAP, Lenis |
| Authentication | Firebase Authentication (Google, GitHub, Email/Password) |
| ML Model | TensorFlow / Keras — MobileNetV2 (transfer learning) |
| ML Backend | Python, Flask |
| Bot | python-telegram-bot |
| Email Automation | EmailJS |
| WhatsApp Automation | Twilio WhatsApp API |
| Dataset | PlantVillage (Kaggle) |

---

## Project Structure

```
KisanAI/
│
├── index.html                  ← Full frontend (website)
├── .gitignore                  ← Excludes secrets from Git
│
├── leafenv/                    ← Python virtual environment (ML backend)
├── venv/                       ← Python virtual environment (auth/whatsapp backend)
│
└── backend/
    ├── .env                    ← Secrets (NEVER pushed to GitHub)
    ├── requirements.txt
    │
    ├── data/
    │   └── raw/                ← PlantVillage dataset (5 classes used)
    │       ├── Tomato_healthy/
    │       ├── Tomato_Late_blight/
    │       ├── Potato___healthy/
    │       ├── Potato___Late_blight/
    │       └── Pepper__bell___healthy/
    │
    ├── models/
    │   ├── leaf_model.h5       ← Trained CNN model
    │   └── class_names.json    ← Class index mapping
    │
    ├── train.py                 ← Trains the CNN model
    ├── predict.py               ← Loads model, runs predictions
    ├── treatment.py             ← Disease → treatment info (EN + Marathi)
    ├── app.py                   ← Flask API (serves /predict route)
    ├── telegram_bot.py          ← Telegram bot (photo → diagnosis)
    └── whatsapp.py               ← Twilio WhatsApp automation service
```

---

## Prerequisites

Before you begin, make sure you have:

- **Python 3.11 or 3.12** installed (⚠️ avoid 3.13/3.14 — TensorFlow does not support them yet)
- **Git** installed
- A **Firebase** account (free) — [console.firebase.google.com](https://console.firebase.google.com)
- An **EmailJS** account (free) — [emailjs.com](https://emailjs.com)
- A **Twilio** account (free trial) — [twilio.com/try-twilio](https://twilio.com/try-twilio)
- A **Telegram** account, to create a bot via [@BotFather](https://t.me/BotFather)
- A **Kaggle** account, to download the dataset

---

## Setup Instructions

Follow these steps in order. Commands are written for **macOS/Linux** (zsh/bash). Windows users should use Git Bash or WSL for the same commands.

### Step 1 — Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/KisanAI.git
cd KisanAI
```

### Step 2 — Set up the ML backend environment

The leaf disease model needs its own isolated environment (TensorFlow has heavy dependencies).

```bash
python3 -m venv leafenv
source leafenv/bin/activate
```

Your terminal prompt should now show `(leafenv)`.

```bash
cd backend
pip install -r requirements.txt
```

### Step 3 — Download the dataset

1. Go to [kaggle.com/datasets/emmarex/plantdisease](https://www.kaggle.com/datasets/emmarex/plantdisease)
2. Download and unzip the dataset
3. Place the extracted folders inside `backend/data/raw/`

You only need these 5 folders for training (the rest can stay unused):
```
Tomato_healthy
Tomato_Late_blight
Potato___healthy
Potato___Late_blight
Pepper__bell___healthy
```

### Step 4 — Train the leaf disease model

```bash
python train.py
```

This takes **30–90 minutes** depending on your machine. When it finishes, confirm these two files exist:

```bash
ls models/
# Should show: leaf_model.h5   class_names.json
```

### Step 5 — Set up your `.env` file (secrets)

Create a `.env` file inside `backend/`:

```bash
touch .env
```

Open it and add the following (see [Getting Your API Keys](#getting-your-api-keys) below for how to obtain each value):

```env
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TELEGRAM_BOT_TOKEN=your_telegram_bot_token
```

⚠️ This file is already listed in `.gitignore` — it will never be pushed to GitHub. Never share these values publicly.

### Step 6 — Run the ML API server

From inside `backend/` with `leafenv` still active:

```bash
python app.py
```

You should see:
```
🌿 KisanAI ML API — live on port 5000
POST http://localhost:5000/predict
GET  http://localhost:5000/health
```

Leave this terminal running.

### Step 7 — Run the Telegram bot (separate terminal)

Open a **new terminal window**, activate `leafenv` again, and run:

```bash
cd KisanAI/backend
source ../leafenv/bin/activate
python telegram_bot.py
```

Test it by messaging your bot on Telegram with a leaf photo.

### Step 8 — Set up the auth/automation environment (separate from ML)

Open a **third terminal window** (project root, not backend):

```bash
cd KisanAI
python3 -m venv venv
source venv/bin/activate
cd backend
pip install flask flask-cors twilio python-dotenv
```

### Step 9 — Run the WhatsApp automation service

```bash
python whatsapp.py
```

You should see:
```
🌿 KisanAI WhatsApp service — live on port 5001
```

### Step 10 — Serve the frontend locally

Open a **fourth terminal window** (project root):

```bash
cd KisanAI
python3 -m http.server 8000
```

Open your browser to:
```
http://localhost:8000
```

⚠️ Use `localhost`, not `127.0.0.1` — Firebase Google Sign-In only trusts `localhost` by default.

---

## Getting Your API Keys

### Firebase (Authentication)
1. Go to [console.firebase.google.com](https://console.firebase.google.com) → Create Project
2. Build → Authentication → Get Started → Enable Google, GitHub, Email/Password
3. Project Settings → General → Your apps → Web app → copy the config object
4. Paste the 6 values into the Firebase config block in `index.html`

### EmailJS (Email automation)
1. Sign up at [emailjs.com](https://emailjs.com)
2. Add an Email Service (connect your Gmail) → copy the **Service ID**
3. Create two templates (Welcome, Diagnosis Report) → copy each **Template ID**
4. Account → General → copy your **Public Key**
5. Paste all 4 values into the EmailJS config block in `index.html`

### Twilio (WhatsApp automation)
1. Sign up at [twilio.com/try-twilio](https://twilio.com/try-twilio)
2. Messaging → Try it out → Send a WhatsApp message → note the sandbox number and join code
3. From your own WhatsApp, send the join code to the sandbox number
4. Console home page → copy **Account SID** and **Auth Token** → add to `backend/.env`

### Telegram Bot
1. Open Telegram → search **@BotFather** → send `/newbot`
2. Give it a name and a username ending in `bot`
3. Copy the token BotFather gives you → add to `backend/.env`

---

## Running the Full System (Quick Reference)

You need **4 terminals running simultaneously**:

| Terminal | Environment | Command | Port |
|---|---|---|---|
| 1 | `leafenv` | `python backend/app.py` | 5000 |
| 2 | `leafenv` | `python backend/telegram_bot.py` | — |
| 3 | `venv` | `python backend/whatsapp.py` | 5001 |
| 4 | any | `python3 -m http.server 8000` | 8000 |

Then open: `http://localhost:8000`

---

## Testing the System

**Test the ML model directly:**
```bash
cd backend
python3 -c "from predict import predict_disease; print(predict_disease('data/raw/Tomato_Late_blight/[filename].JPG'))"
```

**Test the Flask API:**
```bash
curl -X POST http://localhost:5000/predict \
  -F "image=@backend/data/raw/Tomato_Late_blight/[filename].JPG"
```

**Test the Telegram bot:**
Send `/start` then a leaf photo to your bot on Telegram.

**Test the website:**
Register an account, upload a leaf photo on the Detect Disease page, confirm a real diagnosis appears and an email arrives.

---

## Known Limitations

- The current model recognizes only 5 classes across 3 crops (Tomato, Potato, Pepper). Images outside this scope will still return a confident-sounding but incorrect prediction — a known limitation of softmax-based classifiers with no "unknown" category.
- The Twilio WhatsApp integration uses the free Sandbox, which only messages numbers that have manually joined it. A verified WhatsApp Business number would be required for public deployment.
- Email deliverability may land in spam on first send from a new sending address; this improves once recipients mark a message "Not spam" once.
- `whatsapp.py` and `telegram_bot.py` must be run locally — they are not deployed to Vercel, which only serves the static frontend.

---

## Future Improvements

- [ ] Grad-CAM explainability overlay on disease predictions
- [ ] Crowdsourced disease outbreak heatmap across India
- [ ] Weather-integrated disease risk forecasting
- [ ] Cinematic India water quality map with time slider
- [ ] Offline-first PWA support for low-connectivity areas
- [ ] KisanBot — conversational AI agronomist in Marathi

---

## Authors

Built by Abhishek Sali as a final year Data Science capstone project.

> "AI that speaks the farmer's language."
