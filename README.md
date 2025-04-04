import sqlite3
from flask import Flask, render_template, request, redirect, url_for

# Initialize Flask app
app = Flask(__name__)

# Define database file
DATABASE = 'rolsa.db'

def get_db():
    """
    Connect to the SQLite database and return the connection.
    Uses the same database connection throughout the app.
    """
    conn = sqlite3.connect(DATABASE)
    conn.row_factory = sqlite3.Row  # Rows will be returned as dictionaries
    return conn

# Route: Home (display all the bookings)
@app.route('/')
def rolsa():
    """
    Fetch all bookings from the database and render them on the index page.
    """
    conn = get_db()
    cursor = conn.cursor()
    cursor.execute("SELECT emailaddress, postcode, fullname FROM bookings")
    bookings = cursor.fetchall()
    conn.close()
    return render_template('index.html', bookings=bookings)

# Route: Add a new booking
@app.route('/add', methods=['POST'])
def rolsa():
    """
    Add a new booking to the database.
    """
    emailaddress = request.form.get('emailaddress')
    postcode = request.form.get('postcode')
    fullname = request.form.get('fullname')
    housenumber = request.form.get('housenumber')

    if emailaddress and fullname:
        conn = get_db()
        cursor = conn.cursor()
        cursor.execute("INSERT INTO bookings (emailaddress, postcode, fullname, housenumber) VALUES (?, ?, ?,?)",
                       (emailaddress, postcode, fullname,housenumber))
        conn.commit()
        conn.close()
        return redirect(url_for(''))

# Route: Delete a booking
@app.route('/delete/<int:booking_id>')
def delete_booking(booking_id):
    """
    Delete a booking from the database.
    """
    conn = get_db()
    cursor = conn.cursor()
    cursor.execute("DELETE FROM bookings WHERE id=?", (booking_id,))
    conn.commit()
    conn.close()
    return redirect(url_for('wishlist'))

if __name__ == '__main__':
    # Create the database and table if they do not exist
    with sqlite3.connect(DATABASE) as conn:
        cursor = conn.cursor()
        cursor.execute("""
            CREATE TABLE IF NOT EXISTS bookings (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                emailaddress TEXT NOT NULL,
                postcode TEXT NOT NULL,
                fullname TEXT NOT NULL
            )
        """)
        conn.commit()
    # Start the Flask app
    app.run(debug=True)
