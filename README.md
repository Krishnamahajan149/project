// ================= BACKEND (Node.js + Express + MongoDB) =================

// 1. Install dependencies:
// npm init -y
// npm install express mongoose cors body-parser

// server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
app.use(cors());
app.use(bodyParser.json());

// Connect MongoDB
mongoose.connect('mongodb://127.0.0.1:27017/foodwaste', {
  useNewUrlParser: true,
  useUnifiedTopology: true
});

// Schema
const FoodSchema = new mongoose.Schema({
  name: String,
  quantity: String,
  location: String,
  contact: String,
  status: { type: String, default: 'Available' }
});

const Food = mongoose.model('Food', FoodSchema);

// Routes
app.post('/add-food', async (req, res) => {
  const food = new Food(req.body);
  await food.save();
  res.send(food);
});

app.get('/foods', async (req, res) => {
  const foods = await Food.find();
  res.send(foods);
});

app.put('/claim/:id', async (req, res) => {
  await Food.findByIdAndUpdate(req.params.id, { status: 'Claimed' });
  res.send('Claimed');
});

app.listen(5000, () => console.log('Server running on port 5000'));


// ================= FRONTEND (React) =================

// 1. Create React App
// npx create-react-app frontend
// cd frontend
// npm install axios

// src/App.js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

function App() {
  const [foods, setFoods] = useState([]);
  const [form, setForm] = useState({ name: '', quantity: '', location: '', contact: '' });

  const fetchFoods = async () => {
    const res = await axios.get('http://localhost:5000/foods');
    setFoods(res.data);
  };

  useEffect(() => {
    fetchFoods();
  }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    await axios.post('http://localhost:5000/add-food', form);
    fetchFoods();
  };

  const claimFood = async (id) => {
    await axios.put(`http://localhost:5000/claim/${id}`);
    fetchFoods();
  };

  return (
    <div style={{ padding: 20 }}>
      <h1>Food Waste Management</h1>

      <form onSubmit={handleSubmit}>
        <input placeholder="Food Name" onChange={e => setForm({ ...form, name: e.target.value })} />
        <input placeholder="Quantity" onChange={e => setForm({ ...form, quantity: e.target.value })} />
        <input placeholder="Location" onChange={e => setForm({ ...form, location: e.target.value })} />
        <input placeholder="Contact" onChange={e => setForm({ ...form, contact: e.target.value })} />
        <button type="submit">Add Food</button>
      </form>

      <h2>Available Food</h2>
      {foods.map(food => (
        <div key={food._id} style={{ border: '1px solid black', margin: 10, padding: 10 }}>
          <p>{food.name} - {food.quantity}</p>
          <p>{food.location}</p>
          <p>Status: {food.status}</p>
          {food.status === 'Available' && (
            <button onClick={() => claimFood(food._id)}>Claim</button>
          )}
        </div>
      ))}
    </div>
  );
}

export default App;


// ================= DATABASE =================

// MongoDB (NoSQL) Example Document:
// {
//   "_id": "...",
//   "name": "Rice",
//   "quantity": "5 plates",
//   "location": "Pune",
//   "contact": "9876543210",
//   "status": "Available"
// }


// ================= RUN PROJECT =================

// Step 1: Start MongoDB
// mongod

// Step 2: Run backend
// node server.js

// Step 3: Run frontend
// npm start

// Open: http://localhost:3000
