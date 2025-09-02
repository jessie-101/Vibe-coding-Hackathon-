# Vibe-coding-Hackathon-

index.html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>StudyBuddy Quiz</title>
  <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
  <div class="container">
    <h1>Welcome to StudyBuddy </h1>
    <p>Test your knowledge with a free quiz!</p>

    <div id="quiz">
      <p><b>Q1:</b> What does HTML stand for?</p>
      <button onclick="checkAnswer(this, true)">HyperText Markup Language</button>
      <button onclick="checkAnswer(this, false)">HighText Markdown Language</button>
      <button onclick="checkAnswer(this, false)">Hyperlinks and Text Markup Language</button>

      <p><b>Q2:</b> What does CSS stand for?</p>
      <button onclick="checkAnswer(this, true)">Cascading Style Sheets</button>
      <button onclick="checkAnswer(this, false)">Creative Style System</button>
      <button onclick="checkAnswer(this, false)">Computer Styling Syntax</button>
    </div>

    <p id="result"></p>

    <a href="{{ url_for('payment') }}">
      <button class="premium-btn">Unlock Premium Quizzes </button>
    </a>
  </div>
  <script src="{{ url_for('static', filename='js/script.js') }}"></script>
</body>
</html>



payment.html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Payment - StudyBuddy</title>
  <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
  <div class="container">
    <h2>Upgrade to Premium </h2>
    <p>Pay KES 10 via M-Pesa to access all quizzes.</p>

    <form action="{{ url_for('payment') }}" method="POST">
      <input type="text" name="phone" placeholder="Enter your phone (2547...)" required>
      <button type="submit">Pay with M-Pesa</button>
    </form>
  </div>
</body>
</html>



success.html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Payment Success - StudyBuddy</title>
  <link rel="stylesheet" href="{{ url_for('static', filename='css/style.css') }}">
</head>
<body>
  <div class="container">
    <h2>Payment Request Sent</h2>
    <p>We sent an M-Pesa prompt to <b>{{ phone }}</b>. Complete payment to unlock premium quizzes.</p>

    <a href="{{ url_for('premium', phone=phone) }}">
      <button class="premium-btn">Go to Premium Quizzes </button>
    </a>
  </div>
</body>
</html>



style.css

body {
  font-family: Arial, sans-serif;
  background: #f9f9f9;
  color: #333;
  text-align: center;
  padding: 20px;
}

.container {
  max-width: 600px;
  margin: auto;
  background: #fff;
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0px 4px 6px rgba(0,0,0,0.1);
}

button {
  display: block;
  width: 100%;
  margin: 10px 0;
  padding: 12px;
  background: #4CAF50;
  color: white;
  font-size: 16px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}

.premium-btn {
  background: #ff9800;
}

.premium-btn:hover {
  background: #e68900;
}

input {
  width: 90%;
  padding: 12px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 10px;
}



script.js

let score = 0;

function checkAnswer(button, isCorrect) {
  if (isCorrect) {
    score++;
    button.style.backgroundColor = "green";
  } else {
    button.style.backgroundColor = "red";
  }
  document.getElementById("result").innerText = "Your Score: " + score;
}

 
 
 
 Python.

from flask import Flask, render_template, request
import sqlite3

app = Flask(__name__)

# --- Database Setup ---
def init_db():
    conn = sqlite3.connect("studybuddy.db")
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  phone TEXT,
                  is_premium INTEGER DEFAULT 0)''')
    conn.commit()
    conn.close()

init_db()

@app.route('/')
def home():
    return render_template('index.html')

@app.route('/payment', methods=['GET', 'POST'])
def payment():
    if request.method == 'POST':
        phone = request.form['phone']
        # Store user with non-premium first
        conn = sqlite3.connect("studybuddy.db")
        c = conn.cursor()
        c.execute("INSERT INTO users (phone, is_premium) VALUES (?, ?)", (phone, 1))
        conn.commit()
        conn.close()
        return render_template('success.html', phone=phone)
    return render_template('payment.html')

@app.route('/premium/<phone>')
def premium(phone):
    conn = sqlite3.connect("studybuddy.db")
    c = conn.cursor()
    c.execute("SELECT is_premium FROM users WHERE phone=?", (phone,))
    row = c.fetchone()
    conn.close()
    if row and row[0] == 1:
        return f"<h1> Welcome to Premium, {phone}!</h1><p>Enjoy more quizzes here.</p>"
    else:
        return "<h1>Access Denied</h1><p>Please complete payment to access premium quizzes.</p>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)



   requirements.txt
   
   Flask==2.3.2
Flask-SQLAlchemy==3.0.5
requests==2.31.0
gunicorn==21.2.0


render.yaml

services:
  - type: web
    name: studybuddy-app
    env: python
    buildCommand: "pip install -r requirements.txt"
    startCommand: "gunicorn app:app"
    plan: free
    autoDeploy: true






