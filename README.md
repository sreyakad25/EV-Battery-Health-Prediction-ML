# Electric Vehicle Battery Health Prediction Using Machine Learning
### Cross-Platform Full-Stack Architecture (Django Web + Flutter Mobile + Cloud Firestore)

[![Python](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/)
[![Django](https://img.shields.io/badge/Django-5.0+-green.svg)](https://www.djangoproject.com/)
[![Flutter](https://img.shields.io/badge/Flutter-3.47+-02569B.svg)](https://flutter.dev/)
[![Firebase](https://img.shields.io/badge/Firebase-Auth%20%26%20Firestore-orange.svg)](https://firebase.google.com/)
[![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-1.4+-yellow.svg)](https://scikit-learn.org/)
[![License](https://img.shields.io/badge/License-Academic-lightgrey.svg)]()

---

## 1. Project Overview

**Electric Vehicle Battery Health Prediction Using Machine Learning** is an enterprise-grade, multi-platform system engineered to predict and track the **State of Health (SoH %)** and remaining usable life of EV lithium-ion battery packs. The platform models electrochemical degradation kinetics driven by cycling aging, ambient thermal conditions, internal cell impedance, calendar duration, and rapid DC charging intensity.

### Live Deployments
- **Vercel Web Production:** [https://electric-vehicle-battery-health-pre.vercel.app/](https://electric-vehicle-battery-health-pre.vercel.app/)
- **Render Web Production:** [https://electric-vehicle-battery-health.onrender.com/](https://electric-vehicle-battery-health.onrender.com/)
- **Flutter Mobile Application:** Android & iOS native client in `mobile_app/`

---

## 2. Key Architecture & Features

### 🔄 1. Unified Cross-Platform Synchronization
- **Single Source of Truth:** Google Firebase Authentication and Google Cloud Firestore serve as the unified persistent backend across Vercel, Render, and Flutter.
- **Hierarchical Path Isolation:** Predictions are saved to and streamed from:
  ```
  users/{firebase_uid}/predictions/{prediction_id}
  ```
- **Instant Cross-Device Sync:** An evaluation run on Vercel, Render, or the Flutter mobile client is instantly accessible across the other two platforms without database fragmentation.
- **Strict Data Isolation:** Built-in security rules and user-bound Firestore subcollections ensure users can only read and mutate their own predictions.

### 🤖 2. Machine Learning Pipeline
- **Ensemble & Baseline Regressors:** Evaluates **Random Forest Regressor** (top performer, R² ~0.972, MAE ~0.84%), **Linear Regression**, and **Decision Tree Regressor**.
- **Dual Prediction Workflows:**
  - **Standard Assessment:** Instant estimation using vehicle model (Tesla Model 3, BYD Atto 3, Tata Nexon EV, MG ZS EV, etc.), battery chemistry (NMC vs. LFP), and vehicle age.
  - **Advanced Laboratory Mode:** Comprehensive degradation estimation using battery capacity (kWh), internal impedance (mΩ), discharge C-rates, ambient temperature (°C), and DC fast charging ratios.
- **Actionable Diagnostics:** Output includes SoH %, health grading (Grade A through D), degradation risk category, estimated remaining usable kWh, and full driving range.

### 🔐 3. Authentication & Production UI
- **Firebase Authentication:** Seamless Email/Password authentication and one-tap **Google Sign-In** across Web and Mobile.
- **Password Reset (Forgot Password):** Integrated with Firebase Auth (`sendPasswordResetEmail`) to dispatch reset links directly to registered email addresses on both Web and Mobile.
- **Session Persistence:** Persistent authentication state with automatic token sync between Firebase and the Django backend session.
- **Production-Ready UI:** Clean, modern interface in dark teal (`#0F172A`) and electric emerald (`#00B774`). Dev Mode / test access bypasses are removed from the production UI.

---

## 3. Dataset Architecture

The machine learning models are trained on the **Electric Vehicle (EV) Battery Degradation & Charge** dataset published by **Bertnardo Mario Uskono** on Kaggle ([Dataset Link](https://www.kaggle.com/datasets/bertnardomariouskono/electric-vehicle-ev-battery-degradation-and-charge)):

- **10,000 synthetic records** modeling electrochemical degradation kinetics (cycle aging, Arrhenius thermal equation, and internal impedance growth).
- **Dataset Attributes (13 Columns):**
  1. `Vehicle_ID`: Unique vehicle identifier.
  2. `Car_Model`: EV make and model (`Tesla Model 3`, `BYD Atto 3`, `Nissan Leaf`, `Hyundai Kona`, etc.).
  3. `Battery_Type`: Chemistry formulation (`LFP` vs `NMC`).
  4. `Battery_Capacity_kWh`: Nominal battery capacity (40 to 100 kWh).
  5. `Vehicle_Age_Months`: Calendar age in operational months.
  6. `Total_Charging_Cycles`: Equivalent full charge/discharge cycles.
  7. `Avg_Temperature_C`: Thermal history in °C.
  8. `Fast_Charge_Ratio`: Proportion of DC rapid charging (0.0 to 1.0).
  9. `Avg_Discharge_Rate_C`: Mean driving discharge C-rate.
  10. `Driving_Style`: Aggression profile (`Conservative`, `Moderate`, `Aggressive`).
  11. `Internal_Resistance_Ohm`: Cell internal impedance in Ohms.
  12. `SoH_Percent`: Target variable 1 — State of Health percentage (55% to 100%).
  13. `Battery_Status`: Target variable 2 — Categorical classification (`Healthy` vs `Replace Required`).

---

## 4. Machine Learning Evaluation Benchmarks

Evaluation performed on held-out test records (1,500 samples, 70% / 15% / 15% train/val/test split):

| Model | MAE (Mean Absolute Error) | RMSE (Root Mean Sq Error) | R² Score | Performance Status |
| :--- | :---: | :---: | :---: | :---: |
| **Random Forest Regressor** | **0.845%** | **1.083%** | **0.9724** | **Top Performer** |
| **Linear Regression** | 0.848% | 1.069% | 0.9732 | Transparent Baseline |
| **Decision Tree Regressor** | 1.252% | 1.606% | 0.9394 | Non-linear Tree |

### Feature Importance Hierarchy
1. **Total Charging Cycles:** ~42.3% of model variance
2. **Fast Charge Ratio:** ~21.7% of degradation impact
3. **Internal Resistance:** ~14.8% impedance contribution
4. **Average Temperature:** ~11.2% thermal degradation
5. **Vehicle Age (Months):** ~6.1% calendar aging

---

## 5. System Architecture & Repository Structure

```
├── manage.py                         # Django management script
├── requirements.txt                  # Python dependencies
├── README.md                         # Project documentation
├── .env.example                      # Environment variables template
├── config/
│   ├── settings.py                   # Django settings & Firebase environment loader
│   ├── urls.py                       # Root URL routing
│   ├── wsgi.py
│   └── asgi.py
├── accounts/                         # User Authentication & Profile App
│   ├── models.py                     # UserProfile model
│   ├── views.py                      # Auth views, password reset, token sync APIs
│   ├── urls.py                       # Accounts URL routes
│   └── middleware.py                 # Security headers & session middleware
├── predictor/                        # ML Predictor & History App
│   ├── models.py                     # Local fallback models & schemas
│   ├── views.py                      # Web views (Dashboard, Predict, Results, History)
│   ├── firestore_service.py          # Server-side Cloud Firestore REST & SDK integration
│   ├── ml_service.py                 # Singleton model inferencing service
│   ├── train_models.py               # 70/15/15 pipeline training script
│   └── api/
│       ├── views.py                  # Cross-platform REST endpoints
│       └── urls.py
├── templates/                        # Responsive HTML5 Templates
│   ├── base.html                     # Main shell with sidebar & top navigation
│   ├── accounts/                     # Login, Register, Forgot Password, Profile
│   └── predictor/                    # Dashboard, Predict, Result, History, Dataset
├── static/                           # Web Assets
│   ├── css/style.css                 # Dark teal & electric emerald design tokens
│   ├── js/auth.js                    # Firebase Auth & Google Sign-In client logic
│   └── js/main.js                    # Interactive charts, SoH radial gauge & history filters
├── mobile_app/                       # Flutter Mobile Application
│   ├── pubspec.yaml                  # Flutter dependencies & assets config
│   ├── lib/
│   │   ├── main.dart                 # App initialization & route table
│   │   ├── firebase_options.dart     # Firebase configuration for mobile
│   │   ├── screens/
│   │   │   ├── splash_screen.dart    # Splash screen with auto session check
│   │   │   ├── welcome_screen.dart   # First-time onboarding screen
│   │   │   ├── login_screen.dart     # Email & Google login + Forgot Password
│   │   │   ├── register_screen.dart  # Account registration screen
│   │   │   ├── dashboard_screen.dart # Real-time metrics, quick actions & menu
│   │   │   ├── predict_screen.dart   # Standard & Advanced SoH prediction
│   │   │   ├── history_screen.dart   # Firestore streaming prediction history
│   │   │   └── profile_screen.dart   # Account details & security settings
│   │   ├── services/
│   │   │   ├── auth_service.dart     # Mobile Firebase Auth service
│   │   │   └── firestore_service.dart# Mobile Cloud Firestore CRUD & stream service
│   │   └── widgets/                  # Shared UI components (gauges, logo, waves)
│   └── test/
│       └── widget_test.dart          # 10 comprehensive widget & flow tests
└── artifacts/                        # Serialized ML models & metrics
    ├── model_random_forest.joblib
    ├── model_linear_regression.joblib
    ├── model_decision_tree.joblib
    └── metrics.json
```

---

## 6. Installation & Execution Guide

### Web Application (Django)

#### 1. Clone the repository & create virtual environment
```powershell
# PowerShell (Windows)
python -m venv venv
.\venv\Scripts\Activate.ps1

# Linux / macOS / Git Bash
python3 -m venv venv
source venv/bin/activate
```

#### 2. Install dependencies
```bash
pip install -r requirements.txt
```

#### 3. Configure environment variables
Copy `.env.example` to `.env` and fill in your Firebase and Django credentials:
```bash
cp .env.example .env
```
Key environment variables:
```ini
DJANGO_SECRET_KEY=your_secret_key
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1,.vercel.app,.onrender.com

# Firebase Authentication & Firestore Credentials
FIREBASE_PROJECT_ID=your-firebase-project-id
FIREBASE_CLIENT_EMAIL=your-service-account@project.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----"
FIREBASE_WEB_API_KEY=your-web-api-key
```

#### 4. Apply database migrations & verify
```bash
python manage.py migrate
python manage.py check
```

#### 5. Launch the local development server
```bash
python manage.py runserver
```
Visit **[http://127.0.0.1:8000/](http://127.0.0.1:8000/)** in your browser.

---

### Mobile Application (Flutter)

#### 1. Navigate to `mobile_app` & install dependencies
```bash
cd mobile_app
flutter pub get
```

#### 2. Run static analysis & widget tests
```bash
flutter analyze
flutter test
```

#### 3. Launch on emulator or physical device
```bash
# List connected devices
flutter devices

# Run on selected device
flutter run
```

---

## 7. Automated Testing Suite

The repository includes comprehensive automated test suites for both Web and Mobile components:

### Django Backend Tests
```bash
python manage.py test accounts predictor
```
- **13 test cases** verifying authentication views, session synchronization, prediction calculation endpoints, Firestore persistence adapters, and rate limiters.
- **Status:** `13/13 Passed (OK)`

### Flutter Mobile Tests
```bash
cd mobile_app
flutter test
```
- **10 automated tests** covering:
  - Brand identity splash loading
  - Unauthenticated cold start routing to `/login`
  - Authenticated session persistence directly to `/dashboard`
  - New user flow: Registration ➔ Login ➔ Dashboard
  - Existing user login with session restoration
  - Logout flow with state teardown
  - Absence of Dev Mode / test access elements in production UI
  - Real-time prediction input and SoH calculation validation
- **Status:** `10/10 Passed (All tests passed!)`

---

## 8. API Reference

### Authentication Endpoints (`/accounts/api/`)
| Endpoint | Method | Description |
| :--- | :---: | :--- |
| `/accounts/api/register/` | `POST` | Registers account in Firebase & syncs to Firestore |
| `/accounts/api/login/` | `POST` | Authenticates email/password against Firebase Auth |
| `/accounts/api/google-login/` | `POST` | Exchanges Google ID token for authenticated session |
| `/accounts/api/sync-firebase-token/` | `POST` | Synchronizes active Firebase ID token with Django session |
| `/accounts/api/firebase-config/` | `GET` | Supplies public client Firebase initialization parameters |

### Predictor Endpoints (`/predictor/api/`)
| Endpoint | Method | Description |
| :--- | :---: | :--- |
| `/predictor/api/predict/` | `POST` | Runs ML model inference on battery input parameters |
| `/predictor/api/save-prediction/` | `POST` | Saves prediction to `users/{uid}/predictions/{id}` |
| `/predictor/api/delete-prediction/<id>/` | `DELETE` | Deletes specific prediction from user's Firestore records |
| `/predictor/api/chart-data/` | `GET` | Fetches historical trend points for analytics charts |

---

## 9. Contributors & Academic Attribution
- **Author / Developer:** Vishak P
- **Research Topic:** Electric Vehicle Battery Health Prediction Using Machine Learning
- **Dataset Attribution:** Bertnardo Mario Uskono (Kaggle EV Battery Degradation & Charge)
- **License:** Academic & Educational Research Use
