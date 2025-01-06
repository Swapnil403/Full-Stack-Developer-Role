# Full-Stack-Developer-Role
Answers of my assessment have been uploaded hear 

Q1.....

Backend Code (Node.js + Express)

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const PORT = 5000;

// Middleware
app.use(express.json());
app.use(cors());

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/user_availability', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const availabilitySchema = new mongoose.Schema({
  user: { type: String, required: true },
  start: { type: Date, required: true },
  end: { type: Date, required: true },
  duration: { type: Number, required: true },
  scheduledSlots: { type: Array, default: [] },
});

const Availability = mongoose.model('Availability', availabilitySchema);

// Routes

// Add availability
app.post('/api/availability', async (req, res) => {
  try {
    const { user, start, end, duration } = req.body;
    const availability = new Availability({ user, start, end, duration });
    await availability.save();
    res.status(201).send(availability);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Get availability by user
app.get('/api/availability/:user', async (req, res) => {
  try {
    const { user } = req.params;
    const availability = await Availability.find({ user });
    res.status(200).send(availability);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Update availability
app.put('/api/availability/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const updates = req.body;
    const availability = await Availability.findByIdAndUpdate(id, updates, { new: true });
    res.status(200).send(availability);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Delete availability
app.delete('/api/availability/:id', async (req, res) => {
  try {
    const { id } = req.params;
    await Availability.findByIdAndDelete(id);
    res.status(200).send({ message: 'Availability deleted successfully' });
  } catch (error) {
    res.status(400).send(error);
  }
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

// Frontend Code (React)

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import Calendar from 'react-calendar';
import 'react-calendar/dist/Calendar.css';

const App = () => {
  const [availability, setAvailability] = useState([]);
  const [user, setUser] = useState('user@gmail.com');
  const [selectedDate, setSelectedDate] = useState(new Date());
  const [start, setStart] = useState('');
  const [end, setEnd] = useState('');
  const [duration, setDuration] = useState(30);

  // Fetch user availability
  useEffect(() => {
    const fetchAvailability = async () => {
      const response = await axios.get(`/api/availability/${user}`);
      setAvailability(response.data);
    };

    fetchAvailability();
  }, [user]);

  // Add availability
  const handleAddAvailability = async () => {
    const newAvailability = {
      user,
      start: new Date(`${selectedDate.toISOString().split('T')[0]}T${start}`),
      end: new Date(`${selectedDate.toISOString().split('T')[0]}T${end}`),
      duration,
    };

    const response = await axios.post('/api/availability', newAvailability);
    setAvailability([...availability, response.data]);
  };

  return (
    <div>
      <h1>Dynamic User Availability</h1>
      <label>Email:</label>
      <input
        type="email"
        value={user}
        onChange={(e) => setUser(e.target.value)}
      />
      <Calendar
        onChange={(date) => setSelectedDate(date)}
        value={selectedDate}
      />
      <div>
        <label>Start Time:</label>
        <input
          type="time"
          value={start}
          onChange={(e) => setStart(e.target.value)}
        />
        <label>End Time:</label>
        <input
          type="time"
          value={end}
          onChange={(e) => setEnd(e.target.value)}
        />
        <label>Duration (minutes):</label>
        <input
          type="number"
          value={duration}
          onChange={(e) => setDuration(e.target.value)}
        />
        <button onClick={handleAddAvailability}>Add Availability</button>
      </div>
      <h2>Availability</h2>
      <ul>
        {availability.map((slot) => (
          <li key={slot._id}>
            {new Date(slot.start).toLocaleString()} - {new Date(slot.end).toLocaleString()} ({slot.duration} minutes)
          </li>
        ))}
      </ul>
    </div>
  );
};

export default App;

Q2.
// Backend Code (Node.js + Express) with Admin Scheduling

const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const PORT = 5000;

// Middleware
app.use(express.json());
app.use(cors());

// MongoDB connection
mongoose.connect('mongodb://localhost:27017/user_availability', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

const availabilitySchema = new mongoose.Schema({
  user: { type: String, required: true },
  start: { type: Date, required: true },
  end: { type: Date, required: true },
  duration: { type: Number, required: true },
  scheduledSlots: { type: Array, default: [] },
});

const sessionSchema = new mongoose.Schema({
  user: { type: String, required: true },
  start: { type: Date, required: true },
  end: { type: Date, required: true },
  duration: { type: Number, required: true },
  attendees: [{
    name: String,
    email: String,
  }],
});

const Availability = mongoose.model('Availability', availabilitySchema);
const Session = mongoose.model('Session', sessionSchema);

// Routes

// Add availability
app.post('/api/availability', async (req, res) => {
  try {
    const { user, start, end, duration } = req.body;
    const availability = new Availability({ user, start, end, duration });
    await availability.save();
    res.status(201).send(availability);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Get availability by user
app.get('/api/availability/:user', async (req, res) => {
  try {
    const { user } = req.params;
    const availability = await Availability.find({ user });
    res.status(200).send(availability);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Schedule session
app.post('/api/schedule', async (req, res) => {
  try {
    const { user, start, end, duration, attendees } = req.body;

    // Check for conflicts
    const conflicts = await Session.find({
      user,
      $or: [
        { start: { $lt: new Date(end) }, end: { $gt: new Date(start) } },
      ],
    });

    if (conflicts.length > 0) {
      return res.status(400).send({ message: 'Time slot conflict detected' });
    }

    const session = new Session({ user, start, end, duration, attendees });
    await session.save();
    res.status(201).send(session);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Get all sessions for a user
app.get('/api/sessions/:user', async (req, res) => {
  try {
    const { user } = req.params;
    const sessions = await Session.find({ user });
    res.status(200).send(sessions);
  } catch (error) {
    res.status(400).send(error);
  }
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

// Frontend Code (React)

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import Calendar from 'react-calendar';
import 'react-calendar/dist/Calendar.css';

const AdminDashboard = () => {
  const [availability, setAvailability] = useState([]);
  const [sessions, setSessions] = useState([]);
  const [user, setUser] = useState('');
  const [start, setStart] = useState('');
  const [end, setEnd] = useState('');
  const [duration, setDuration] = useState(30);
  const [attendees, setAttendees] = useState([{ name: '', email: '' }]);

  // Fetch user availability
  useEffect(() => {
    if (user) {
      const fetchAvailability = async () => {
        const response = await axios.get(`/api/availability/${user}`);
        setAvailability(response.data);
      };

      fetchAvailability();
    }
  }, [user]);

  // Fetch user sessions
  useEffect(() => {
    if (user) {
      const fetchSessions = async () => {
        const response = await axios.get(`/api/sessions/${user}`);
        setSessions(response.data);
      };

      fetchSessions();
    }
  }, [user]);

  // Schedule session
  const handleScheduleSession = async () => {
    const newSession = {
      user,
      start: new Date(start),
      end: new Date(end),
      duration,
      attendees,
    };

    try {
      const response = await axios.post('/api/schedule', newSession);
      setSessions([...sessions, response.data]);
    } catch (error) {
      alert('Conflict detected or invalid input');
    }
  };

  return (
    <div>
      <h1>Admin Dashboard</h1>
      <label>Select User:</label>
      <input
        type="text"
        placeholder="Enter user email"
        value={user}
        onChange={(e) => setUser(e.target.value)}
      />

      <h2>Availability</h2>
      <ul>
        {availability.map((slot) => (
          <li key={slot._id}>
            {new Date(slot.start).toLocaleString()} - {new Date(slot.end).toLocaleString()} ({slot.duration} minutes)
          </li>
        ))}
      </ul>

      <h2>Schedule a Session</h2>
      <label>Start Time:</label>
      <input
        type="datetime-local"
        value={start}
        onChange={(e) => setStart(e.target.value)}
      />

      <label>End Time:</label>
      <input
        type="datetime-local"
        value={end}
        onChange={(e) => setEnd(e.target.value)}
      />

      <label>Duration (minutes):</label>
      <input
        type="number"
        value={duration}
        onChange={(e) => setDuration(e.target.value)}
      />

      <h3>Attendees</h3>
      {attendees.map((attendee, index) => (
        <div key={index}>
          <input
            type="text"
            placeholder="Name"
            value={attendee.name}
            onChange={(e) => {
              const updated = [...attendees];
              updated[index].name = e.target.value;
              setAttendees(updated);
            }}
          />
          <input
            type="email"
            placeholder="Email"
            value={attendee.email}
            onChange={(e) => {
              const updated = [...attendees];
              updated[index].email = e.target.value;
              setAttendees(updated);
            }}
          />
        </div>
      ))}

      <button
        onClick={() => setAttendees([...attendees, { name: '', email: '' }])}
      >
        Add Attendee
      </button>

      <button onClick={handleScheduleSession}>Schedule Session</button>

      <h2>Scheduled Sessions</h2>
      <ul>
        {sessions.map((session) => (
          <li key={session._id}>
            {new Date(session.start).toLocaleString()} - {new Date(session.end).toLocaleString()} ({session.duration} minutes)
            <ul>
              {session.attendees.map((attendee, idx) => (
                <li key={idx}>{attendee.name} ({attendee.email})</li>
              ))}
            </ul>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default AdminDashboard;
Q3.
Backend Enhancements:
1.	Modify Session (Reschedule):
o	PUT /api/schedule/:id allows rescheduling sessions.
2.	Cancel Session:
o	DELETE /api/schedule/:id deletes a session.
3.	Notification Preferences:
o	A user schema enhancement for notification preferences (email, sms).
// Modify session (Reschedule)
app.put('/api/schedule/:id', async (req, res) => {
  try {
    const { id } = req.params;
    const { start, end } = req.body;

    // Check for conflicts
    const existingSession = await Session.find({
      _id: { $ne: id },
      $or: [
        { start: { $lt: new Date(end) }, end: { $gt: new Date(start) } },
      ],
    });

    if (existingSession.length > 0) {
      return res.status(400).send({ message: 'Conflict with existing sessions' });
    }

    const updatedSession = await Session.findByIdAndUpdate(
      id,
      { start, end },
      { new: true }
    );

    res.status(200).send(updatedSession);
    // Add notification logic here
  } catch (error) {
    res.status(400).send(error);
  }
});

// Cancel session
app.delete('/api/schedule/:id', async (req, res) => {
  try {
    const { id } = req.params;

    const deletedSession = await Session.findByIdAndDelete(id);
    res.status(200).send(deletedSession);

    // Add notification logic here
  } catch (error) {
    res.status(400).send(error);
  }
});

// User notification preferences
const userSchema = new mongoose.Schema({
  email: { type: String, required: true },
  notificationPreferences: {
    email: { type: Boolean, default: true },
    sms: { type: Boolean, default: false },
  },
});
const User = mongoose.model('User', userSchema);
Q4.
1.	HTML Structure for Scheduling and Availability
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scheduling and Availability</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>Manage Your Availability</h1>
    </header>
    
    <section class="availability">
        <h2>Available Time Slots</h2>
        <div class="time-slots">
            <button class="slot">9:00 AM</button>
            <button class="slot">10:00 AM</button>
            <button class="slot">11:00 AM</button>
            <button class="slot">1:00 PM</button>
            <button class="slot">2:00 PM</button>
        </div>
    </section>

    <section class="sessions">
        <h2>Your Sessions</h2>
        <ul class="session-list">
            <li>Session 1: 9:00 AM - 10:00 AM</li>
            <li>Session 2: 1:00 PM - 2:00 PM</li>
        </ul>
        <button class="add-session-btn">Add New Session</button>
    </section>

    <footer>
        <p>© 2025 Your Company. All rights reserved.</p>
    </footer>

    <script src="script.js"></script>
</body>
</html>
2.	CSS for Responsive and Accessible Design
/* styles.css */

/* Global styles */
body {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    background-color: #f4f4f9;
}

header, footer {
    background-color: #282c34;
    color: #ffffff;
    text-align: center;
    padding: 15px;
}

h1, h2 {
    margin: 0;
    padding: 0;
}

button {
    padding: 10px;
    margin: 5px;
    font-size: 16px;
    cursor: pointer;
    border: none;
    background-color: #4CAF50;
    color: white;
    border-radius: 4px;
}

button:hover {
    background-color: #45a049;
}

button:disabled {
    background-color: #c1c1c1;
    cursor: not-allowed;
}

/* Time slots and session list */
.time-slots {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    margin-top: 20px;
}

.slot {
    width: 150px;
    margin: 10px;
    text-align: center;
}

/* Responsive Layout */
.sessions, .availability {
    margin: 20px;
}

.session-list {
    list-style-type: none;
    padding: 0;
}

.session-list li {
    margin-bottom: 10px;
}

@media (max-width: 768px) {
    .time-slots {
        flex-direction: column;
        align-items: center;
    }

    .slot {
        width: 100%;
        margin: 5px 0;
    }
}

/* Accessibility: Ensure good color contrast */
button, h1, h2 {
    color: #fff;
}

footer p {
    font-size: 14px;
}

/* Focus styles for accessibility */
button:focus, .slot:focus {
    outline: 3px solid #FFD700;
}
3.	JavaScript for Managing Sessions and Availability
// script.js

document.addEventListener('DOMContentLoaded', function () {
    // Handle adding new session
    const addSessionBtn = document.querySelector('.add-session-btn');
    addSessionBtn.addEventListener('click', function () {
        const availableSlots = document.querySelectorAll('.slot');
        availableSlots.forEach(slot => {
            slot.addEventListener('click', function () {
                const sessionTime = this.textContent;
                addSession(sessionTime);
            });
        });
    });

    // Function to add a session to the list
    function addSession(time) {
        const sessionList = document.querySelector('.session-list');
        const sessionItem = document.createElement('li');
        sessionItem.textContent = `Session: ${time}`;
        sessionList.appendChild(sessionItem);
    }
});

1. HTML
a. How do you ensure accessibility when creating a web page?
•	Semantic HTML: Use proper HTML elements like <header>, <footer>, <nav>, <article>, and <section> to help screen readers understand the page structure.
•	ARIA (Accessible Rich Internet Applications): Use ARIA attributes like aria-label, aria-describedby, and aria-hidden to improve accessibility.
•	Color contrast: Ensure sufficient contrast between text and background for readability.
•	Keyboard accessibility: Ensure all interactive elements are focusable and usable with keyboard navigation.
b. What are semantic HTML tags, and why are they important?
•	Semantic HTML tags: These are HTML elements that carry meaning about their content (e.g., <header>, <footer>, <article>, <section>, <nav>, etc.).
•	Importance:
o	They help with SEO.
o	They improve accessibility for users with disabilities.
o	They make the structure of the document clearer for developers and maintainers.
c. How would you implement an HTML form with validation for email, phone number, and password strength? Here’s an example of a form with validation:
html
Copy code
<form id="userForm">
  <label for="email">Email:</label>
  <input type="email" id="email" name="email" required>
  
  <label for="phone">Phone:</label>
  <input type="tel" id="phone" name="phone" pattern="^[0-9]{10}$" required>
  
  <label for="password">Password:</label>
  <input type="password" id="password" name="password" required minlength="8" pattern="^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$">
  
  <button type="submit">Submit</button>
</form>

<script>
  document.getElementById('userForm').addEventListener('submit', function(e) {
    const email = document.getElementById('email').value;
    const phone = document.getElementById('phone').value;
    const password = document.getElementById('password').value;

    if (!email || !phone || !password) {
      e.preventDefault();
      alert("Please fill all fields correctly.");
    }
  });
</script>
________________________________________
2. CSS
a. Explain the difference between relative, absolute, and fixed positioning in CSS.
•	Relative positioning: An element is positioned relative to its normal position in the document flow. You can adjust it using top, left, right, or bottom.
•	Absolute positioning: An element is positioned relative to its nearest positioned ancestor (i.e., one with relative, absolute, or fixed set). If no ancestor is found, it will be relative to the <html> element.
•	Fixed positioning: The element is fixed to the viewport, meaning it remains in the same position even when the page is scrolled.
css
Copy code
.relative { position: relative; top: 20px; left: 30px; }
.absolute { position: absolute; top: 50px; left: 100px; }
.fixed { position: fixed; top: 10px; right: 10px; }
b. How would you create a responsive navigation bar without using any libraries?
Here’s a simple CSS-only responsive navigation bar:
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Responsive Navbar</title>
  <style>
    body { font-family: Arial, sans-serif; }
    nav { background-color: #333; }
    nav ul { list-style-type: none; padding: 0; margin: 0; }
    nav ul li { display: inline-block; }
    nav ul li a { color: white; padding: 15px 20px; display: block; text-decoration: none; }
    nav ul li a:hover { background-color: #ddd; color: black; }
    
    /* Mobile Styles */
    @media (max-width: 768px) {
      nav ul { text-align: center; }
      nav ul li { display: block; width: 100%; }
    }
  </style>
</head>
<body>
  <nav>
    <ul>
      <li><a href="#home">Home</a></li>
      <li><a href="#about">About</a></li>
      <li><a href="#services">Services</a></li>
      <li><a href="#contact">Contact</a></li>
    </ul>
  </nav>
</body>
</html>
c. Provide a solution for centering a div both horizontally and vertically using CSS.
You can use Flexbox to center a div both horizontally and vertically:
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Centered Div</title>
  <style>
    .container {
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    .box {
      width: 200px;
      height: 200px;
      background-color: #4CAF50;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="box"></div>
  </div>
</body>
</html>
________________________________________
3. JavaScript Problem-Solving
a. Write a function to check if a given string is a palindrome.
javascript
Copy code
function isPalindrome(str) {
  const cleanedStr = str.replace(/[^a-zA-Z0-9]/g, '').toLowerCase();
  return cleanedStr === cleanedStr.split('').reverse().join('');
}

console.log(isPalindrome("A man, a plan, a canal, Panama")); // true
b. Implement a function to flatten a nested array.
javascript
Copy code
function flattenArray(arr) {
  return arr.reduce((acc, val) => Array.isArray(val) ? acc.concat(flattenArray(val)) : acc.concat(val), []);
}

console.log(flattenArray([1, [2, [3, 4], 5], 6])); // [1, 2, 3, 4, 5, 6]
c. Implement a custom dropdown menu using JavaScript.
html
Copy code
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Custom Dropdown</title>
  <style>
    .dropdown { position: relative; display: inline-block; }
    .dropdown-content { display: none; position: absolute; background-color: #f9f9f9; min-width: 160px; z-index: 1; }
    .dropdown:hover .dropdown-content { display: block; }
    .dropdown-content a { color: black; padding: 12px 16px; text-decoration: none; display: block; }
    .dropdown-content a:hover { background-color: #f1f1f1; }
  </style>
</head>
<body>

<div class="dropdown">
  <button class="dropbtn">Dropdown</button>
  <div class="dropdown-content">
    <a href="#">Option 1</a>
    <a href="#">Option 2</a>
    <a href="#">Option 3</a>
  </div>
</div>

</body>
</html>
________________________________________
4. React
a. Explain the difference between controlled and uncontrolled components in React.
•	Controlled Components: The form data is controlled by React state. The value of the form element is set by the state, and updates are handled via onChange event.
•	Uncontrolled Components: The form data is handled by the DOM. You access the value of the form element via refs.
jsx
Copy code
// Controlled Component Example
function ControlledForm() {
  const [email, setEmail] = useState("");
  
  return (
    <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
  );
}
b. How would you optimize a React app for performance?
•	Use React.memo to prevent unnecessary re-renders of functional components.
•	Use useCallback and useMemo to memoize functions and values.
•	Implement code-splitting using React's React.lazy() and Suspense.
•	Avoid inline functions inside JSX because they can cause unnecessary re-renders.
c. Implement a simple React component to display a paginated list of items.
jsx
Copy code
import React, { useState } from 'react';

const PaginatedList = ({ items }) => {
  const [currentPage, setCurrentPage] = useState(1);
  const itemsPerPage = 5;

  const indexOfLastItem = currentPage * itemsPerPage;
  const indexOfFirstItem = indexOfLastItem - itemsPerPage;
  const currentItems = items.slice(index



