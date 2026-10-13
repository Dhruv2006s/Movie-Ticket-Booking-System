# Reelgood — movie booking

A React + Express movie discovery and booking app. The interface includes searchable and filterable films, saved titles, showtime selection, interactive seat selection, booking confirmation and MongoDB backed reviews.

PostgreSQL is the primary store for customer accounts, bookings, and saved films. If PostgreSQL is unavailable but MongoDB is connected, those account records and bookings persist in MongoDB instead (`MONGODB_DB`, default `reelgood`). Registration stores a salted scrypt password hash, and sign-in uses a signed, seven-day, HttpOnly session cookie. Bookings and saved films are attached to the authenticated account. Customers can see only their own booking history and saved list.

## Polyglot data architecture

- **PostgreSQL** is the preferred store for accounts, bookings, and seat reservations. A relational database provides transactions, constraints, and the consistency needed to prevent duplicate reservations.
- **MongoDB** stores movie catalog documents and customer reviews, whose metadata and review content can evolve independently.
- **Redis** caches the movie catalog for five minutes. It is an optimization; the API continues to work if Redis is offline.

The API seeds a small starter film catalog in MongoDB. If PostgreSQL is offline, account and booking data use MongoDB and remain persistent. The store is chosen once at startup and stays consistent across restarts; `ACCOUNT_STORE=postgres` or `ACCOUNT_STORE=mongodb` can pin it explicitly. If MongoDB is offline, movie browsing uses the bundled catalog. Reviews require MongoDB.

## Run locally

1. Install Node.js and Docker Desktop.
2. Copy `.env.example` to `.env`.
   Set `JWT_SECRET` to a long random secret. The requested local admin login is `Admin` / `Admin`; change `ADMIN_PASSWORD` before sharing or deploying the site.
3. Start the databases with `docker compose up -d`.
4. Install packages with `npm install`.
5. Start the app with `npm run dev`.
6. Open `http://localhost:5173`.

The API listens on port 4000. The frontend proxies `/api` requests to it.

## Admin portal

Open `http://localhost:5173/admin` and sign in with `ADMIN_USERNAME` and `ADMIN_PASSWORD` from `.env` (the requested local demo credentials are `Admin` / `Admin`). The dashboard reports booking counts and ticket counts per movie, each reservation's date and showtime, customer and seat details, ticket sales, and the latest 50 bookings. Admin access uses a separate signed HttpOnly session cookie. Change the demo password before sharing or deploying the site.

## API

- `GET /api/health`
- `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/logout`, `GET /api/auth/me`
- `GET /api/account/bookings`, `PUT /api/account/favorites/:movieId` (signed in)
- `POST/GET /api/admin/session`, `POST /api/admin/logout`, `GET /api/admin/metrics` (admin session required)
- `GET /api/movies`, `GET /api/movies/:id`
- `GET /api/shows/:movieId/:date`
- `GET /api/seats/:movieId/:date/:time`
- `POST /api/bookings`
- `GET/POST /api/movies/:id/reviews`

The demo uses a fixed ticket price. Payment processing, email verification, and password reset are not configured. Before public deployment, connect a payment provider and transactional email service, configure HTTPS and production secrets, and review the booking policy for cancellations and refunds. Seat holds and reservations are checked under a PostgreSQL transaction.
