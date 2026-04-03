# ⚖ QMeasure — Quantity Measurement Frontend

A React-based frontend for performing unit conversions and quantity operations, backed by a Spring Boot REST API with JWT + OAuth2 authentication.

---

## Features

- **5 Operations** — Convert, Add, Subtract, Divide, and Compare quantities
- **4 Measurement Types** — Length, Weight, Volume, and Temperature
- **Auth** — Username/password login + GitHub OAuth2 (JWT-based)
- **Operation History** — View past operations (requires login), filterable by type
- **Public Access** — All operations work without logging in; history is protected

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | React 18 (Vite) |
| Routing | Manual page state (`App.jsx`) |
| Auth | JWT (localStorage) + GitHub OAuth2 |
| HTTP | Native `fetch` API (`api.js`) |
| Styling | Component-scoped CSS |

---

## Project Structure

```
src/
├── api.js              # Centralized API helper (fetch wrapper + auth header)
├── App.jsx             # Root component — page state & routing logic
├── main.jsx            # React DOM entry point
└── pages/
    ├── Dashboard.jsx   # Main calculator UI (public)
    ├── Dashboard.css
    ├── History.jsx     # Operation history (protected)
    ├── History.css
    ├── Login.jsx       # Login / Register + GitHub OAuth2
    └── Login.css
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- Spring Boot backend running on `http://localhost:8080`

### Install & Run

```bash
npm install
npm run dev
```

App will be available at `http://localhost:5173`.

---

## API Reference

All requests go to `http://localhost:8080`. The `api.js` module handles base URL, `Content-Type`, and JWT injection automatically.

### Auth Endpoints (public)

| Method | Path | Description |
|---|---|---|
| `POST` | `/auth/register` | Register a new user |
| `POST` | `/auth/login` | Login, returns `{ token }` |

### Operation Endpoints (public, token optional)

| Method | Path | Description |
|---|---|---|
| `POST` | `/api/convert` | Convert a quantity to a target unit |
| `POST` | `/api/add` | Add two quantities |
| `POST` | `/api/subtract` | Subtract two quantities |
| `POST` | `/api/divide` | Divide two quantities |
| `POST` | `/api/compare` | Check if two quantities are equal |

### Protected

| Method | Path | Description |
|---|---|---|
| `GET` | `/api/history` | Get all past operations (JWT required) |

---

## Supported Units

| Type | Units |
|---|---|
| LENGTH | `FEET`, `INCHES`, `YARDS`, `CENTIMETERS` |
| WEIGHT | `KILOGRAM`, `GRAM`, `POUNDS` |
| VOLUME | `LITER`, `MILLILITER`, `GALLON` |
| TEMPERATURE | `CELSIUS`, `FAHRENHEIT`, `KELVIN` |

---

## Authentication Flow

```
User visits app
       │
       ├─ No token → Dashboard (public operations work)
       │
       ├─ Clicks "History" → redirected to Login page
       │       │
       │       ├─ Username/password → POST /auth/login → JWT stored in localStorage
       │       │
       │       └─ GitHub OAuth2 → /oauth2/authorization/github → redirect back
       │                         with ?token=xxx → stored in localStorage
       │
       └─ Token present → History page (GET /api/history with Bearer token)
```

### OAuth2 Callback

After GitHub login, the backend redirects to `/?token=<jwt>`. `App.jsx` reads this from `window.location.search`, stores it, and clears the URL.

---

## Pages

### Dashboard (`/`)

- Select an operation from the top bar
- Enter quantity values and units
- Optionally select a target unit (for Convert / Add / Subtract)
- Click **Run** — result appears inline
- No login required

### Login (`/login` state)

- Switch between **Sign In** and **Register** tabs
- GitHub OAuth2 button
- On success, redirected straight to History

### History (`/history` state)

- Shows all past operations, newest first
- Filter by operation type using the pill buttons
- Each card shows inputs, operation symbol, and result

---

## Environment

The base URL is hardcoded in `api.js`:

```js
const BASE = "http://localhost:8080";
```

To change it for production, update this value or extract it into a `.env` file:

```env
VITE_API_BASE=https://your-api-domain.com
```

```js
// api.js
const BASE = import.meta.env.VITE_API_BASE || "http://localhost:8080";
```

---

## Notes

- JWT tokens are stored in `localStorage` under the key `jwt_token`
- Operations that require two quantities (`ADD`, `SUBTRACT`, `DIVIDE`, `COMPARE`) validate that both inputs share the same measurement type
- The History page shows a type-mismatch warning in the UI before the API is called
