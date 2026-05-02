# Natours

A full-stack tour booking web application built with Node.js, Express, and MongoDB. Users can browse nature tours, read and write reviews, and book tours with secure online payments.

---

## Table of Contents

- [Features](#features)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Scripts](#scripts)
- [Security](#security)

---

## Features

- **Tour Browsing** — View all available tours with details like duration, difficulty, group size, price, locations, and guides
- **User Authentication** — JWT-based signup, login, and logout with secure HTTP-only cookies
- **Role-Based Access Control** — Four roles: `user`, `guide`, `lead-guide`, and `admin` with protected routes per role
- **Password Management** — Forgot password / reset password flow via email token
- **Reviews & Ratings** — Authenticated users can create, update, and delete reviews on tours; average ratings are auto-calculated
- **Stripe Payments** — Secure checkout sessions via Stripe for tour bookings
- **Email Notifications** — Welcome email on signup and password reset emails via Nodemailer (Mailtrap in dev, SendGrid in production)
- **Image Upload** — Users can upload a profile photo (handled with Multer)
- **Geospatial Queries** — Find tours within a given radius and calculate distances from a coordinate
- **Advanced API Querying** — Filtering, sorting, field selection, and pagination on all list endpoints
- **Server-Side Rendering** — Pug-powered views for the overview, tour detail, account, login, and signup pages

---

## Tech Stack

| Layer            | Technology                         |
| ---------------- | ---------------------------------- |
| Runtime          | Node.js (ES Modules)               |
| Framework        | Express.js                         |
| Database         | MongoDB + Mongoose                 |
| Templating       | Pug                                |
| Authentication   | JSON Web Tokens (JWT) + bcrypt     |
| Payments         | Stripe                             |
| Email            | Nodemailer (dev) / SendGrid (prod) |
| File Uploads     | Multer                             |
| Frontend Bundler | Parcel                             |

---

## Project Structure

```
├── app.js                  # Express app setup and global middleware
├── server.js               # Server entry point and DB connection
├── controllers/
│   ├── auth-controller.js  # Signup, login, logout, password reset, route protection
│   ├── tours-controller.js # Tour CRUD + geospatial endpoints
│   ├── users-controller.js # User profile management (admin + self)
│   ├── review-controller.js
│   ├── booking-controller.js # Stripe checkout session + booking creation
│   ├── view-controller.js  # Server-side rendered page handlers
│   ├── handlerFactory.js   # Generic CRUD factory functions
│   └── error-controller.js # Global error handler
├── models/
│   ├── tours-model.js      # Tour schema with GeoJSON, virtuals, and slug
│   ├── users-model.js      # User schema with password hashing
│   ├── reviews-model.js    # Review schema with auto average rating calculation
│   └── booking-models.js   # Booking schema
├── routes/
│   ├── tours-route.js
│   ├── users-route.js
│   ├── reviews-route.js
│   ├── booking-route.js
│   └── view-route.js
├── middleware/
│   └── uploadImages.js     # Multer configuration for user photo uploads
├── utils/
│   ├── apiFeatures.js      # APIQuery class: filter, sort, fields, pagination
│   ├── appError.js         # Custom operational error class
│   ├── catchAsync.js       # Async error wrapper
│   ├── email.js            # Email class using Nodemailer + Pug templates
│   └── sendResponse.js     # Unified JSON response helper
├── views/                  # Pug templates for all rendered pages
│   └── email/              # Email-specific Pug templates
└── public/                 # Static assets (CSS, JS, images)
```

---

## API Reference

All API endpoints are prefixed with `/api/v1`.

### Tours — `/api/v1/tours`

| Method | Endpoint                        | Access            | Description                             |
| ------ | ------------------------------- | ----------------- | --------------------------------------- |
| GET    | `/`                             | Protected         | Get all tours                           |
| POST   | `/`                             | Public            | Create a new tour                       |
| GET    | `/:id`                          | Public            | Get a single tour                       |
| PATCH  | `/:id`                          | Public            | Update a tour                           |
| DELETE | `/:id`                          | Admin, Lead-Guide | Delete a tour                           |
| GET    | `/distances/:latlng/unit/:unit` | Public            | Get distances from a point to all tours |
| GET    | `/:tourId/reviews`              | Protected         | Get all reviews for a tour              |
| POST   | `/:tourId/reviews`              | Protected         | Create a review for a tour              |

### Users — `/api/v1/users`

| Method             | Endpoint                 | Access    | Description                         |
| ------------------ | ------------------------ | --------- | ----------------------------------- |
| POST               | `/signup`                | Public    | Register a new user                 |
| POST               | `/login`                 | Public    | Log in                              |
| GET                | `/logout`                | Public    | Log out                             |
| POST               | `/forget-password`       | Public    | Send password reset email           |
| PATCH              | `/reset-password/:token` | Public    | Reset password with token           |
| PATCH              | `/update-password`       | Protected | Change current password             |
| PATCH              | `/updateMe`              | Protected | Update own profile (+ photo upload) |
| DELETE             | `/deleteMe`              | Protected | Deactivate own account              |
| GET                | `/me`                    | Protected | Get own profile                     |
| GET                | `/`                      | Admin     | Get all users                       |
| GET, PATCH, DELETE | `/:id`                   | Admin     | Manage any user                     |

### Reviews — `/api/v1/reviews`

| Method | Endpoint | Access    | Description     |
| ------ | -------- | --------- | --------------- |
| GET    | `/`      | Protected | Get all reviews |
| POST   | `/`      | Protected | Create a review |
| GET    | `/:id`   | Protected | Get a review    |
| PATCH  | `/:id`   | Protected | Update a review |
| DELETE | `/:id`   | Protected | Delete a review |

### Bookings — `/api/v1/booking`

| Method | Endpoint                    | Access    | Description                      |
| ------ | --------------------------- | --------- | -------------------------------- |
| GET    | `/checkout-session/:tourId` | Protected | Create a Stripe checkout session |

---

## Getting Started

### Prerequisites

- Node.js v18+
- MongoDB instance (local or [MongoDB Atlas](https://www.mongodb.com/atlas))
- Stripe account
- A Mailtrap account (for development email testing)

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/0xmuhammed9/natours-website.git
cd natours-website

# 2. Install dependencies
npm install

# 3. Create a .env file and fill in the required variables (see below)

# 4. Start the development server
npm run dev
```

---

## Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
# Server
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=mongodb+srv://<user>:<password>@cluster.mongodb.net/natours

# JWT
JWT_SECRET=your-jwt-secret-key
JWT_EXPIRES_IN=90d
JWT_COOKIE_EXPIRES_IN=90

# Email (development — Mailtrap)
EMAIL_HOST=sandbox.smtp.mailtrap.io
EMAIL_PORT=2525
EMAIL_USERNAME=your-mailtrap-username
EMAIL_PASSWORD=your-mailtrap-password
EMAIL_FROM=noreply@natours.io

# Email (production — SendGrid)
SENDGRID_USERNAME=apikey
SENDGRID_PASSWORD=your-sendgrid-api-key

# Stripe
STRIPE_SECRETKEY=sk_test_your_stripe_secret_key
```

---

## Scripts

| Script             | Description                                   |
| ------------------ | --------------------------------------------- |
| `npm run dev`      | Start server in development mode with nodemon |
| `npm run prod`     | Start server in production mode with nodemon  |
| `npm run watch:js` | Watch and bundle frontend JS with Parcel      |
| `npm run build:js` | Build and bundle frontend JS for production   |

---

## Security

The application implements multiple layers of security:

- **Helmet** — Sets secure HTTP headers
- **Rate Limiting** — Max 100 requests per hour per IP on `/api` routes
- **Data Sanitization** — Prevents NoSQL injection (`express-mongo-sanitize`) and XSS attacks (`xss-clean`)
- **HPP** — Prevents HTTP parameter pollution with a whitelist for allowed duplicate query params
- **JWT in HTTP-only Cookies** — Tokens are never accessible via JavaScript
- **bcrypt** — Passwords are hashed with a cost factor of 10
- **CORS** — Configured to allow requests only from the expected origin
