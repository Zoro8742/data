from flask import Flask, render_template, request
import pymysql #type ignore
import os

app = Flask(__name__)

# Database connection function
def get_db_connection():
    return pymysql.connect(
        host=os.environ.get("DB_HOST", "localhost"),
        user=os.environ.get("DB_USER", "flask_user"),
        password=os.environ.get("DB_PASSWORD", "flask_password"),
        db=os.environ.get("DB_NAME", "flask_db"),
        cursorclass=pymysql.cursors.DictCursor
    )

@app.route("/", methods=["GET", "POST"])
def index():
    results = None
    year = ""
    if request.method == "POST":
        year = request.form.get("year")
        with get_db_connection() as conn:
            with conn.cursor() as cursor:
                cursor.execute("SELECT * FROM student_scores WHERE year = %s", (year,))
                results = cursor.fetchone() or "No data found for this year."
    return render_template("index.html", results=results, year=year)

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000, debug=True)
