
# Guide to Cookie Authentication in Next.js

This guide provides a comprehensive overview of handling cookie authentication for login and signup processes, as well as checking authentication status during page render.

## Step 1: Install Dependencies

Ensure you have the following dependencies installed in your project:

```bash
npm install bcrypt bcryptjs dotenv js-cookie jsonwebtoken pg mongoose
```

- **bcrypt** and **bcryptjs**: For hashing passwords.
- **dotenv**: For environment variable management.
- **js-cookie**: For handling cookies in the frontend.
- **jsonwebtoken**: For creating and verifying JWT tokens.
- **jwt-decode**: For decoding JWT tokens.
- **pg**: For PostgreSQL database interaction.
- **mongoose**: For MongoDB database interaction.

## Folder Setup

Create the following structure in your project root:

```
root/
├── db.js         # Database query setup for PostgreSQL or Mongoose
└── .env          # Environment variable configuration
```

Create the API routes for authentication:

```
app/api/auth/
├── checkAuth.js
├── login.js
└── signup.js
```

## Step 2: Implement API Routes

### Check Authentication (checkAuth.js)

```javascript
import { NextResponse } from "next/server";
import dotenv from "dotenv";
import jwt from "jsonwebtoken";
import { cookies } from "next/headers";

dotenv.config();

const validateToken = (token) => {
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);  // Verify JWT token is done this way using .verify
    return Boolean(decoded);
  } catch (error) {
    console.error("Token validation error:", error);
    return false;
  }
};

export async function GET(req) {
  const cookieStore = await cookies();  // Get cookies from headers in nextjs in this way
  const token = (cookieStore.get("token"))?.value; // Retrieve token

  if (!token) {
    return NextResponse.json({ message: "Unauthorized" }, { status: 401 });
  }

  if (validateToken(token)) { // Validate token
    return NextResponse.json({ message: "Authorized" }, { status: 200 });
  } else {
    return NextResponse.json({ message: "Unauthorized" }, { status: 401 });
  }
}
```

### Login (login.js)

```javascript
import { NextResponse } from "next/server";
import { query } from "@/db";  // Adjust import for your DB setup
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import dotenv from "dotenv";

dotenv.config();

export async function POST(req) {
  const { email, password } = await req.json();

  if (!email || !password) {
    return NextResponse.json(
      { message: "Email and password are required" },
      { status: 400 }
    );
  }

  try {
    const result = await query("SELECT * FROM users WHERE email = $1", [email]);
    const user = result.rows[0]; // Get user from query result

    if (!user) {
      return NextResponse.json({ message: "No user found" }, { status: 404 });
    }

    const isPasswordValid = bcrypt.compareSync(password, user.password); // Validate password
    if (!isPasswordValid) {
      return NextResponse.json({ message: "Invalid password" }, { status: 401 });
    }

    const token = jwt.sign({ id: user.id }, process.env.JWT_SECRET, { expiresIn: "1h" }); // Create JWT token using JWT.sign

    const response = NextResponse.json({ status: 200, data: { token } });
    response.cookies.set("token", token, { maxAge: 60 * 60 }); // Set cookie to senmd to frontend

    return response;
  } catch (error) {
    return NextResponse.json({ message: "Internal server error", error: error.message }, { status: 500 });
  }
}
```

### Signup (signup.js)

```javascript
import { NextResponse } from "next/server";
import { query } from "@/db";  // Adjust import for your DB setup
import bcrypt from "bcrypt";
import jwt from "jsonwebtoken";
import dotenv from "dotenv";

dotenv.config();

export async function POST(req) {
  const { email, password } = await req.json();

  if (!email || !password) {
    return NextResponse.json({
      status: 400,
      message: "Email and password are required",
    });
  }

  try {
    const hashedPassword = bcrypt.hashSync(password, 10); // Hash password using bcrypt in this way
    const result = await query("INSERT INTO users (email, password) VALUES ($1, $2) RETURNING *", [email, hashedPassword]);
    const user = result.rows[0];

    if (!user) {
      return NextResponse.json({ status: 500, message: "Internal server error" });
    }

    const token = jwt.sign({ id: user.id }, process.env.JWT_SECRET, { expiresIn: "1h" }); // Create JWT token

    const response = NextResponse.json({ status: 200, data: { token } });
    response.cookies.set("token", token, { maxAge: 60 * 60 }); // Set cookie

    return response;
  } catch (error) {
    return NextResponse.json({ status: 500, message: "Internal server error" });
  }
}
```

## Step 3: Frontend Integration

### Handling Login

In your login function:

```javascript
const handleLogin = async (email, password) => {
  const res = await fetch("/api/auth/login", {
    method: "POST",
    body: JSON.stringify({ email, password }),
    headers: {
      "Content-Type": "application/json",
    },
  });

  const data = await res.json();

  if (res.ok) {
    toast.success("Logged in successfully");
    Cookies.set("token", data.data.token, { expires: 1, path: "/" }); // Do not forget to import 'Cookies' from 'js-cookie' after that you s=can store the cookie in the browser as this example
    router.push("/"); // Redirect after login
  } else {
    toast.error(data.message);
  }
};
```

### Handling Signup

Use the same cookie setting logic in your signup function:

```javascript
const handleSignup = async (email, password) => {
  const res = await fetch("/api/auth/signup", {
    method: "POST",
    body: JSON.stringify({ email, password }),
    headers: {
      "Content-Type": "application/json",
    },
  });

  const data = await res.json();

  if (res.ok) {
    toast.success("Signed up successfully");
    Cookies.set("token", data.data.token, { expires: 1, path: "/" }); // Do not forget to import 'Cookies' from 'js-cookie' after that you s=can store the cookie in the browser as this example
    router.push("/"); // Redirect after signup
  } else {
    toast.error(data.message);
  }
};
```

### Checking Authentication Status

To check the authentication status of the user on every page:

```javascript
(async function checkAuth() {
  const res = await fetch("/api/auth/checkAuth", {
    method: "GET",
    credentials: "include", // Send cookies with request
  });

  const data = await res.json();

  if (data.message === "Authorized") {
    router.push("/"); // Redirect if authorized
  }

  return;
})();
```

### Handling Logout

For logout functionality:

```javascript
const handleLogout = async () => {
  const cookie = Cookies.get("token");

  if (!cookie) {
    return;
  }

  Cookies.remove("token"); // Remove cookie
  toast.success("Logged out successfully");
};
```

## Notes

- You can customize the JWT expiration time and cookie properties as per your application's requirements.
- The code examples for MongoDB and PostgreSQL are provided as is; ensure your database queries match your schema.
- Always handle sensitive data securely and follow best practices for authentication and authorization.
