# Quick Start Guide

## 🚀 Get Started in 3 Steps

### Step 1: Install Dependencies
```bash
pip install -r requirements.txt
```

### Step 2: Run the Application
```bash
python app_flask.py
```

### Step 3: Open in Browser
```
http://127.0.0.1:5000/
```

---

## 📝 First Time Setup

### Create an Account
1. Click "Sign Up" on the login page
2. Enter your name, email, and password
3. Click "Create Account"

### Login
1. Enter your email and password
2. Click "Login"

---

## 🎯 Using the Application

### Diagnosis Mode
1. Select "Diagnosis Mode" from the main menu
2. Choose a disease (Diabetes, Blood Pressure, or Lung Cancer)
3. Fill in your health information
4. Click "Predict Risk"
5. View your risk assessment and recommendations
6. Optionally save the record to your history

### Training Mode
1. Select "Training Mode" from the main menu
2. Choose a difficulty level (Easy, Moderate, Hard)
3. Enter your name for the leaderboard
4. Answer 10 medical questions
5. Get immediate feedback on each answer
6. View your final score and performance comment
7. Check your progress history and leaderboard ranking

---

## 🔍 Features Overview

### Diagnosis Models
- **Diabetes**: 17-parameter assessment including BMI, lifestyle, and symptoms
- **Blood Pressure**: 13+ parameter evaluation with family history
- **Lung Cancer**: 13 symptom and risk factor assessment

### Quiz System
- 500 medical MCQs
- 3 difficulty levels
- 10 random questions per quiz
- Immediate feedback
- Progress tracking
- Leaderboard

### Patient Records
- Save all diagnosis results
- View complete history
- Track confidence scores
- Organized by date

---

## 💡 Tips

1. **BMI Calculation**: You can either enter your BMI directly or let the app calculate it from height and weight
2. **Quiz Difficulty**: Start with "Easy" if you're new to medical topics
3. **Leaderboard**: Your best score is tracked and displayed
4. **Patient History**: All diagnosis results are saved to your account

---

## ⚠️ Important

- This tool is for **educational purposes only**
- Do NOT use for clinical diagnosis
- Always consult a healthcare professional for medical advice
- Results are based on ML models trained on sample data

---

## 🆘 Troubleshooting

### Port Already in Use
If port 5000 is already in use, modify `app_flask.py`:
```python
if __name__ == '__main__':
    app.run(debug=True, port=5001)  # Change to different port
```

### Database Issues
Delete `medical_ai.db` and restart the app to reset the database:
```bash
del medical_ai.db
python app_flask.py
```

### Model Loading Warnings
These are non-critical warnings about scikit-learn version compatibility. The app will still work fine.

---

## 📞 Support

For issues or questions, refer to:
- `README.md` - Full project documentation
- `docs/PROJECT_SUMMARY.md` - Detailed feature overview
- `docs/PROJECT_STATUS.md` - Current status and verification results

---

**Happy Learning! 🎓**
