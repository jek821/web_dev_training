## **Step 1: Setting Up the Project**

### **1. Install Dependencies**

#### **Backend**
1. Create a backend project folder:
   ```bash
   mkdir pet-tracker-backend
   cd pet-tracker-backend
   ```
2. Initialize a Node.js project:
   ```bash
   npm init -y
   ```
3. Install required dependencies:
   ```bash
   npm install express mongoose dotenv cors auth0 express-jwt
   ```
4. Install development dependencies:
   ```bash
   npm install --save-dev nodemon
   ```

#### **Frontend**
1. Go back to the parent directory and create a frontend folder:
   ```bash
   mkdir pet-tracker-frontend
   cd pet-tracker-frontend
   ```
2. Initialize a Vite app with Vue:
   ```bash
   npm create vite@latest pet-tracker-frontend --template vue
   ```
3. Install required dependencies:
   ```bash
   npm install axios pinia tailwindcss postcss autoprefixer
   ```
4. Initialize TailwindCSS:
   ```bash
   npx tailwindcss init
   ```

---

### **2. Set Up File Structure**

#### **Backend**
Inside `pet-tracker-backend`, create these directories and files:
```bash
mkdir config controllers middlewares models routes utils
touch server.js
```

Your final structure should look like this:
```
pet-tracker-backend/
├── config/
│   └── db.js
├── controllers/
│   └── userController.js
│   └── petController.js
│   └── tradeController.js
├── middlewares/
│   └── authMiddleware.js
│   └── errorMiddleware.js
├── models/
│   └── user.js
│   └── pet.js
│   └── trade.js
├── routes/
│   └── userRoutes.js
│   └── petRoutes.js
│   └── tradeRoutes.js
├── utils/
│   └── logger.js
├── .env
├── server.js
```

#### **Frontend**
Inside `src/` of `pet-tracker-frontend`, structure it like this:
```
src/
├── assets/
├── components/
│   └── PetList.vue
│   └── TradeModal.vue
│   └── Inventory.vue
├── router/
│   └── index.js
├── stores/
│   └── authStore.js
│   └── petStore.js
│   └── tradeStore.js
├── styles/
│   └── tailwind.css
├── utils/
│   └── api.js
├── views/
│   └── Dashboard.vue
│   └── AdminPage.vue
│   └── Login.vue
├── App.vue
├── main.js
```

---

## **Step 2: Backend Setup**

### **1. Configure MongoDB**
1. Create a free MongoDB Atlas account.
2. Create a cluster, add a database, and name it `pet-tracker`.
3. Create a database user with username and password.
4. Whitelist your IP or allow all IPs (`0.0.0.0/0`).

5. Add a connection string to `.env`:
   ```
   MONGO_URI=mongodb+srv://<username>:<password>@cluster0.mongodb.net/pet-tracker?retryWrites=true&w=majority
   ```

### **2. Configure Auth0**
1. Create an Auth0 tenant and application.
2. Set application type to **Regular Web Application**.
3. Note the domain, client ID, and client secret.
4. Add callback URLs (e.g., `http://localhost:3000/callback`).
5. Add logout URLs (e.g., `http://localhost:3000`).
6. Add Auth0 configuration to `.env`:
   ```
   AUTH0_DOMAIN=<your-auth0-domain>
   AUTH0_CLIENT_ID=<your-client-id>
   AUTH0_CLIENT_SECRET=<your-client-secret>
   ```

---

### **3. Backend Code**

#### **Database Connection**
- File: `config/db.js`
```javascript
const mongoose = require('mongoose');

const connectToDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true });
    console.log('Connected to MongoDB');
  } catch (err) {
    console.error('MongoDB connection failed:', err);
    process.exit(1);
  }
};

module.exports = connectToDB;
```

#### **Auth Middleware**
- File: `middlewares/authMiddleware.js`
```javascript
const { expressjwt: jwt } = require('express-jwt');

const authMiddleware = jwt({
  secret: process.env.AUTH0_CLIENT_SECRET,
  audience: process.env.AUTH0_CLIENT_ID,
  issuer: `https://${process.env.AUTH0_DOMAIN}/`,
  algorithms: ['HS256'],
});

