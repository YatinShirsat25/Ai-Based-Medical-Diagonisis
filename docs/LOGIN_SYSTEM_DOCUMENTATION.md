# Login System & Data Storage Documentation

## 📋 Table of Contents
1. [Login System Overview](#login-system-overview)
2. [Authentication Flow](#authentication-flow)
3. [Database Architecture](#database-architecture)
4. [Data Storage Details](#data-storage-details)
5. [Security Implementation](#security-implementation)
6. [Complete Data Flow](#complete-data-flow)

---

## Login System Overview

### What is the Login System?

The login system is the **authentication mechanism** that:
- Allows users to create accounts (signup)
- Allows users to log in with email and password
- Maintains user sessions
- Protects routes from unauthorized access
- Stores user credentials securely

### Key Components

```
┌─────────────────────────────────────────────────────────┐
│                    LOGIN SYSTEM                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Frontend (HTML Form)                               │
│     ├─ Email input field                               │
│     ├─ Password input field                            │
│     └─ Submit button                                   │
│                                                         │
│  2. Backend (Flask Routes)                             │
│     ├─ /login (GET/POST)                               │
│     ├─ /logout (GET)                                   │
│     └─ @login_required decorator                       │
│                                                         │
│  3. Database (SQLite)                                  │
│     ├─ Users table                                     │
│     └─ Hashed passwords                                │
│                                                         │
│  4. Security (Bcrypt)                                  │
│     ├─ Password hashing                                │
│     ├─ Salt generation                                 │
│     └─ Password verification                           │
│                                                         │
│  5. Session Management (Flask)                         │
│     ├─ Session storage                                 │
│     ├─ User ID tracking                                │
│     └─ Session timeout                                 │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Authentication Flow

### Step-by-Step Login Process

```
USER ENTERS CREDENTIALS
        ↓
    [HTML Form]
        ↓
    POST /login
        ↓
    [Flask Route Handler]
        ↓
    Extract email & password
        ↓
    Query database for user
        ↓
    User found?
        ├─ YES → Verify password with Bcrypt
        │         ├─ Password correct?
        │         │   ├─ YES → Create session
        │         │   │         ├─ Store user_id
        │         │   │         ├─ Store user_name
        │         │   │         ├─ Store user_email
        │         │   │         └─ Redirect to mode_selection
        │         │   └─ NO → Show "Invalid credentials"
        └─ NO → Show "Invalid credentials"
```

### Code Implementation

```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        if 'login' in request.form:
            # Step 1: Get user input
            email = request.form.get('email')
            password = request.form.get('password')
            
            # Step 2: Call login_user function
            success, user = login_user(email, password)
            
            # Step 3: Check if login was successful
            if success:
                # Step 4: Create session
                session['user_id'] = user[0]
                session['user_name'] = user[1]
                session['user_email'] = user[2]
                
                # Step 5: Redirect to next page
                return redirect(url_for('mode_selection'))
            else:
                # Step 6: Show error
                flash('Invalid credentials', 'error')
        
        elif 'signup' in request.form:
            # Signup logic
            name = request.form.get('name')
            email = request.form.get('email')
            password = request.form.get('password')
            
            if name.strip() and email.strip() and password.strip():
                success, msg = add_user(name, email, password)
                if success:
                    flash(msg, 'success')
                else:
                    flash(msg, 'error')
            else:
                flash('Please fill all fields', 'warning')
    
    return render_template('login.html')
```

---

## Database Architecture

### Database File Location

```
C:\Users\Aryan\Desktop\Projects 🚀\AI-based-Medical-Diagnosis-and-Training-Tool-main\
└── medical_ai.db  (SQLite database file)
```

### Database Type: SQLite3

**Why SQLite?**
- Lightweight (single file)
- No server required
- Perfect for development
- Easy to backup (just copy the file)
- Supports SQL queries

### Database Tables

#### Table 1: Users Table

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT,
    email TEXT UNIQUE,
    password TEXT
)
```

**Columns:**
- `id` - Unique identifier (auto-incremented)
- `name` - User's full name
- `email` - User's email (must be unique)
- `password` - Hashed password (NOT plain text)

**Example Data:**
```
id | name          | email              | password
---|---------------|--------------------|-----------------------------------------
1  | John Doe      | john@example.com   | $2b$12$abcdef...xyz (hashed)
2  | Jane Smith    | jane@example.com   | $2b$12$ghijkl...uvw (hashed)
3  | Aryan Kumar   | aryan@example.com  | $2b$12$mnopqr...stu (hashed)
```

#### Table 2: Patient Records Table

```sql
CREATE TABLE patient_records (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER REFERENCES users(id),
    first_name TEXT,
    last_name TEXT,
    phone TEXT,
    age INTEGER,
    gender TEXT,
    symptoms TEXT,
    disease TEXT,
    diagnosis_result TEXT,
    confidence_score REAL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
)
```

**Columns:**
- `id` - Unique record identifier
- `user_id` - Links to users table (foreign key)
- `first_name` - Patient's first name
- `last_name` - Patient's last name
- `phone` - Patient's phone number
- `age` - Patient's age
- `gender` - Patient's gender
- `symptoms` - Symptoms entered by user
- `disease` - Type of disease assessed
- `diagnosis_result` - Result (Low/Medium/High risk)
- `confidence_score` - Model's confidence percentage
- `created_at` - Timestamp of record creation

**Example Data:**
```
id | user_id | first_name | last_name | disease    | diagnosis_result | created_at
---|---------|------------|-----------|------------|------------------|-------------------
1  | 1       | John       | Doe       | Diabetes   | Low Risk         | 2026-03-21 10:30
2  | 1       | John       | Doe       | BP         | Medium Risk      | 2026-03-21 11:45
3  | 2       | Jane       | Smith     | Lung       | High Risk        | 2026-03-21 12:00
```

---

## Data Storage Details

### Where Data is Stored

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA STORAGE LOCATIONS                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. USER CREDENTIALS                                       │
│     Location: medical_ai.db → users table                  │
│     What: Email, hashed password, name                     │
│     Security: Bcrypt hashing with salt                     │
│                                                             │
│  2. PATIENT RECORDS                                        │
│     Location: medical_ai.db → patient_records table        │
│     What: Diagnosis results, symptoms, scores              │
│     Security: Linked to user_id (user-specific)            │
│                                                             │
│  3. SESSION DATA                                           │
│     Location: Server memory (Flask session)                │
│     What: user_id, user_name, user_email                   │
│     Duration: Until logout or session timeout              │
│                                                             │
│  4. ML MODELS                                              │
│     Location: models/ folder (pickle files)                │
│     What: Trained ML models for predictions                │
│     Files:                                                 │
│     ├─ diabetes_model.pkl                                  │
│     ├─ bp_awareness_model.pkl                              │
│     └─ lungcancer_rf_model.pkl                             │
│                                                             │
│  5. QUIZ DATA                                              │
│     Location: disease_mcq_dataset_500.csv                  │
│     What: 500 medical questions                            │
│     Format: CSV file                                       │
│                                                             │
│  6. TRAINING DATA                                          │
│     Location: CSV files in root directory                  │
│     Files:                                                 │
│     ├─ diabetes.csv                                        │
│     ├─ bp_awareness_dataset_6000.csv                       │
│     └─ lungcancer.csv                                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### How Data Flows Through the System

```
1. USER SIGNUP
   ├─ User enters: name, email, password
   ├─ Password is hashed with Bcrypt
   ├─ Data stored in: users table
   └─ Result: New user account created

2. USER LOGIN
   ├─ User enters: email, password
   ├─ Query: SELECT password FROM users WHERE email=?
   ├─ Verify: bcrypt.checkpw(input_password, stored_hash)
   ├─ Create: Session with user_id, user_name, user_email
   └─ Result: User logged in, redirected to mode_selection

3. DIAGNOSIS
   ├─ User enters: health information
   ├─ Data processed: Encoded, scaled, normalized
   ├─ ML Model: Makes prediction
   ├─ Store: Result in patient_records table
   └─ Result: Diagnosis saved to user's history

4. QUIZ
   ├─ Load: 10 random questions from CSV
   ├─ User answers: Questions
   ├─ Calculate: Score and percentage
   ├─ Store: Progress in session (temporary)
   └─ Result: Quiz completed, score displayed

5. LOGOUT
   ├─ Clear: Session data
   ├─ Delete: user_id, user_name, user_email
   └─ Result: User logged out, redirected to login
```

---

## Security Implementation

### 1. Password Hashing with Bcrypt

**What is Bcrypt?**
- A password hashing algorithm
- Automatically generates salt
- Slow by design (prevents brute-force attacks)
- Industry standard for password security

**How it works:**

```python
# Signup: Hash password before storing
password = "MyPassword123"
hashed_pw = bcrypt.hashpw(password.encode('utf-8'), bcrypt.gensalt())
# Result: $2b$12$abcdefghijklmnopqrstuvwxyz... (never the same twice)

# Login: Verify password
input_password = "MyPassword123"
stored_hash = "$2b$12$abcdefghijklmnopqrstuvwxyz..."
is_correct = bcrypt.checkpw(input_password.encode('utf-8'), stored_hash.encode('utf-8'))
# Result: True or False
```

**Why Bcrypt is Secure:**
- ✅ One-way hashing (can't reverse)
- ✅ Salt included (prevents rainbow tables)
- ✅ Slow computation (prevents brute-force)
- ✅ Adaptive (can increase difficulty over time)

### 2. Parameterized Queries (SQL Injection Prevention)

```python
# ❌ VULNERABLE
query = f"SELECT * FROM users WHERE email='{email}'"
cur.execute(query)

# ✅ SECURE (What we use)
cur.execute("SELECT * FROM users WHERE email=?", (email,))
```

**Why it's secure:**
- SQL structure is defined first
- User input passed separately
- Database driver escapes special characters
- Input treated as data, not code

### 3. Session Management

```python
# Create session after successful login
session['user_id'] = user[0]
session['user_name'] = user[1]
session['user_email'] = user[2]

# Session is stored server-side
# Only session ID sent to client (in cookie)
# User data never exposed in URL
```

### 4. Login Required Decorator

```python
@login_required
def protected_route():
    # Only accessible if user is logged in
    pass

# Implementation
def login_required(f):
    @wraps(f)
    def decorated_function(*args, **kwargs):
        if 'user_id' not in session:
            return redirect(url_for('login'))
        return f(*args, **kwargs)
    return decorated_function
```

---

## Complete Data Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER JOURNEY                                │
└─────────────────────────────────────────────────────────────────────┘

1. SIGNUP
   ┌──────────────────┐
   │  User enters:    │
   │  - Name          │
   │  - Email         │
   │  - Password      │
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Backend Processing:                 │
   │  1. Validate input                   │
   │  2. Hash password with Bcrypt        │
   │  3. Check email uniqueness           │
   │  4. Insert into users table          │
   └────────┬─────────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Database (medical_ai.db)            │
   │  users table:                        │
   │  id | name | email | password       │
   │  1  | John | j@... | $2b$12$...    │
   └──────────────────────────────────────┘

2. LOGIN
   ┌──────────────────┐
   │  User enters:    │
   │  - Email         │
   │  - Password      │
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Backend Processing:                 │
   │  1. Query: SELECT password FROM...   │
   │  2. Verify with bcrypt.checkpw()     │
   │  3. Create session                   │
   │  4. Store user_id, user_name, etc.   │
   └────────┬─────────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Session (Server Memory)             │
   │  session['user_id'] = 1              │
   │  session['user_name'] = 'John'       │
   │  session['user_email'] = 'j@...'     │
   └──────────────────────────────────────┘

3. DIAGNOSIS
   ┌──────────────────┐
   │  User enters:    │
   │  - Health info   │
   │  - Symptoms      │
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Backend Processing:                 │
   │  1. Validate input                   │
   │  2. Encode categorical data          │
   │  3. Scale numerical data             │
   │  4. Load ML model                    │
   │  5. Make prediction                  │
   │  6. Store result in database         │
   └────────┬─────────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Database (medical_ai.db)            │
   │  patient_records table:              │
   │  id | user_id | disease | result     │
   │  1  | 1       | Diabetes| Low Risk   │
   └──────────────────────────────────────┘

4. QUIZ
   ┌──────────────────┐
   │  User takes:     │
   │  - 10 questions  │
   │  - Answers       │
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Backend Processing:                 │
   │  1. Load questions from CSV          │
   │  2. Check answers                    │
   │  3. Calculate score                  │
   │  4. Store in session (temporary)     │
   └────────┬─────────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Session (Server Memory)             │
   │  session['score'] = 8                │
   │  session['progress_data'] = [...]    │
   └──────────────────────────────────────┘

5. LOGOUT
   ┌──────────────────┐
   │  User clicks:    │
   │  - Logout button │
   └────────┬─────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Backend Processing:                 │
   │  1. Clear session                    │
   │  2. Delete all session data          │
   │  3. Redirect to login                │
   └────────┬─────────────────────────────┘
            │
            ▼
   ┌──────────────────────────────────────┐
   │  Session (Cleared)                   │
   │  All data deleted                    │
   │  User logged out                     │
   └──────────────────────────────────────┘
```

---

## Database Queries Used

### User Management Queries

```python
# 1. Create user (signup)
cur.execute("""
    INSERT INTO users (name, email, password) 
    VALUES (?, ?, ?)
""", (name, email, hashed_password))

# 2. Get user (login)
cur.execute("""
    SELECT id, name, email, password 
    FROM users 
    WHERE email=?
""", (email,))

# 3. Check email uniqueness
cur.execute("""
    SELECT * FROM users WHERE email=?
""", (email,))
```

### Patient Records Queries

```python
# 1. Save diagnosis result
cur.execute("""
    INSERT INTO patient_records 
    (user_id, first_name, last_name, phone, age, gender, 
     symptoms, disease, diagnosis_result, confidence_score)
    VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
""", (user_id, first_name, last_name, phone, age, gender, 
      symptoms, disease, result, confidence))

# 2. Get patient history
cur.execute("""
    SELECT first_name, last_name, phone, age, gender, 
           symptoms, disease, diagnosis_result, confidence_score, created_at
    FROM patient_records
    WHERE user_id = ?
    ORDER BY created_at DESC
""", (user_id,))
```

---

## Data Persistence

### What Data Persists?

| Data | Storage | Persistence |
|------|---------|-------------|
| User credentials | SQLite DB | ✅ Permanent (until deleted) |
| Patient records | SQLite DB | ✅ Permanent (until deleted) |
| Session data | Server memory | ❌ Temporary (until logout) |
| Quiz progress | Session | ❌ Temporary (until logout) |
| ML models | Pickle files | ✅ Permanent (until updated) |

### Backup & Recovery

**Database Backup:**
```
Location: medical_ai.db
Backup: Copy the file to another location
Restore: Replace the file with backup copy
```

**What happens if database is deleted?**
- All user accounts are lost
- All patient records are lost
- App creates new empty database on restart
- Users need to create new accounts

---

## Summary

### Login System
- ✅ Secure password hashing with Bcrypt
- ✅ Parameterized queries (SQL injection proof)
- ✅ Session-based authentication
- ✅ Login required decorator for protected routes

### Data Storage
- ✅ SQLite database (medical_ai.db)
- ✅ Two main tables: users, patient_records
- ✅ Permanent storage for credentials and records
- ✅ Temporary session storage for active users

### Security
- ✅ Passwords never stored in plain text
- ✅ SQL injection prevention
- ✅ Session management
- ✅ Input validation

---

**Document Created**: March 21, 2026
**Status**: ✅ Complete Documentation
