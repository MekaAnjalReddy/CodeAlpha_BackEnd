Backend Development Tasks & Instructions — CodeAlpha

TASK 1: Simple URL Shortener 
● Create a backend server using Flask (Python) or Express.js (Node.js). 
● Build an API endpoint to accept long URLs and generate a unique short code. 
● Store the mapping of short code and original URL in a database (like SQLite, MongoDB, etc.). 
● Set up a redirect route where accessing the short URL takes the user to the original long URL. 
● Optional: Add a basic frontend to input long URLs and show the shortened version

Source Code:
This task involves creating a simple URL shortener using a backend server with a database to store the mappings between short codes and original URLs. Here’s how to approach the task step by step.
Requirements
•	Backend Framework: Flask (Python) or Express.js (Node.js)
•	Database: SQLite (simple choice for this example, but you could use MongoDB or another database)
•	Frontend: Basic HTML form to submit URLs and display the shortened version
•	Redirect Route: Redirect from the shortened URL to the original URL.
Let's break this down step by step. I'll use Flask (Python) for the backend and SQLite for the database. Then, we will set up a basic frontend to test it.
Step 1: Setting up the Flask backend
1.	Install dependencies:
First, you need to install Flask and SQLite:
2.	pip install flask sqlite3
3.	Create the Flask server:
Here's the Flask application that will serve the purpose of URL shortening:
from flask import Flask, request, redirect, render_template_string
import sqlite3
import hashlib
import string
import random

app = Flask(__name__)

