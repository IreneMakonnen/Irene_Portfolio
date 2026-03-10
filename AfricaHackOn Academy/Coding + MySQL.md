# Coding & MySQL Integration

**Objective**: Implement CODING scripts to interact with a MySQL database, reinforcing database connectivity, CRUD operations, and data retrieval.

## Database Creation

Using MySQL, created a database named student_records.

```shell
sudo service mysql start

sudo service mariadb start

sudo apt update

sudo apt install apache2 php libapache2-mod-php php-mbstring php-zip php-gd php-json php-curl php-mysql

sudo apt install phpmyadmin
#apache2 as the webserver

sudo phpenmod mbstring
sudo a2dissite site1.conf #Disabled the custom site
sudo a2ensite 000-default.conf #Enabled the default site
sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
sudo systemctl restart apache2

# Check if mysql is running
sudo systemctl status mysql
sudo systemctl start mysql

sudo mysql #to login as a user 
sudo mysql -u root -p #to login as root 

ALTER USER 'root'@'localhost' IDENTIFIED BY 'NewPassword';

FLUSH PRIVILEGES;
```

- Checked if mysql is running

```sh
sudo systemctl status mysql
sudo systemctl start mysql
```

- Website: http://localhost/phpmyadmin

```sql
CREATE DATABASE student_records;
```

## Table Creation

Within the `student_records` database, created a table called students with the following columns:
    - id (INT, Primary Key, Auto Increment)
    - name (VARCHAR(100))
    - age (INT)
    - email (VARCHAR(100))

```sql
show databases;

USE student_records;

CREATE TABLE students (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
    age INT,
    email VARCHAR(100)
);
```

- Website: `http://localhost/phpmyadmin`

## Data Insertion

Inserted at least four records into the students table.

```sql
INSERT INTO students (name, age, email) VALUES
('Chuck Norris', 25, 'chucknorris@martialarts.com'),
('Bruce Lee', 23, 'brucelee@martialarts.com'),
('Jackie Chan', 21, 'jackiechan@martialarts.com'),
('Jet Li', 20, 'jetli@martialarts.com');
```

- Website: http://localhost/phpmyadmin

![Database CLI View](<attachments/Coding + MySQL/Database CLI View.png>)

![Database Website View](<attachments/Coding + MySQL/Database Website View.png>)

## Data Retrieval

Wrote a coding script to connect to the MySQL database and retrieve all records from the students table. Display the records in an HTML table.

![Data Retrieval](<attachments/Coding + MySQL/Data Retrieval.png>)

- Website: `http://localhost/student_database/data-retrieval.php`

## Data Update

Implemented a coding script that updates a student's email based on their id.

![Update Student Email](<attachments/Coding + MySQL/Update Student Email.png>)
![Updated Student Email](<attachments/Coding + MySQL/Updated Student Email.png>)

- Website: `http://localhost/student_database/data-retrieval.php`

## Data Deletion

Created a coding script to delete a student record based on their id.

![Delete Student Record](<attachments/Coding + MySQL/Delete student record.png>)
![Student Record Deleted](<attachments/Coding + MySQL/Student Record Deleted.png>)

- Website: `http://localhost/student_database/data-retrieval.php`

## Connection between a Python script and a MySQL database

```shell
# Install flask (a small web framework that lets Python make websites) in a virtual environment
python3 -m venv flask
source flask/bin/activate
pip install flask mysql-connector-python
```

## A script to insert user inputs from a HTML form into tthe database

```python
from flask import Flask, request, jsonify, render_template_string
import mysql.connector

app = Flask(__name__)

# --- MySQL Connection Configuration ---
db_config = {
    "host": "localhost",
    "user": "user",           # MySQL username
    "password": "password",  # MySQL password
    "database": "student_records"  # Database name
}

# --- HTML Template (for web form) ---
HTML_TEMPLATE = """
<!DOCTYPE html>
<html>
<head>
    <title>Student Records</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; text-align: center; }
        table { border-collapse: collapse; width: 80%; margin: 20px auto; }
        th, td { border: 1px solid #000; padding: 8px 12px; text-align: left; }
        th { background-color: #f9f9f9; }
        form { margin-bottom: 30px; }
        input, button { padding: 5px; margin: 5px; }
    </style>
</head>
<body>
    <h2>Student Records</h2>

    <form method="POST" action="/add">
        <input type="text" name="name" placeholder="Name" required>
        <input type="number" name="age" placeholder="Age" required>
        <input type="email" name="email" placeholder="Email" required>
        <button type="submit">Add Student</button>
    </form>

    <table>
        <tr><th>ID</th><th>Name</th><th>Age</th><th>Email</th></tr>
        {% for student in students %}
        <tr>
            <td>{{ student[0] }}</td>
            <td>{{ student[1] }}</td>
            <td>{{ student[2] }}</td>
            <td>{{ student[3] }}</td>
        </tr>
        {% endfor %}
    </table>
</body>
</html>
"""

# --- Helper: Connect to DB ---
def get_connection():
    return mysql.connector.connect(**db_config)

# --- WEB ROUTES ---
@app.route("/add", methods=["POST"])
def add_student():
    name = request.form["name"]
    age = request.form["age"]
    email = request.form["email"]

    conn = get_connection()
    cursor = conn.cursor()
    sql = "INSERT INTO students (name, age, email) VALUES (%s, %s, %s)"
    cursor.execute(sql, (name, age, email))
    conn.commit()
    cursor.close()
    conn.close()
    return "✅ Student added successfully! <a href='/'>Back</a>"

if __name__ == "__main__":
    app.run(debug=True)
```

## Python script displaying data from a MySQL database in a table forma

```shell
# Run the python script
python3 mysql-Postman.py

* Serving Flask app 'mysql-Postman'
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: xxx-xxx-xxx
```

![Running Python Script](<attachments/Coding + MySQL/Python Script.png>)

Visited `http://localhost:5000` in the browser

![Python script displaying MySQL database](<attachments/Coding + MySQL/Python script displaying MySQL database.png>)

## Installed Postman and fetched the data inserted to the database via an API request.

**In Postman:**

1. `GET` request
Access to all student records

`http://127.0.0.1:5000/api/students`

```json
// JSON list (snippet below)
[
    {
        "age": 25,
        "email": "charlesraynorris@martialarts.com",
        "id": 1,
        "name": "Chuck Norris"
    },
    {
        "age": 23,
        "email": "brucelee@martialarts.com",
        "id": 2,
        "name": "Bruce Lee"
    },
    {
        "age": 23,
        "email": "michelleyang@martialarts.com",
        "id": 15,
        "name": "Michelle Yang"
    }
]
```

![Postman GET method](<attachments/Coding + MySQL/Postman GET method.png>)

2. `POST` Request
Added a student

```JSON
// Added the record below
{
  "name": "Postman",
  "age": 26,
  "email": "Postman@example.com"
}
```

![POST request in Postman](<attachments/Coding + MySQL/POST request Postman.png>)

3. `PUT` Request
Updated a student's email

```JSON
// Added the record below
{
    "age": 19,
    "email": "cynthia@martials.com",
    "id": 12,
    "name": "Cynthia Lee"
}
```

![Updated student's email](<attachments/Coding + MySQL/PUT request Postman.png>)

4. `DELETE` request
Deleted a record

![DELETE request in Postman](<attachments/Coding + MySQL/DELETE request Postman.png>)

Can also use: `http://localhost/student_database/data-retrieval.php`
