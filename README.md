//Backend of my project 
//It will use a backend and a template engine that will directly render the HTML page
const express = require('express');
const sqlite3 = require('sqlite3').verbose();
const bodyParser = require('body-parser');
const bcrypt = require('bcrypt');
require('dotenv').config();

const app = express();
const db = new sqlite3.Database('./quick_bite.db');

// Middleware
app.use(bodyParser.urlencoded({ extended: false }));
app.use(express.static('public')); // Static files like CSS, images
app.set('view engine', 'ejs');

// Create Tables
db.serialize(() => {
    db.run("CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT UNIQUE, password TEXT)");
    db.run("CREATE TABLE IF NOT EXISTS payments (id INTEGER PRIMARY KEY, user_id INTEGER, amount REAL, status TEXT DEFAULT 'Pending')");
});

// Home Page
app.get('/', (req, res) => {
    res.render('index'); // Render home page
});

// Register Page
app.get('/register', (req, res) => {
    res.render('register');
});

app.post('/register', async (req, res) => {
    const { username, password } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    db.run("INSERT INTO users (username, password) VALUES (?, ?)", [username, hashedPassword], function(err) {
        if (err) return res.send("Error: " + err.message);
        res.redirect('/login');
    });
});

// Login Page
app.get('/login', (req, res) => {
    res.render('login');
});

app.post('/login', (req, res) => {
    const { username, password } = req.body;
    db.get("SELECT * FROM users WHERE username = ?", [username], async (err, user) => {
        if (err || !user) return res.send("Invalid credentials");
        const isValid = await bcrypt.compare(password, user.password);
        if (!isValid) return res.send("Invalid credentials");
        res.redirect('/home');
    });
});

// Home (After Login)
app.get('/home', (req, res) => {
    res.render('home');
});

// Payment Page
app.get('/payment', (req, res) => {
    res.render('payment');
});

app.post('/payment', (req, res) => {
    const { user_id, amount } = req.body;
    db.run("INSERT INTO payments (user_id, amount) VALUES (?, ?)", [user_id, amount], function(err) {
        if (err) return res.send("Error: " + err.message);
        res.redirect('/home');
    });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(Server running on http://localhost:${PORT});
});
//Defining project structure
/quick-bite
  ├── /views
  │   ├── index.ejs
  │   ├── register.ejs
  │   ├── login.ejs
  │   ├── home.ejs
  │   ├── payment.ejs
  ├── /public (For CSS, images)
  ├── server.js
  ├── package.json
  ├── .env
  //Backend of Homepage
  <!DOCTYPE html>
<html>
<head>
    <title>Quick Bite</title>
</head>
<body>
    <h1>Welcome to Quick Bite!</h1>
    <a href="/register">Register</a> | <a href="/login">Login</a>
</body>
</html>
//REgister page
<!DOCTYPE html>
<html>
<head>
    <title>Register</title>
</head>
<body>
    <h2>Register</h2>
    <form method="POST" action="/register">
        <input type="text" name="username" placeholder="Username" required>
        <input type="password" name="password" placeholder="Password" required>
        <button type="submit">Register</button>
    </form>
</body>
</html>
//Login page
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h2>Login</h2>
    <form method="POST" action="/login">
        <input type="text" name="username" placeholder="Username" required>
        <input type="password" name="password" placeholder="Password" required>
        <button type="submit">Login</button>
    </form>
</body>
</html>
//After Login page
<!DOCTYPE html>
<html>
<head>
    <title>Home</title>
</head>
<body>
    <h1>Welcome to Quick Bite</h1>
    <a href="/payment">Make Payment</a>
</body>
</html>
//Payment Transaction page
<!DOCTYPE html>
<html>
<head>
    <title>Payment</title>
</head>
<body>
    <h2>Make Payment</h2>
    <form method="POST" action="/payment">
        <input type="number" name="user_id" placeholder="User ID" required>
        <input type="number" name="amount" placeholder="Amount" required>
        <button type="submit">Pay</button>
    </form>
</body>
</html>
