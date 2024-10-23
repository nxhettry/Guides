Here’s a well-structured and styled version of your README, incorporating comments for clarity and implementing `checkAuth` as middleware:

```markdown
# Next.js Frontend with Node.js Backend

This project is a full-stack application featuring a Next.js frontend and a Node.js backend. It includes user authentication functionalities, such as login and signup, utilizing JWT for token-based authentication.

## Project Structure

Here's an overview of the project structure:

```
/project-root
│
├── /frontend             # Next.js Frontend
│   ├── /components       # Reusable components
│   ├── /pages            # Next.js pages
│   ├── /styles           # CSS styles
│   └── /utils            # Utility functions
│
├── /backend              # Node.js Backend
│   ├── /controllers      # Controllers for handling requests
│   ├── /routes           # API routes
│   ├── /middlewares       # Middleware for authentication and error handling
│   ├── /models           # Database models
│   ├── /config           # Configuration files (e.g., database connection)
│   └── server.js         # Main entry point for the Express server
│
└── .env                  # Environment variables
```

## Installation

### Prerequisites

Ensure you have the following installed on your machine:

- Node.js
- npm or yarn
- PostgreSQL (or your preferred database)

### Setup

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd project-root
   ```

2. **Install dependencies for both frontend and backend:**
   ```bash
   # Navigate to the frontend
   cd frontend
   npm install

   # Navigate to the backend
   cd ../backend
   npm install
   ```

3. **Create an `.env` file in the `/backend` directory with the following content:**
   ```env
   JWT_SECRET=your_jwt_secret
   DATABASE_URL=your_database_connection_string
   ```

## Frontend Logic

The frontend handles user authentication through the following functions:

### Login Function

This function handles user login. It validates input fields and sends a login request to the backend.

```javascript
const handleLogin = async (event) => {
    event.preventDefault(); // Prevent the default form submission
    const formData = new FormData(form.current); // Get form data
    const email = formData.get("email");
    const password = formData.get("password");

    // Validate email and password
    if (!email || !password) {
        toast.error("Please fill all the fields");
        return;
    }

    // Send a POST request to the login API
    const res = await fetch("http://localhost:<serverPort>/api/auth/login", {
        method: "POST",
        body: JSON.stringify({ email, password }),
        headers: {
            "Content-Type": "application/json",
        },
    });

    const data = await res.json();

    // Handle response from the server
    if (res.ok) {
        toast.success("Logged in successfully");
        Cookies.set("token", data.data.token, { expires: 1, path: "/" });
        router.push("/"); // Redirect to the homepage
    } else {
        toast.error(data.message); // Show error message
    }
};
```

### Signup Function

This function handles user registration. It validates input fields and sends a signup request to the backend.

```javascript
const handleSignup = async (event) => {
    event.preventDefault(); // Prevent the default form submission
    const formData = new FormData(form.current); // Get form data
    const email = formData.get("email");
    const password = formData.get("password");

    // Validate email and password
    if (!email || !password) {
        toast.error("Please fill all the fields");
        return;
    }

    // Send a POST request to the signup API
    const res = await fetch("http://localhost:<serverPort>/api/auth/signup", {
        method: "POST",
        body: JSON.stringify({ email, password }),
        headers: {
            "Content-Type": "application/json",
        },
    });

    const data = await res.json();

    // Handle response from the server
    if (res.ok) {
        toast.success("Signed up successfully");
        Cookies.set("token", data.data.token, { expires: 1 });
        router.push("/"); // Redirect to the homepage
    } else {
        toast.error(data.message); // Show error message
    }
};
```

### Check Authentication Function

This function checks if the user is authenticated. It redirects to the homepage if authenticated.

```javascript
(async function checkAuth() {
    const res = await fetch("http://localhost:<serverPort>/api/auth/checkAuth", {
        method: "GET",
        credentials: "include",
    });

    const data = await res.json();

    // Redirect if the user is authorized
    if (data.message === "Authorized") {
        router.push("/");
    } 
})();
```

## Backend Setup

The backend is built with Express.js and handles the authentication logic in separate controllers.

### Install Express.js

Navigate to the `/backend` directory and install Express:

```bash
npm install express dotenv jsonwebtoken bcrypt
```

### Server Setup

Create `server.js` in the `/backend` directory:

```javascript
import express from 'express';
import dotenv from 'dotenv';
import authRoutes from './routes/auth.js'; // Importing authentication routes
import checkAuth from './middlewares/checkAuth.js'; // Importing authentication middleware

dotenv.config();

const app = express();
app.use(express.json()); // Middleware to parse JSON bodies

// Middleware to check authentication on protected routes
app.use(checkAuth); // Apply checkAuth middleware to protect routes

// API Routes
app.use('/api/auth', authRoutes); // Authentication routes

// Start the server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
```

