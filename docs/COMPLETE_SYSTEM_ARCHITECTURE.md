# Complete System Architecture

## System Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    MEDICAL DIAGNOSIS SYSTEM                         │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    FRONTEND (Browser)                        │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐   │  │
│  │  │ Login Page  │  │ Mode Select  │  │ Diagnosis Forms  │   │  │
│  │  └─────────────┘  └──────────────┘  └──────────────────┘   │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────────┐   │  │
│  │  │ Quiz Page   │  │ Results Page │  │ History/Progress │   │  │
│  │  └─────────────┘  └──────────────┘  └──────────────────┘   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              ↕                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                  BACKEND (Flask Server)                      │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │ Routes & Controllers                                │    │  │
│  │  │ ├─ /login (GET/POST)                               │    │  │
│  │  │ ├─ /logout (GET)                                   │    │  │
│  │  │ ├─ /diagnosis/* (GET/POST)                         │    │  │
│  │  │ ├─ /training/* (GET/POST)                          │    │  │
│  │  │ └─ /mode_selection (GET)                           │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │ Business Logic                                      │    │  │
│  │  │ ├─ Authentication (login_user, add_user)           │    │  │
│  │  │ ├─ Data Processing (encoding, scaling)             │    │  │
│  │  │ ├─ ML Predictions (model.predict)                  │    │  │
│  │  │ └─ Data Persistence (save_patient_record)          │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  │  ┌─────────────────────────────────────────────────────┐    │  │
│  │  │ Session Management                                  │    │  │
│  │  │ ├─ user_id                                          │    │  │
│  │  │ ├─ user_name                                        │    │  │
│  │  │ ├─ user_email                                       │    │  │
│  │  │ └─ quiz_data                                        │    │  │
│  │  └─────────────────────────────────────────────────────┘    │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                              ↕                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    DATA LAYER                                │  │
│  │  ┌──────────────────────────────────────────────────────┐   │  │
│  │  │ SQLite Database (medical_ai.db)                     │   │  │
│  │  │ ├─ users table                                      │   │  │
│  │  │ │  ├─ id (PRIMARY KEY)                              │   │  │
│  │  │ │  ├─ name                                          │   │  │
│  │  │ │  ├─ email (UNIQUE)                                │   │  │
│  │  │ │  └─ password (hashed)                             │   │  │
│  │  │ └─ patient_records table                            │   │  │
│  │  │    ├─ id (PRIMARY KEY)                              │   │  │
│  │  │    ├─ user_id (FOREIGN KEY)                         │   │  │
│  │  │    ├─ diagnosis_result                              │   │  │
│  │  │    ├─ confidence_score                              │   │  │
│  │  │    └─ created_at (TIMESTAMP)                        │   │  │
│  │  └──────────────────────────────────────────────────────┘   │  │
│  │  ┌──────────────────────────────────────────────────────┐   │  │
│  │  │ ML Models (Pickle Files)                            │   │  │
│  │  │ ├─ diabetes_model.pkl                               │   │  │
│  │  │ ├─ bp_awareness_model.pkl                           │   │  │
│  │  │ └─ lungcancer_rf_model.pkl                          │   │  │
│  │  └──────────────────────────────────────────────────────┘   │  │
│  │  ┌──────────────────────────────────────────────────────┐   │  │
│  │  │ CSV Data Files                                      │   │  │
│  │  │ ├─ disease_mcq_dataset_500.csv (Quiz questions)     │   │  │
│  │  │ ├─ diabetes.csv (Training data)                     │   │  │
│  │  │ └─ bp_awareness_dataset_6000.csv (Training data)    │   │  │
│  │  └──────────────────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────────────────┘  │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Architecture

### 1. Authentication Flow

```
USER INPUT (Email, Password)
        ↓
    [HTML Form]
        ↓
    POST /login
        ↓
    [Flask Route]
        ├─ Extract email & password
        ├─ Call login_user(email, password)
        │   ├─ Query: SELECT password FROM users WHERE email=?
        │   ├─ Verify: bcrypt.checkpw(input, stored_hash)
        │   └─ Return: (success, user_data)
        ├─ If success:
        │   ├─ Create session
        │   ├─ Store user_id, user_name, user_email
        │   └─ Redirect to mode_selection
        └─ If failed:
            └─ Show error message
```

### 2. Diagnosis Flow

```
USER INPUT (Health Information)
        ↓
    [HTML Form]
        ↓
    POST /diagnosis/[disease]
        ↓
    [Flask Route]
        ├─ Validate input
        ├─ Create DataFrame
        ├─ Encode categorical variables
        ├─ Scale numerical features
        ├─ Load ML model
        ├─ Make prediction
        ├─ Calculate confidence
        ├─ Save to patient_records table
        └─ Display result
```

### 3. Quiz Flow

```
USER SELECTS DIFFICULTY
        ↓
    [Quiz Levels Page]
        ↓
    POST /training/name_input
        ↓
    [Flask Route]
        ├─ Load CSV: disease_mcq_dataset_500.csv
        ├─ Filter by difficulty
        ├─ Select 10 random questions
        ├─ Store in session
        └─ Redirect to quiz
        ↓
    [Quiz Page]
        ├─ Display question
        ├─ User selects answer
        ├─ POST /training/quiz
        │   ├─ Check if correct
        │   ├─ Update score
        │   ├─ Show feedback
        │   └─ Move to next question
        └─ Repeat until all 10 questions
        ↓
    [Results Page]
        ├─ Calculate percentage
        ├─ Save to progress_data
        ├─ Update leaderboard
        └─ Display results
```

---

## Database Schema

### Users Table

```
┌─────────────────────────────────────────────────────────┐
│                    USERS TABLE                          │
├─────────────────────────────────────────────────────────┤
│ Column    │ Type    │ Constraints  │ Description        │
├───────────┼─────────┼──────────────┼────────────────────┤
│ id        │ INTEGER │ PRIMARY KEY  │ Auto-increment ID  │
│           │         │ AUTOINCREMENT│                    │
├───────────┼─────────┼──────────────┼────────────────────┤
│ name      │ TEXT    │              │ User's full name   │
├───────────┼─────────┼──────────────┼────────────────────┤
│ email     │ TEXT    │ UNIQUE       │ User's email       │
│           │         │              │ (must be unique)   │
├───────────┼─────────┼──────────────┼────────────────────┤
│ password  │ TEXT    │              │ Bcrypt hashed      │
│           │         │              │ password           │
└─────────────────────────────────────────────────────────┘
```

### Patient Records Table

```
┌──────────────────────────────────────────────────────────┐
│              PATIENT_RECORDS TABLE                       │
├──────────────────────────────────────────────────────────┤
│ Column              │ Type      │ Description            │
├─────────────────────┼───────────┼────────────────────────┤
│ id                  │ INTEGER   │ Primary key            │
├─────────────────────┼───────────┼────────────────────────┤
│ user_id             │ INTEGER   │ Foreign key (users.id) │
├─────────────────────┼───────────┼────────────────────────┤
│ first_name          │ TEXT      │ Patient first name     │
├─────────────────────┼───────────┼────────────────────────┤
│ last_name           │ TEXT      │ Patient last name      │
├─────────────────────┼───────────┼────────────────────────┤
│ phone               │ TEXT      │ Patient phone number   │
├─────────────────────┼───────────┼────────────────────────┤
│ age                 │ INTEGER   │ Patient age            │
├─────────────────────┼───────────┼────────────────────────┤
│ gender              │ TEXT      │ Patient gender         │
├─────────────────────┼───────────┼────────────────────────┤
│ symptoms            │ TEXT      │ Symptoms entered       │
├─────────────────────┼───────────┼────────────────────────┤
│ disease             │ TEXT      │ Disease type           │
├─────────────────────┼───────────┼────────────────────────┤
│ diagnosis_result    │ TEXT      │ Result (Low/Med/High)  │
├─────────────────────┼───────────┼────────────────────────┤
│ confidence_score    │ REAL      │ Model confidence %     │
├─────────────────────┼───────────┼────────────────────────┤
│ created_at          │ TIMESTAMP │ Record creation time   │
└──────────────────────────────────────────────────────────┘
```

---

## File Structure

```
AI-based-Medical-Diagnosis-and-Training-Tool/
│
├── app_flask.py                    # Main Flask application
│   ├─ Database functions
│   ├─ Authentication functions
│   ├─ Routes (login, diagnosis, training)
│   └─ ML model loading
│
├── medical_ai.db                   # SQLite database
│   ├─ users table
│   └─ patient_records table
│
├── models/                         # ML models
│   ├─ diabetes_model.pkl
│   ├─ bp_awareness_model.pkl
│   ├─ lungcancer_rf_model.pkl
│   ├─ [scalers and encoders]
│   └─ [feature lists]
│
├── templates/                      # HTML templates
│   ├─ login.html
│   ├─ mode_selection.html
│   ├─ diagnosis_diabetes.html
│   ├─ diagnosis_bp.html
│   ├─ diagnosis_lung.html
│   ├─ quiz.html
│   ├─ training_dashboard.html
│   └─ [other templates]
│
├── static/                         # CSS and images
│   ├─ style.css
│   ├─ bg.png
│   └─ [other images]
│
├── disease_mcq_dataset_500.csv     # Quiz questions
├── diabetes.csv                    # Training data
├── bp_awareness_dataset_6000.csv   # Training data
├── lungcancer.csv                  # Training data
│
└── requirements.txt                # Python dependencies
```

---

## Security Architecture

```
┌─────────────────────────────────────────────────────────┐
│              SECURITY LAYERS                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Layer 1: Input Validation                             │
│  ├─ Check for empty inputs                             │
│  ├─ Validate email format                              │
│  ├─ Validate numeric ranges                            │
│  └─ Reject invalid data                                │
│                                                         │
│  Layer 2: SQL Injection Prevention                      │
│  ├─ Use parameterized queries                          │
│  ├─ Separate SQL from user input                       │
│  ├─ Database driver escapes values                     │
│  └─ No string concatenation in SQL                     │
│                                                         │
│  Layer 3: Password Security                            │
│  ├─ Hash with Bcrypt                                   │
│  ├─ Generate salt automatically                        │
│  ├─ Slow computation (prevents brute-force)            │
│  └─ Never store plain text                             │
│                                                         │
│  Layer 4: Session Management                           │
│  ├─ Server-side session storage                        │
│  ├─ Session ID in cookie                               │
│  ├─ User data not exposed in URL                       │
│  └─ Session timeout                                    │
│                                                         │
│  Layer 5: Access Control                               │
│  ├─ @login_required decorator                          │
│  ├─ Check session['user_id']                           │
│  ├─ Redirect to login if not authenticated             │
│  └─ Protect all sensitive routes                       │
│                                                         │
│  Layer 6: Error Handling                               │
│  ├─ Generic error messages                             │
│  ├─ No database errors to user                         │
│  ├─ Log errors server-side                             │
│  └─ Prevent information leakage                        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

**Document Created**: March 21, 2026
**Status**: ✅ Complete Architecture Documentation