module.exports = authMiddleware;
```

#### **User Schema**
- File: `models/user.js`
```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  role: { type: String, enum: ['user', 'admin'], default: 'user' },
  inventory: [{ type: String }],
  pets: [{ type: mongoose.Schema.Types.ObjectId, ref: 'Pet' }],
});

module.exports = mongoose.model('User', userSchema);
```

#### **Server Setup**
- File: `server.js`
```javascript
require('dotenv').config();
const express = require('express');
const connectToDB = require('./config/db');
const userRoutes = require('./routes/userRoutes');
const authMiddleware = require('./middlewares/authMiddleware');

const app = express();
connectToDB();

app.use(express.json());
app.use('/api/users', userRoutes);

app.listen(5000, () => {
  console.log('Server running on http://localhost:5000');
});
```

---

## **Step 3: Frontend Setup**

### **TailwindCSS Configuration**
- Update `tailwind.config.js`:
```javascript
module.exports = {
  content: ['./index.html', './src/**/*.{vue,js,ts,jsx,tsx}'],
  theme: { extend: {} },
  plugins: [],
};
```
- Add Tailwind to `styles/tailwind.css`:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```
- Import Tailwind into `main.js`:
```javascript
import './styles/tailwind.css';
```

---

### **Router Configuration**
- File: `router/index.js`
```javascript
import { createRouter, createWebHistory } from 'vue-router';
import Dashboard from '../views/Dashboard.vue';
import Login from '../views/Login.vue';

const routes = [
  { path: '/', component: Login },
  { path: '/dashboard', component: Dashboard },
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});

export default router;
```

---

### **Auth Store**
- File: `stores/authStore.js`
```javascript
import { defineStore } from 'pinia';

export const useAuthStore = defineStore('auth', {
  state: () => ({
    user: null,
    token: null,
  }),
  actions: {
    login(userData, token) {
      this.user = userData;
      this.token = token;
    },
    logout() {
      this.user = null;
      this.token = null;
    },
  },
});
```

---

## **Step 4: Backend Feature Implementations**

### **1. User Routes**
- File: `routes/userRoutes.js`
```javascript
const express = require('express');
const { getUsers, getUserById, createUser } = require('../controllers/userController');
const authMiddleware = require('../middlewares/authMiddleware');

const router = express.Router();

router.get('/', authMiddleware, getUsers);
router.get('/:id', authMiddleware, getUserById);
router.post('/', createUser); // Public route for user registration

module.exports = router;
```

### **2. User Controller**
- File: `controllers/userController.js`
```javascript
const User = require('../models/user');

// Get all users (for trading interface)
exports.getUsers = async (req, res) => {
  try {
    const users = await User.find().select('username inventory pets role');
    res.status(200).json(users);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch users' });
  }
};

// Get single user by ID
exports.getUserById = async (req, res) => {
  try {
    const user = await User.findById(req.params.id).populate('pets');
    if (!user) return res.status(404).json({ error: 'User not found' });
    res.status(200).json(user);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch user' });
  }
};

// Create new user
exports.createUser = async (req, res) => {
  const { username, email } = req.body;
  try {
    const newUser = new User({ username, email, inventory: [], pets: [] });
    await newUser.save();
    res.status(201).json(newUser);
  } catch (err) {
    res.status(500).json({ error: 'Failed to create user' });
  }
};
```

---

### **3. Trade Routes**
- File: `routes/tradeRoutes.js`
```javascript
const express = require('express');
const { createTrade, getTrades } = require('../controllers/tradeController');
const authMiddleware = require('../middlewares/authMiddleware');

const router = express.Router();

router.get('/', authMiddleware, getTrades);
router.post('/', authMiddleware, createTrade);

module.exports = router;
```

