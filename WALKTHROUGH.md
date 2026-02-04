# AstroTech Wealth - Codebase Walkthrough

This document provides a high-level overview of the AstroTech Wealth application, explaining the architecture, key directories, and core functionalities of both the Backend and Frontend.

## 🏗 Project Structure

The project is structured as a **monorepo** containing both the frontend and backend applications.

```
/
├── Backend/                 # Python FastAPI Backend
├── Frontend/                # React + Vite Frontend
├── Procfile                 # Start command for Railway deployment
├── requirements.txt         # Root dependencies for Railway detection
└── ... (Deployment docs)
```

---

## 🐍 Backend (`/Backend`)

Built with **FastAPI**, the backend handles authentication, database interactions (MongoDB), payment processing, and the core astrology logic.

### Key Files & Directories

*   **`main.py`**: The entry point of the application.
    *   Initializes `FastAPI`.
    *   Configures **CORS** to allow requests from the frontend (Vercel/Local).
    *   Defines API routes: `/signup`, `/token` (login), `/create-order` (payment), `/consultation`, and `/generate-report`.
*   **`database.py`**: Manages the **MongoDB** connection using `motor` (AsyncIO).
    *   Connects to the database specified in `MONGO_URI`.
    *   Exports collections: `users`, `transactions`, `consultations`.
*   **`auth.py`**: Handles User Authentication.
    *   **JWT** (JSON Web Token) generation and verification.
    *   Password hashing using `bcrypt` (specifically version 3.2.2 for compatibility).
*   **`payment.py`**: Integrates **Razorpay** for payment processing.
    *   Creates payment orders.
    *   Verifies payment signatures to ensure transaction security.
*   **`schemas.py`**: Defines **Pydantic models** for data validation.
    *   Ensures consistent data structures for Users, Tokens, Orders, and Consultations.
*   **`core/`**: logic for `report_service`.

### Core Dependencies
*   `fastapi`, `uvicorn`: Web server.
*   `motor`: Async MongoDB driver.
*   `razorpay`: Payment gateway.
*   `passlib[bcrypt]`, `python-jose`: Security & Auth.

---

## ⚛️ Frontend (`/Frontend`)

Built with **React** and **Vite**, styled with **Tailwind CSS**, and animated with **Framer Motion**.

### Key Directories

*   **`src/components/`**: options for UI components.
    *   **`Hero.jsx`**: The main landing section. Features a split layout (Text + Astrological Wheel) with responsive design adjustments for tablets.
    *   **`Navbar.jsx`**: Responsive navigation bar. Handles login/logout state and mobile menu toggling.
    *   **`Features.jsx`**: Interactive grid of Zodiac signs. Includes a modal with detailed astrological data.
    *   **`Pricing.jsx`**: Displays subscription plans. Integrates directly with Razorpay logic to initiate payments.
    *   **`BookConsultancy.jsx`** & **`ConsultationForm.jsx`**: Forms for booking expert sessions.
*   **`src/services/`**:
    *   **`api.js`**: Centralized **Axios** configuration.
        *   Manages the base URL (points to Railway in production).
        *   Interceptors automatically attach the JWT token to requests.
        *   Handles global error responses.

### Styling & Animation
*   **Tailwind CSS**: Utility-first styling for rapid UI development (`index.css`).
*   **Framer Motion**: Used extensively for entry animations (`fadeInUp`), hover effects, and modal transitions.

---

## 🚀 Deployment

### Backend (Railway)
*   **Method**: Deployed via GitHub integration.
*   **Configuration**:
    *   **`requirements.txt` (Root)**: Included to force Railway to detect a Python project.
    *   **`Procfile`**: Commands Railway to `cd Backend` and start `uvicorn`.
*   **Environment Variables**: `MONGO_URI`, `SECRET_KEY`, `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`.

### Frontend (Vercel)
*   **Method**: Deployed via Vercel Git integration.
*   **Configuration**:
    *   **`VITE_API_URL`**: Set to the Railway backend URL to ensure proper communication.

---

## 🔄 Data Flow Example: Payment

1.  **User** clicks "Choose Plan" in `Pricing.jsx`.
2.  **Frontend** calls `api.createOrder` -> **Backend** (`/create-order`).
3.  **Backend** talks to Razorpay API, creates an order, saves it to MongoDB, returns Order ID.
4.  **Frontend** opens Razorpay Checkout Modal.
5.  **User** pays.
6.  **Frontend** sends payment signature to **Backend** (`/verify-payment`).
7.  **Backend** verifies signature, updates transaction status in MongoDB, and returns success.
