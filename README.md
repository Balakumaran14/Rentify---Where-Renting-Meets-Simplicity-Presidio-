const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  firstName: { type: String, required: true },
  lastName: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  phoneNumber: { type: String, required: true },
  password: { type: String, required: true },
  role: { type: String, required: true, enum: ['buyer', 'seller'] },
});

module.exports = mongoose.model('User', userSchema);


models/Property.js
This model will store property information such as title, description, location, number of bedrooms, number of bathrooms, rent, and a reference to the seller (user).

const mongoose = require('mongoose');

const propertySchema = new mongoose.Schema({
  title: { type: String, required: true },
  description: { type: String, required: true },
  location: { type: String, required: true },
  bedrooms: { type: Number, required: true },
  bathrooms: { type: Number, required: true },
  rent: { type: Number, required: true },
  seller: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
});

module.exports = mongoose.model('Property', propertySchema);


const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    await mongoose.connect('mongodb://localhost:27017/rentify', {
      useNewUrlParser: true,
      useUnifiedTopology: true,
      useCreateIndex: true,
    });
    console.log('MongoDB connected');
  } catch (err) {
    console.error(err.message);
    process.exit(1);
  }
};

module.exports = connectDB;


server.js
const express = require('express');
const connectDB = require('./db');
const cors = require('cors');
const bodyParser = require('body-parser');
const userRoutes = require('./routes/userRoutes');
const propertyRoutes = require('./routes/propertyRoutes');

const app = express();

// Connect to MongoDB
connectDB();

// Middleware
app.use(cors());
app.use(bodyParser.json());

// Routes
app.use('/api/users', userRoutes);
app.use('/api/properties', propertyRoutes);

const PORT = 5000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});



controllers/userController.js
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

exports.register = async (req, res) => {
  const { firstName, lastName, email, phoneNumber, password, role } = req.body;
  try {
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({
      firstName,
      lastName,
      email,
      phoneNumber,
      password: hashedPassword,
      role,
    });
    await user.save();
    res.status(201).json(user);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

exports.login = async (req, res) => {
  const { email, password } = req.body;
  try {
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    const isMatch = await bcrypt.compare(password, user.password);
    if (!isMatch) {
      return res.status(400).json({ error: 'Invalid credentials' });
    }
    const token = jwt.sign({ id: user._id }, 'yourSecretKey', { expiresIn: '1h' });
    res.status(200).json({ token });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

controllers/propertyController.js
const Property = require('../models/Property');

exports.createProperty = async (req, res) => {
  const { title, description, location, bedrooms, bathrooms, rent, seller } = req.body;
  try {
    const property = new Property({
      title,
      description,
      location,
      bedrooms,
      bathrooms,
      rent,
      seller,
    });
    await property.save();
    res.status(201).json(property);
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
};

exports.getProperties = async (req, res) => {
  try {
    const properties = await Property.find().populate('seller');
    res.status(200).json(properties);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};



routes/userRoutes.js
const express = require('express');
const { register, login } = require('../controllers/userController');

const router = express.Router();

router.post('/register', register);
router.post('/login', login);

module.exports = router;

routes/propertyRoutes.js
const express = require('express');
const { createProperty, getProperties } = require('../controllers/propertyController');

const router = express.Router();

router.post('/', createProperty);
router.get('/', getProperties);

module.exports = router;
