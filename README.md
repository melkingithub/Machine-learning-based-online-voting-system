# Machine-learning-based-online-voting-system

# Train Machine Learning Model
```
import pandas as pd
from sklearn.linear_model import LogisticRegression
import joblib

# Sample training data
data = {
    "vote_attempts": [1, 1, 2, 5, 1, 4],
    "time_taken": [30, 25, 10, 3, 40, 5],
    "fraud": [0, 0, 1, 1, 0, 1]
}

df = pd.DataFrame(data)

X = df[["vote_attempts", "time_taken"]]
y = df["fraud"]

model = LogisticRegression()
model.fit(X, y)

joblib.dump(model, "fraud_model.pkl")
print("ML model trained successfully")
```
# Database Creation
```
import sqlite3

conn = sqlite3.connect("voting.db")
cursor = conn.cursor()

cursor.execute("""
CREATE TABLE IF NOT EXISTS votes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    voter_id TEXT,
    candidate TEXT
)
""")

conn.commit()
conn.close()
print("Database created")
```

# Flask Backend
```
from flask import Flask, render_template, request, redirect
import sqlite3
import joblib

app = Flask(__name__)
model = joblib.load("fraud_model.pkl")

def get_db():
    return sqlite3.connect("voting.db")

@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        return redirect("/vote")
    return render_template("login.html")

@app.route("/vote", methods=["GET", "POST"])
def vote():
    if request.method == "POST":
        voter_id = request.form["voter_id"]
        candidate = request.form["candidate"]

        # ML Fraud Check
        prediction = model.predict([[1, 20]])

        if prediction[0] == 1:
            return "Fraud detected. Vote blocked."

        db = get_db()
        cursor = db.cursor()
        cursor.execute(
            "INSERT INTO votes (voter_id, candidate) VALUES (?, ?)",
            (voter_id, candidate)
        )
        db.commit()
        db.close()

        return "Vote submitted successfully"

    return render_template("vote.html")

@app.route("/result")
def result():
    db = get_db()
    cursor = db.cursor()
    cursor.execute("SELECT candidate, COUNT(*) FROM votes GROUP BY candidate")
    results = cursor.fetchall()
    db.close()
    return render_template("result.html", results=results)

if __name__ == "__main__":
    app.run(debug=True)
```

# HTML Files
```
<html>
<body>
<h2>Voter Login</h2>
<form method="post">
    <button type="submit">Login</button>
</form>
</body>
</html>
```
