# car_rental-system
car_rental system
from flask import Flask, jsonify, request
import pyodbc
from flask_cors import CORS

app = Flask(__name__)
CORS(app)  # Allow frontend to communicate with backend

# Connect to MS Access database
DB_PATH = r"C:\path\to\your\database.accdb"  # Update this path
conn_str = f"DRIVER={{Microsoft Access Driver (*.mdb, *.accdb)}};DBQ={DB_PATH};"

def get_db_connection():
    return pyodbc.connect(conn_str)

@app.route('/cars', methods=['GET'])
def get_cars():
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT id, make, type, price, available FROM Cars")
    cars = [dict(id=row[0], make=row[1], type=row[2], price=row[3], available=row[4]) for row in cursor.fetchall()]
    conn.close()
    return jsonify(cars)

@app.route('/book/<int:car_id>', methods=['POST'])
def book_car(car_id):
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("UPDATE Cars SET available = FALSE WHERE id = ?", (car_id,))
    conn.commit()
    conn.close()
    return jsonify({"message": "Car booked successfully!"})

if __name__ == '__main__':
    app.run(debug=True)
