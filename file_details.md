## **Backend Files**

### **1. `config/db.js`**
- **Purpose**: Establishes and manages the connection to the MongoDB Atlas database.
- **Interactions**:
  - `server.js` uses it to connect to MongoDB at application startup.
  - Relies on `.env` for the `MONGO_URI`.
- **Directory**: `config/`
  - This directory centralizes configuration files, keeping environment-specific logic separate from core application logic.

---

### **2. `middlewares/authMiddleware.js`**
- **Purpose**: Validates incoming requests using Auth0 JWTs, ensuring users are authenticated.
- **Interactions**:
  - Used by route files (e.g., `userRoutes.js`, `tradeRoutes.js`) to protect endpoints.
  - Relies on Auth0 credentials stored in `.env`.
- **Directory**: `middlewares/`
  - Middleware files are placed here to group reusable logic that processes requests before they reach controllers.

---

### **3. `middlewares/errorMiddleware.js`**
- **Purpose**: Centralized error-handling middleware to catch and format errors.
- **Interactions**:
  - Invoked automatically when a route or controller throws an error.
  - Used by `server.js`.
- **Directory**: `middlewares/`
  - Logical grouping of middleware components.

---

### **4. `models/user.js`**
- **Purpose**: Defines the schema for users, including fields like `username`, `email`, `inventory`, and `pets`.
- **Interactions**:
  - Used by `userController.js` and `tradeController.js` to query or update user data.
  - References `pets` from the `Pet` schema.
- **Directory**: `models/`
  - Models are placed here to separate database schema definitions from other logic.

---

### **5. `models/pet.js`**
- **Purpose**: Defines the schema for pets, including `name`, `type`, and `stats`.
- **Interactions**:
  - Referenced by `User` schema via `pets` array.
  - Used in `petController.js` for CRUD operations.
- **Directory**: `models/`
  - Schema definitions are centralized for maintainability.

---

### **6. `models/trade.js`**
- **Purpose**: Defines the schema for trades, including `sender`, `receiver`, `items`, and `timestamp`.
- **Interactions**:
  - Used by `tradeController.js` to log trades.
  - References `User` schema for `sender` and `receiver`.
- **Directory**: `models/`
  - Follows the same pattern as other schema definitions.

---

### **7. `controllers/userController.js`**
- **Purpose**: Implements the logic for user-related operations (e.g., fetch, create).
- **Interactions**:
  - Interacts with the `User` model for database queries.
  - Routes in `userRoutes.js` invoke its methods.
- **Directory**: `controllers/`
  - Controllers encapsulate application logic, separating it from routing concerns.

---

### **8. `controllers/petController.js`**
- **Purpose**: Manages CRUD operations for pets (e.g., adding/removing pets).
- **Interactions**:
  - Interacts with the `Pet` model and updates the `User` model's `pets` field.
  - Invoked by `petRoutes.js`.
- **Directory**: `controllers/`
  - Centralizes pet-related application logic.

---

### **9. `controllers/tradeController.js`**
- **Purpose**: Implements the logic for user-to-user trades.
- **Interactions**:
  - Uses `Trade`, `User`, and `Pet` models to validate and log trades.
  - Routes in `tradeRoutes.js` invoke its methods.
- **Directory**: `controllers/`
  - Focuses on trade-related operations, keeping logic modular.

---

### **10. `routes/userRoutes.js`**
- **Purpose**: Defines API endpoints for user-related operations (e.g., `GET /api/users`, `POST /api/users`).
- **Interactions**:
  - Invokes methods from `userController.js`.
  - Uses `authMiddleware.js` to protect sensitive routes.
- **Directory**: `routes/`
  - Grouped here to separate endpoint definitions from core application logic.

---

### **11. `routes/petRoutes.js`**
- **Purpose**: Defines API endpoints for pet-related operations.
- **Interactions**:
  - Invokes methods from `petController.js`.
  - Uses `authMiddleware.js` to protect endpoints.
- **Directory**: `routes/`
  - Keeps routing logic modular and isolated.

---

### **12. `routes/tradeRoutes.js`**
- **Purpose**: Defines API endpoints for trade-related operations.
- **Interactions**:
  - Invokes methods from `tradeController.js`.
  - Uses `authMiddleware.js` to ensure only authenticated users can trade.
- **Directory**: `routes/`
  - Same reasoning as above.

---

### **13. `utils/logger.js`**
- **Purpose**: Provides utility functions for logging user and trade activities.
- **Interactions**:
  - Used by controllers like `tradeController.js` to log actions.
- **Directory**: `utils/`
  - Utility files are placed here to group reusable, cross-cutting concerns.

---

### **14. `server.js`**
- **Purpose**: Entry point for the backend application. Sets up middleware, connects to MongoDB, and defines routes.
- **Interactions**:
  - Initializes `db.js` to connect to MongoDB.
  - Registers routes from `routes/`.
  - Uses `authMiddleware.js` and `errorMiddleware.js`.
- **Directory**: Root directory.
  - Positioned here as the primary script to start the backend server.

---

## **Frontend Files**

### **1. `main.js`**
- **Purpose**: Entry point for the Vue app. Configures global dependencies (e.g., Pinia, Router) and renders the root component.
- **Interactions**:
  - Imports `App.vue`, `router/index.js`, and `stores/`.
- **Directory**: Root of `src/`.
  - Positioned here as the standard entry for Vue apps.

---

### **2. `App.vue`**
- **Purpose**: Root component that serves as the layout wrapper for the application.
- **Interactions**:
  - Includes `<router-view>` to render routed components.
  - Uses global styles from `styles/tailwind.css`.
- **Directory**: Root of `src/`.
  - Vue convention for the root layout component.

---

### **3. `router/index.js`**
- **Purpose**: Configures Vue Router for navigation between views.
- **Interactions**:
  - Registers views like `Dashboard.vue` and `AdminPage.vue`.
- **Directory**: `src/router/`
  - Logical separation of navigation logic.

---

### **4. `stores/authStore.js`**
- **Purpose**: Manages authentication state (e.g., user info, token).
- **Interactions**:
  - Accessed by components like `TradeModal.vue` for user-related data.
- **Directory**: `src/stores/`
  - Grouped with other global state management files.

---

### **5. `components/TradeModal.vue`**
- **Purpose**: Handles the UI for proposing trades.
- **Interactions**:
  - Uses `authStore.js` for current user data.
  - Submits trade requests to the backend via Axios.
- **Directory**: `src/components/`
  - Reusable UI component logic.

---

### **6. `views/Dashboard.vue`**
- **Purpose**: Main user dashboard, displaying pets, inventory, and trade options.
- **Interactions**:
  - Fetches user data via `authStore.js`.
  - Renders `PetList.vue` and `Inventory.vue` components.
- **Directory**: `src/views/`
  - High-level view tied to routing.

---

### **7. `views/AdminPage.vue`**
- **Purpose**: Displays admin-exclusive logs for trades and online users.
- **Interactions**:
  - Fetches logs from the backend via Axios.
- **Directory**: `src/views/`
  - High-level view for admin-specific content.

---

### **8. `utils/api.js`**
- **Purpose**: Centralizes Axios configuration for making API requests.
- **Interactions**:
  - Used by components like `TradeModal.vue` and views like `AdminPage.vue`.
- **Directory**: `src/utils/`
  - Keeps API logic modular and reusable.

---