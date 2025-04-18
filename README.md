// Backend for Medicine Platform (Node.js + Express + MongoDB)
const express = require('express'); const mongoose = require('mongoose'); const multer = require('multer'); const cors = require('cors'); const app = express();
app.use(express.json()); app.use(cors());
// MongoDB Connection mongoose.connect('mongodb://localhost:27017/medicine-app', { useNewUrlParser: true, useUnifiedTopology: true, });
// Models const User = mongoose.model('User', new mongoose.Schema({ email: String, password: String, }));
const Medicine = mongoose.model('Medicine', new mongoose.Schema({ name: String, price: String, description: String, }));
const Order = mongoose.model('Order', new mongoose.Schema({ userId: String, items: Array, prescriptionUrl: String, }));
// File Upload Setup const storage = multer.diskStorage({ destination: (req, file, cb) => cb(null, 'uploads/'), filename: (req, file, cb) => cb(null, Date.now() + '-' + file.originalname) }); const upload = multer({ storage });
// Routes app.post('/api/register', async (req, res) => { const user = new User(req.body); await user.save(); res.send({ message: 'User registered' }); });
app.post('/api/login', async (req, res) => { const user = await User.findOne(req.body); if (user) res.send({ message: 'Login successful', user }); else res.status(401).send({ message: 'Invalid credentials' }); });
app.get('/api/medicines', async (req, res) => { const meds = await Medicine.find(); res.send(meds); });
app.post('/api/order', upload.single('prescription'), async (req, res) => { const { userId, items } = req.body; const prescriptionUrl = req.file ? /uploads/${req.file.filename} : ''; const order = new Order({ userId, items: JSON.parse(items), prescriptionUrl }); await order.save(); res.send({ message: 'Order placed' }); });
app.use('/uploads', express.static('uploads'));
app.listen(5000, () => console.log('Server started on http://localhost:5000'));