# Set up SQLite database
def init_db():
    conn = sqlite3.connect('urls.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS url_mapping
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  long_url TEXT NOT NULL,
                  short_code TEXT NOT NULL UNIQUE)''')
    conn.commit()
    conn.close()

# Function to generate short code
def generate_short_code():
    # Generate a random 6-character short code
    chars = string.ascii_letters + string.digits
    short_code = ''.join(random.choice(chars) for _ in range(6))
    return short_code

# API endpoint to shorten URL
@app.route('/shorten', methods=['POST'])
def shorten_url():
    long_url = request.form['url']
    short_code = generate_short_code()

    # Store the mapping in the database
    conn = sqlite3.connect('urls.db')
    c = conn.cursor()
    c.execute('INSERT INTO url_mapping (long_url, short_code) VALUES (?, ?)', (long_url, short_code))
    conn.commit()
    conn.close()

    return f"Shortened URL: <a href='/{short_code}'>/{short_code}</a>"

# Redirect route to long URL
@app.route('/<short_code>')
def redirect_to_url(short_code):
    conn = sqlite3.connect('urls.db')
    c = conn.cursor()
    c.execute('SELECT long_url FROM url_mapping WHERE short_code = ?', (short_code,))
    row = c.fetchone()
    conn.close()

    if row:
        return redirect(row[0])
    else:
        return "URL not found!", 404

# Home route with form
@app.route('/')
def home():
    return render_template_string('''
        <h1>URL Shortener</h1>
        <form method="POST" action="/shorten">
            <input type="text" name="url" placeholder="Enter your URL" required>
            <button type="submit">Shorten</button>
        </form>
    ''')

if __name__ == '__main__':
    init_db()
    app.run(debug=True)
Step 2: Walkthrough of the Code
•	init_db(): This function initializes the SQLite database, creating a url_mapping table if it doesn’t exist. It has columns for id, long_url, and short_code.
•	generate_short_code(): This generates a random 6-character code from the letters and digits.
•	/shorten: The POST endpoint that accepts a URL via a form submission, generates a short code, stores it in the database, and then returns the shortened URL.
•	/<short_code>: This route will take the short code, look up the corresponding long URL in the database, and then redirect the user to that URL.
•	Home Route (/): Displays a basic HTML form to submit a long URL.
Step 3: Running the Application
1.	Run the Flask app:
2.	python app.py
3.	Open your browser and go to http://127.0.0.1:5000/. You'll see a simple form to enter a URL. After entering the URL and clicking "Shorten", you’ll get a shortened URL which you can use to test the redirection.
Optional: Frontend Enhancements (JavaScript)
If you want to add some frontend functionality using JavaScript to display the shortened URL without reloading the page, here's how you can modify the form:
Modified HTML (with JavaScript):
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>URL Shortener</title>
</head>
<body>
    <h1>URL Shortener</h1>
    <form id="url-form">
        <input type="text" id="url" placeholder="Enter your URL" required>
        <button type="submit">Shorten</button>
    </form>
    <p id="result"></p>

    <script>
        document.getElementById("url-form").addEventListener("submit", function(e) {
            e.preventDefault();
            const url = document.getElementById("url").value;
            
            fetch('/shorten', {
                method: 'POST',
                body: new URLSearchParams({ 'url': url }),
                headers: {
                    'Content-Type': 'application/x-www-form-urlencoded',
                },
            })
            .then(response => response.text())
            .then(data => {
                document.getElementById("result").innerHTML = data;
            });
        });
    </script>
</body>
</html>
This JavaScript will prevent the form from refreshing the page and instead show the shortened URL directly below the form.
Step 4: Testing
1.	Start the Flask app by running python app.py.
2.	Go to the homepage (http://127.0.0.1:5000/), enter a URL, and see the shortened URL.
3.	Click on the shortened URL or enter it in the browser to check if it redirects to the original URL.


---------------------------------------------


TASK 2: Event Registration System 
● Set up backend using Django or Express.js to manage routes and logic. 
● Create models for events and user registrations in your database (like PostgreSQL, MongoDB etc.). 
● Build API endpoints to view event list, event details, and submit registration forms. 
● Link registrations to users and events, and allow users to manage (view/cancel) their registrations.
 ● Optional: Add admin panel or authentication for event organizers

Source code:
To build an Event Registration System, we’ll go through the task in detail, and I'll walk you through setting up both the backend and models, as well as API endpoints.
Here’s how we can approach this using Express.js (Node.js) and MongoDB for the database. MongoDB is flexible for storing event and user data. Additionally, we’ll use Mongoose for ODM (Object Data Modeling) to interact with MongoDB.
Step 1: Setup Express.js and MongoDB
1.1 Install Dependencies
First, install the necessary dependencies:
npm init -y
npm install express mongoose body-parser
1.2 Setup MongoDB
Make sure you have a MongoDB instance running. You can either use a local MongoDB server or use a cloud service like MongoDB Atlas.
1.3 Set Up Basic Express App
Create a basic server.js to set up the Express server.
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

// Initialize the app
const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(bodyParser.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost/event-registration', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
  useFindAndModify: false,
})
  .then(() => console.log('Connected to MongoDB'))
  .catch((err) => console.log(err));

// Set up routes
app.use('/api/events', require('./routes/events'));
app.use('/api/registrations', require('./routes/registrations'));

// Start server
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
Step 2: Define Models for Event and Registration
2.1 Create the Event Model
Create a file called models/Event.js:
const mongoose = require('mongoose');

const EventSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: { type: String, required: true },
  date: { type: Date, required: true },
  location: { type: String, required: true },
  capacity: { type: Number, required: true },
}, { timestamps: true });

module.exports = mongoose.model('Event', EventSchema);
2.2 Create the Registration Model
Create a file called models/Registration.js:
const mongoose = require('mongoose');

const RegistrationSchema = new mongoose.Schema({
  user: { type: String, required: true }, // User can be an email or ID
  event: { type: mongoose.Schema.Types.ObjectId, ref: 'Event', required: true },
  registrationDate: { type: Date, default: Date.now },
}, { timestamps: true });

module.exports = mongoose.model('Registration', RegistrationSchema);
Step 3: Build API Endpoints
3.1 Event Routes (View Event List & Event Details)
Create a new file routes/events.js to handle routes for viewing events.
const express = require('express');
const router = express.Router();
const Event = require('../models/Event');

// View all events
router.get('/', async (req, res) => {
  try {
    const events = await Event.find();
    res.json(events);
  } catch (err) {
    res.status(500).send('Server Error');
  }
});

// View event details
router.get('/:id', async (req, res) => {
  try {
    const event = await Event.findById(req.params.id);
    if (!event) {
      return res.status(404).send('Event not found');
    }
    res.json(event);
  } catch (err) {
    res.status(500).send('Server Error');
  }
});

module.exports = router;
3.2 Registration Routes (Submit, View & Cancel Registration)
Create a new file routes/registrations.js to handle registration operations.
const express = require('express');
const router = express.Router();
const Registration = require('../models/Registration');
const Event = require('../models/Event');

// Register for an event
router.post('/', async (req, res) => {
  const { user, eventId } = req.body;

  try {
    // Check if the event exists
    const event = await Event.findById(eventId);
    if (!event) {
      return res.status(404).send('Event not found');
    }

    // Check if the event is full
    const registrations = await Registration.find({ event: eventId });
    if (registrations.length >= event.capacity) {
      return res.status(400).send('Event is full');
    }

    // Register user for the event
    const registration = new Registration({ user, event: eventId });
    await registration.save();
    res.json({ message: 'Registration successful', registration });
  } catch (err) {
    res.status(500).send('Server Error');
  }
});

// View registrations for a user
router.get('/user/:userId', async (req, res) => {
  try {
    const registrations = await Registration.find({ user: req.params.userId }).populate('event');
    res.json(registrations);
  } catch (err) {
    res.status(500).send('Server Error');
  }
});

// Cancel a registration
router.delete('/:id', async (req, res) => {
  try {
    const registration = await Registration.findById(req.params.id);
    if (!registration) {
      return res.status(404).send('Registration not found');
    }

    // Check if the registration belongs to the user
    await registration.remove();
    res.json({ message: 'Registration canceled successfully' });
  } catch (err) {
    res.status(500).send('Server Error');
  }
});

module.exports = router;
Step 4: Admin Panel or Authentication (Optional)
For the admin panel or authentication part, I recommend using a simple JWT authentication or session-based authentication. For this, we can integrate libraries like jsonwebtoken and bcryptjs.
Here’s how you might set up a basic admin authentication system:
4.1 Install JWT and bcryptjs
npm install jsonwebtoken bcryptjs
4.2 Create the User Model for Authentication
In models/User.js, we can define an admin user model to handle authentication.
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const UserSchema = new mongoose.Schema({
  username: { type: String, required: true },
  password: { type: String, required: true },
  isAdmin: { type: Boolean, default: false },
});

UserSchema.pre('save', async function(next) {
  if (this.isModified('password')) {
    const salt = await bcrypt.genSalt(10);
    this.password = await bcrypt.hash(this.password, salt);
  }
  next();
});

UserSchema.methods.matchPassword = async function(password) {
  return await bcrypt.compare(password, this.password);
};

module.exports = mongoose.model('User', UserSchema);
4.3 Authentication Routes
You could create routes to log in users (admin or general users), issue a JWT, and secure admin routes.
Step 5: Frontend (Optional)
If you want to create a frontend, you can use HTML + JavaScript or a framework like React.js to allow users to interact with the API. The main operations would include submitting a registration form, viewing events, and canceling registrations.
Step 6: Testing
1.	Create events using the /api/events POST route (this can be done using Postman or another tool).
2.	Register users using the /api/registrations POST route.
3.	View events and registrations using the /api/events/:id and /api/registrations/user/:userId routes.
4.	Cancel registrations using the /api/registrations/:id DELETE route.

