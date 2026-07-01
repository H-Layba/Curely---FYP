

# Curely — Your Health Companion

> A Flutter-based mobile health companion application for patients with chronic diseases including diabetes, hypertension, and heart disease.

![Flutter](https://img.shields.io/badge/Flutter-3.0-blue) ![Firebase](https://img.shields.io/badge/Firebase-Firestore-orange) ![Groq](https://img.shields.io/badge/AI-Groq%20LLaMA-purple) ![Render](https://img.shields.io/badge/Backend-Render-green) ![Cloudinary](https://img.shields.io/badge/Storage-Cloudinary-blue)

---

## Table of Contents
- [About](#about)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Backend](#backend)
- [AI & ML Integration](#ai--ml-integration)
- [Firebase Structure](#firebase-structure)
- [Running Tests](#running-tests)
- [Known Limitations](#known-limitations)

---

## About

Curely addresses a real-world problem — chronic disease patients struggle to consistently track their health, manage medications, understand lab reports, and access dietary guidance suited to their conditions. Curely brings all of this into one mobile application powered by AI and cloud services.

---

## Features

- 🏥 **Health Dashboard** — Track blood pressure, blood glucose, temperature, water intake, and sleep with daily/weekly/monthly trend charts
- 📝 **Daily Health Entry** — One-entry-per-day enforced at database level using date as Firestore document ID
- 🥗 **AI Diet Module** — Describe meals to get nutrition breakdown via Groq AI, then receive personalised Pakistani meal suggestions from a custom ML model
- 📄 **Medical Reports** — Upload prescriptions and lab reports, extract text via Gemini OCR, organise into folders, export as PDF
- 💊 **Medication Reminders** — Auto-schedule notifications from OCR-extracted medicine frequency data
- 🤖 **AI Chatbot** — Medical Q&A assistant restricted to health topics only using Groq LLaMA 3.3 70B
- 🚨 **Emergency Page** — One-tap 1122 emergency call with patient critical info (blood type, conditions, allergies) displayed
- 🔔 **Smart Notifications** — Timezone-aware scheduling, survives device reboot via Firestore restoration
- 👤 **Patient Profile** — BMI auto-calculation, chronic disease and allergy tracking, Google and email auth

---

## Tech Stack

| Layer | Technology |
|---|---|
| Mobile Frontend | Flutter (Dart) |
| State Management | Provider + setState |
| Authentication | Firebase Auth (Email + Google) |
| Database | Cloud Firestore |
| File Storage | Cloudinary + Firebase Storage |
| AI Chatbot & Diet | Groq API — LLaMA 3.3 70B |
| OCR | Google Gemini Vision (via Node.js backend) |
| Diet ML Model | Custom Python model hosted on Render |
| Notifications | flutter_local_notifications + timezone |
| Charts | fl_chart |
| PDF Export | pdf + share_plus |
| Backend | Node.js + Express on Render |

---

## Architecture

```
Flutter App
    │
    ├── Firebase ──────────────── Auth, Firestore, Storage
    │
    ├── Groq API ──────────────── LLaMA 3.3 70B
    │       ├── Chatbot (health Q&A)
    │       ├── Diet Call 1 (meal nutrition extraction)
    │       └── Diet Call 2 (personalised meal suggestions)
    │
    ├── Render — Node.js Backend
    │       ├── POST /extract ──── Google Gemini Vision (OCR)
    │       └── POST /delete-images ── Cloudinary Admin API
    │
    ├── Cloudinary ─────────────── Report Image Storage
    │
    └── Render — Python Backend
            └── POST /predict ──── Diet ML Model
```

---

## Project Structure

```
curely/
├── lib/
│   ├── main.dart                        # Entry point, splash screen, routing logic
│   ├── main_scaffold.dart               # Bottom nav shell, drawer, notification restore, logout
│   │
│   ├── auth/
│   │   ├── login_page.dart              # Email + Google Sign-In
│   │   ├── signup_page.dart             # Sign up with live password validation
│   │   └── forgot_password_page.dart    # Firebase password reset email
│   │
│   ├── patient/
│   │   ├── patient_info_page.dart       # One-time profile setup (BMI, conditions, allergies)
│   │   └── daily_health_entry_page.dart # Daily vitals form with range validation
│   │
│   ├── dashboard/
│   │   └── dashboard.dart               # Metrics display, trend charts, PDF export
│   │
│   ├── diet_model/
│   │   └── diet.dart                    # AI meal logging + ML diet prediction pipeline
│   │
│   ├── reports/
│   │   └── report.dart                  # Folder management, OCR upload, med reminders, PDF
│   │
│   ├── chatbot_model/
│   │   └── chatbot.dart                 # Groq-powered medical chatbot
│   │
│   ├── notifications/
│   │   └── notification_service.dart    # Singleton service, timezone fix, schedule/cancel/restore
│   │
│   ├── emergency/
│   │   └── emergency_page.dart          # Emergency call, contacts, critical medical info
│   │
│   ├── settings/
│   │   └── settings_page.dart           # Notification toggles, account deletion, profile edit
│   │
│   └── utils/
│       └── cloudinary_cleanup.dart      # Delete images from Cloudinary on report/account delete
│
├── test/
│   ├── dashboard_logic_test.dart        # Unit tests for BP, glucose, temperature status functions
│   └── notification_service_test.dart   # Unit tests for scheduling and frequency parsing logic
│
├── backend/
│   ├── index.js                         # Node.js Express server (OCR + Cloudinary delete routes)
│   ├── package.json                     # Node dependencies
│   └── package-lock.json                # Locked dependency versions
│
└── pubspec.yaml                         # Flutter dependencies
```

---

## Getting Started

### Prerequisites

- Flutter SDK >= 3.0.0
- Dart >= 3.0.0
- Firebase project with Firestore, Auth, and Storage enabled
- Groq API key — [console.groq.com](https://console.groq.com)
- Google Gemini API key — [aistudio.google.com](https://aistudio.google.com)
- Cloudinary account with an unsigned upload preset

### Installation

1. Clone the repository:
```bash
git clone https://github.com/H-Layba/curely.git
cd curely
```

2. Install Flutter dependencies:
```bash
flutter pub get
```

3. Add Firebase config files:
   - `android/app/google-services.json` — from Firebase console
   - `ios/Runner/GoogleService-Info.plist` — from Firebase console

4. Run the app:
```bash
flutter run
```

---

## Environment Variables

Set these in your Render dashboard for the Node.js backend:

| Variable | Description |
|---|---|
| `CLOUDINARY_CLOUD_NAME` | Your Cloudinary cloud name |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |
| `GEMINI_API_KEY` | Google Gemini API key |

---

## Backend

The Node.js backend is deployed on Render and exposes two endpoints:

### `POST /extract` — Medical Report OCR

Accepts a report image as `multipart/form-data`. Passes it to Google Gemini Vision and returns structured JSON:

```json
{
  "doctor_name": "Dr. Ahmed",
  "hospital_or_clinic": "City Hospital",
  "patient_name": "Ali Khan",
  "date": "2026-06-01",
  "medicines": [
    {
      "medicine_name": "Metformin",
      "dosage": "500mg",
      "frequency": "twice",
      "duration": "30 days",
      "instructions": "after meals"
    }
  ]
}
```

### `POST /delete-images` — Cloudinary Bulk Delete

Accepts an array of Cloudinary public IDs and deletes them. Called when reports or accounts are deleted to prevent orphaned images.

```json
{ "publicIds": ["curely/reports/abc123", "curely/reports/xyz456"] }
```

---

## AI & ML Integration

### Groq API — LLaMA 3.3 70B

Used in two modules:

**Chatbot:**
- System prompt restricts responses to health, symptoms, diseases, diet, and wellness only
- If asked anything unrelated, responds: *"I can only help with medical-related questions"*

**Diet Module — Two separate Groq calls:**
- Call 1: User describes a meal → Groq extracts `{calories, protein, fat, carbs}` as JSON
- Call 2: After ML prediction → Groq generates specific Pakistani meal suggestions based on targets and patient conditions (diabetic-friendly, low-sodium for hypertension, allergy-aware)

### Google Gemini Vision — OCR

- Accessed via the Node.js backend (proxy pattern — API key never exposed to Flutter)
- Accepts prescription/lab report images
- Returns structured medicine data including name, dosage, and frequency

### Custom Diet ML Model

- Trained regression model hosted on Render
- Input: age, weight, height, BMI, gender, chronic conditions, allergies, smoking status, current vitals
- Output: recommended daily calories, protein, fat, and carbohydrate targets
- Endpoint: `https://diet-recommendation-x0yg.onrender.com/predict`

---

## Firebase Structure

```
patients/{uid}
    ├── name, age, weight, height, bmi, gender
    ├── bloodType, chronicDiseases[], allergies[]
    ├── smoking, email
    ├── reminderEnabled, reminderTime
    └── reminderPromptShown

patients/{uid}/settings/preferences
    └── medicationReminder (bool)

patients/{uid}/daily_logs/{YYYY-MM-DD}
    ├── bp_systolic, bp_diastolic
    ├── blood_glucose
    ├── temperature
    ├── water_intake
    └── sleep_hours

users/{uid}/folders/{folderId}
    └── name

users/{uid}/folders/{folderId}/reports/{reportId}
    ├── imageUrl        (Cloudinary public URL)
    ├── extractedText   (cleaned OCR output)
    ├── label
    └── timestamp

users/{uid}/med_reminders/{reportId}
    └── {
          medName: {
            enabled: true,
            times: [{ hour: 8, minute: 0 }]
          }
        }
```

---

## Running Tests

```bash
flutter test
```

### `dashboard_logic_test.dart`
Tests metric status boundary conditions without requiring Firebase or widgets:

| Function | Test Cases |
|---|---|
| `bpStatus(sys, dia)` | Normal, High (>140/90), Low (<90/60) |
| `glucoseStatus(g)` | Normal, High (>140), Low (<70) |
| `tempStatus(t)` | Normal, Fever (≥38°C), Low (<35°C) |
| `parseValue(s)` | Valid number strings, empty, null |

### `notification_service_test.dart`
Tests scheduling logic:

| Function | Test Cases |
|---|---|
| `parseFrequency(s)` | "once" → [8am], "twice" → [8am,8pm], "thrice" → [8am,2pm,8pm] |
| `makeBaseId(id, name)` | Same input always returns same ID |
| `_nextInstanceOfTime()` | Past time returns tomorrow, future time returns today |

---

Application APK 

link: https://drive.google.com/file/d/1-d5IsHUvj4rwKUyadUGYYHUkbEx7E6tc/view?usp=drivesdk