### Middleware Setup

Create a new file `checkAuth.js` in the `/backend/middlewares` directory:

```javascript
import jwt from 'jsonwebtoken';

const checkAuth = (req, res, next) => {
    const token = req.cookies.token; // Get token from cookies

    // Check if token exists
    if (!token) {
        return res.status(401).json({ message: "Unauthorized" });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET); // Verify token
        req.user = decoded; // Attach user information to request object
        next(); // Proceed to the next middleware or route handler
    } catch (error) {
        return res.status(401).json({ message: "Unauthorized" });
    }
};

export default checkAuth;
```

### Routes Setup

Create a new file `auth.js` in the `/backend/routes` directory:

```javascript
import express from 'express';
import { login, signup, checkAuth } from '../controllers/authController.js';

const router = express.Router();

// Define routes for authentication
router.post('/login', login);         // Login route
router.post('/signup', signup);       // Signup route
router.get('/checkAuth', checkAuth); // Check authentication route

export default router;
```

### Controllers Setup

Create `authController.js` in the `/backend/controllers` directory:

```javascript
import { query } from "@/db"; // Assume you have a query function for database operations
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import dotenv from "dotenv";

dotenv.config();

// Check authentication status
export const checkAuth = async (req, res) => {
    const token = req.cookies.token; // Get token from cookies

    // Validate token existence
    if (!token) {
        return res.status(401).json({ message: "Unauthorized" });
    }

    try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET); // Verify token
        return res.status(200).json({ message: "Authorized", user: decoded }); // Return user info
    } catch (error) {
        return res.status(401).json({ message: "Unauthorized" });
    }
};

// Login user
export const login = async (req, res) => {
    const { email, password } = req.body; // Destructure email and password from request body

    // Validate email and password
    if (!email || !password) {
        return res.status(400).json({ message: "Email and password are required" });
    }

    try {
        const result = await query("SELECT * FROM users WHERE email = $1", [email]);
        const user = result.rows[0];

        // Check if user exists
        if (!user) {
            return res.status(404).json({ message: "No user found" });
        }

        // Validate password
        const isPasswordValid = bcrypt.compareSync(password, user.password);
        if (!isPasswordValid) {
            return res.status(401).json({ message: "Invalid password" });
        }

        // Generate JWT token
        const token = jwt.sign({ id: user.id }, process.env.JWT_SECRET, { expiresIn: "1h" });
        res.cookie("token", token, { maxAge: 60 * 60 * 1000 }); // Set cookie with token

        return res.status(200).json({ data: { token } }); // Return token to client
    } catch (error) {
        return res.status(500).json({ message: "Internal server error", error: error.message });
    }
};

// Signup user
export const signup = async (req, res) => {
    const {

 email, password } = req.body; // Destructure email and password from request body

    // Validate email and password
    if (!email || !password) {
        return res.status(400).json({ message: "Email and password are required" });
    }

    try {
        // Hash the password before saving
        const hashedPassword = await bcrypt.hash(password, 10);
        const result = await query("INSERT INTO users (email, password) VALUES ($1, $2) RETURNING *", [email, hashedPassword]);
        const user = result.rows[0];

        // Generate JWT token
        const token = jwt.sign({ id: user.id }, process.env.JWT_SECRET, { expiresIn: "1h" });
        res.cookie("token", token, { maxAge: 60 * 60 * 1000 }); // Set cookie with token

        return res.status(201).json({ data: { token } }); // Return token to client
    } catch (error) {
        return res.status(500).json({ message: "Internal server error", error: error.message });
    }
};
```

## Running the Application

1. **Start the backend server:**
   ```bash
   cd backend
   node server.js
   ```

2. **Start the frontend development server:**
   ```bash
   cd frontend
   npm run dev
   ```

The application should now be running on `http://localhost:3000` (Next.js) and `http://localhost:<serverPort>` (Express). You can navigate to the respective URLs to access the frontend and backend.

## Contributing

Contributions are welcome! Please submit a pull request or open an issue for any changes or improvements.

## License

This project is licensed under the MIT License.
```

### Notes:
- This README includes clear sections on installation, project structure, frontend and backend setup, and usage.
- Ensure that `<repository-url>` and `<serverPort>` are replaced with your actual repository URL and server port number respectively.
- Adjust any specific routes, fields, or logic based on your actual application structure and requirements.