### **4. Trade Controller**
- File: `controllers/tradeController.js`
```javascript
const Trade = require('../models/trade');
const User = require('../models/user');

// Get all trades (Admin feature)
exports.getTrades = async (req, res) => {
  try {
    const trades = await Trade.find().populate('sender receiver');
    res.status(200).json(trades);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch trades' });
  }
};

// Create new trade
exports.createTrade = async (req, res) => {
  const { senderId, receiverId, items } = req.body;

  try {
    const sender = await User.findById(senderId);
    const receiver = await User.findById(receiverId);

    // Validate inventory
    const invalidItems = items.filter(item => !sender.inventory.includes(item));
    if (invalidItems.length > 0) {
      return res.status(400).json({ error: `Sender does not own: ${invalidItems.join(', ')}` });
    }

    // Update inventories
    sender.inventory = sender.inventory.filter(item => !items.includes(item));
    receiver.inventory.push(...items);

    // Save changes
    await sender.save();
    await receiver.save();

    // Log the trade
    const trade = new Trade({ sender: senderId, receiver: receiverId, items });
    await trade.save();

    res.status(201).json(trade);
  } catch (err) {
    res.status(500).json({ error: 'Failed to create trade' });
  }
};
```

---

### **5. Trade Schema**
- File: `models/trade.js`
```javascript
const mongoose = require('mongoose');

const tradeSchema = new mongoose.Schema({
  sender: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  receiver: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
  items: [{ type: String, required: true }],
  timestamp: { type: Date, default: Date.now },
});

module.exports = mongoose.model('Trade', tradeSchema);
```

---

## **Step 5: Frontend Feature Implementations**

### **1. Trade Modal**
- File: `components/TradeModal.vue`
```vue
<template>
  <div class="modal">
    <h2>Trade Items</h2>
    <div class="inventory">
      <h3>Your Inventory</h3>
      <ul>
        <li v-for="item in userInventory" :key="item">
          <label>
            <input type="checkbox" :value="item" v-model="selectedItems" />
            {{ item }}
          </label>
        </li>
      </ul>
    </div>
    <div class="actions">
      <button @click="submitTrade">Propose Trade</button>
      <button @click="$emit('close')">Cancel</button>
    </div>
  </div>
</template>

<script>
import { useAuthStore } from "../stores/authStore";

export default {
  props: ["receiver"],
  data() {
    return {
      selectedItems: [],
    };
  },
  computed: {
    userInventory() {
      const authStore = useAuthStore();
      return authStore.user.inventory;
    },
  },
  methods: {
    async submitTrade() {
      try {
        const response = await this.$axios.post("/api/trades", {
          senderId: this.$authStore.user.id,
          receiverId: this.receiver.id,
          items: this.selectedItems,
        });
        console.log("Trade successful:", response.data);
        this.$emit("close");
      } catch (error) {
        console.error("Trade failed:", error.response.data);
      }
    },
  },
};
</script>

<style scoped>
.modal {
  /* Modal styles */
}
</style>
```

---

### **2. Admin Page**
- File: `views/AdminPage.vue`
```vue
<template>
  <div>
    <h1>Admin Dashboard</h1>
    <h2>Active Users</h2>
    <ul>
      <li v-for="user in onlineUsers" :key="user.id">{{ user.username }}</li>
    </ul>
    <h2>Trade Logs</h2>
    <ul>
      <li v-for="trade in trades" :key="trade.id">
        {{ trade.sender.username }} traded with {{ trade.receiver.username }}: {{ trade.items.join(", ") }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      onlineUsers: [],
      trades: [],
    };
  },
  async mounted() {
    try {
      const [usersRes, tradesRes] = await Promise.all([
        this.$axios.get("/api/online-users"),
        this.$axios.get("/api/trades"),
      ]);
      this.onlineUsers = usersRes.data;
      this.trades = tradesRes.data;
    } catch (error) {
      console.error("Failed to fetch admin data:", error.response.data);
    }
  },
};
</script>
```

---

