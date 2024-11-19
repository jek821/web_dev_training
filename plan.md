### **Frontend: Vite + Tailwind + Axios + Pinia**

#### **File Structure**
- **`components/`**: Reusable components (e.g., Modals, Inventory Management).
- **`router/`**: Vue Router setup for navigation.
- **`stores/`**: Pinia state management (handles global and transient states).
- **`styles/`**: Custom Tailwind CSS configurations.
- **`utils/`**: Helper functions for things like formatting dates, parsing errors.
- **`views/`**: High-level views (e.g., Dashboard, Admin Page, Trade Screen).
- **`assets/`**: Static files (e.g., item icons, pet images).

#### **Core Features**
1. **User Authentication**:
   - Auth0 integration for login/logout.
   - Manage auth state in Pinia.
   - Conditional navigation for authenticated routes (e.g., `/dashboard`).

2. **Dashboard (Main User Page)**:
   - Display user-specific data:
     - Pet list with names, types, and stats.
     - Inventory list.
   - Buttons to:
     - Add new pets (opens modal).
     - View other users for trading.

3. **Trading System**:
   - Trading modal:
     - Shows available items from both users.
     - Uses transient state for proposals (e.g., item selection).
   - Backend endpoint to finalize trade:
     - Update both users’ inventories.
     - Log trade in the admin console.

4. **Admin Page**:
   - Accessible only to admin users.
   - Displays logs:
     - User-to-user trades.
     - Current online users (simulate with frontend pings to backend).

5. **Dynamic UI**:
   - Tailwind CSS for responsive design.
   - Transition effects for modals and state changes.

---

### **Backend: Node Express + MongoDB Atlas + Auth0**

#### **File Structure**
- **`config/`**: Database and Auth0 configuration.
- **`controllers/`**: Logic for routes (e.g., user, pet, trade).
- **`middlewares/`**: Auth and error-handling middleware.
- **`models/`**: MongoDB schemas.
- **`routes/`**: API endpoints.
- **`utils/`**: Utility functions (e.g., logging transactions).

#### **Data Models**
1. **User Schema**:
   - `username`: String, unique.
   - `email`: String, unique.
   - `passwordHash`: String.
   - `inventory`: Array of predefined items.
   - `pets`: Array of `Pet` references.
   - `role`: Enum (`'user' | 'admin'`).

2. **Pet Schema**:
   - `name`: String.
   - `type`: String (e.g., Dog, Cat).
   - `stats`: Object (e.g., happiness, health).

3. **Trade Schema**:
   - `sender`: User reference.
   - `receiver`: User reference.
   - `items`: Object showing items traded.

#### **Core Features**
1. **Authentication Middleware**:
   - Auth0 integration to decode JWT and set user roles.
   - Restrict admin routes.

2. **CRUD Operations**:
   - Users: Create, update, and delete.
   - Pets: Add, update, and remove pets.
   - Trades: Record user trades in MongoDB.

3. **Error Handling**:
   - Middleware for catching and formatting errors.
   - Handle invalid trades (e.g., insufficient items).

4. **Real-Time Updates**:
   - Track "online users" by recording pings every 30 seconds.
   - Use in-memory store (e.g., Redis or simple map) for online user states.

5. **Indexing for Performance**:
   - Create MongoDB indexes:
     - `email` for fast user lookup.
     - `role` for filtering users/admins.
     - Compound index on `sender` and `receiver` in the `Trade` collection.

---

### **State Management: Pinia**

1. **Global States**:
   - **Auth State**:
     - Manage current user (role, inventory).
     - Handle login/logout flow.
   - **Pet State**:
     - Store user’s pets and fetch on dashboard load.
   - **Trade State**:
     - Active trade proposals.
     - Past trade history.

2. **Transient States**:
   - Modals:
     - Track modal visibility and content (e.g., Add Pet, Trade Item).
   - Trading:
     - Selected items for trade.

3. **Advanced Patterns**:
   - Modular stores (e.g., `useAuthStore`, `usePetStore`, `useTradeStore`).
   - Pinia plugins for logging state changes.

---

### **Simulated Real-World Scenarios**
1. **Error Handling**:
   - Frontend: Toast notifications for API errors.
   - Backend: Return standardized error responses.

2. **Middleware**:
   - Authentication:
     - Validate tokens, set user roles.
   - Logging:
     - Log trade actions for admin review.

3. **User-to-User Interactions**:
   - Trade simulation:
     - Initiate trade proposals.
     - Finalize trades via backend.
   - Inventory updates:
     - Dynamically sync traded items in real time.

---