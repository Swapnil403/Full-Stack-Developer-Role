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
