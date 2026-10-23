# RevoShop Frontend

Author: **M Ikhsan Febrian**

| | URL |
| --- | --- |
| 🛍️ Live frontend (Vercel) | **https://revoshop-frontend.vercel.app** |
| ⚙️ Live API (Flask on Vercel + Supabase Postgres) | https://module-2-sann02-hvbj.vercel.app |

| Checkpoint | What | Where |
| --- | --- | --- |
| **3** | Forms, authentication, cart + checkout, admin dashboard, Playwright E2E, Vercel deploy | repo root (`app/`, `components/`, `context/`, `hooks/`, `lib/`, `e2e/`) |
| **2** | Next.js 16 App Router pages with live data from the Module 2 Flask API | repo root ([notes below](#checkpoint-2-nextjs--flask-api)) |
| **1** | HTML/CSS, vanilla JS, TypeScript + Tailwind fundamentals | `html-css/`, `javascript/`, `typescript-tailwind/` ([details below](#checkpoint-1--frontend-fundamentals)) |

---

# Overview

RevoShop is an online shop built with Next.js 16 (App Router) on top of the RevoShop Flask API
from Module 2. Visitors browse products and categories. Customers register, log in, fill a cart
that survives a refresh, check out, and track their orders. Admins manage products, categories
and see every order from a dashboard. Every screen talks to the real API; nothing is mocked.

# Features implemented

### 👤 Logged out
- **Home `/`**: 5 featured products. **Products `/products`**: live search + category filter
  synced to the URL (`?search=&category=`), refetched from the API.
- **Product detail `/products/[id]`**: dynamic page title, quantity counter + Add to Cart.
- **Categories `/categories`** with product counts.
- **Register `/register`**: `RegisterForm` with 4 rules (username required, email format,
  password ≥ 8 chars, passwords match). On `201` the user is logged in and sent to `/`.
  `409` shows "Email already registered".
- **Login `/login`**: `LoginForm`. `200` → admins go to `/dashboard`, customers to `/`.
  `401` → form-level "Invalid email or password". Empty fields are reported under the input.
- Add to Cart and every protected page send a guest to `/login`.

### 🛒 Customer (logged in)
- **Header** shows `Hi, {username}`, cart count and Logout. The session lives in `AuthContext`
  and `localStorage`, so it survives a refresh.
- **`AddToCartButton`** adds to `CartContext` (immutable updates, capped at stock), with a toast.
- **Cart `/cart`**: table with quantity steppers, subtotal per row, Remove per row, total,
  empty state, and **Proceed to checkout disabled when the cart is empty**. Persisted in
  `localStorage`.
- **Checkout `/checkout`**: order summary → **Confirm order** sends `POST /orders` with the JWT,
  clears the cart, shows a toast and redirects to `/orders`.
- **My Orders `/orders`** (own orders only) and **order detail `/orders/[id]`** (items, qty,
  subtotal, total). Another user's order shows "Order not found".
- `/dashboard`, `/dashboard/orders` and `/dashboard/categories` redirect a customer to `/`.

### 🔐 Admin
- **`/dashboard`**: product table with `CreateProductForm` (+ New product) and
  `EditProductForm` (Edit per row), both with validation. Delete asks for confirmation;
  a product in an **active order** cannot be deleted. The API's `409` is shown as a red
  banner and a toast.
- **`/dashboard/categories`**: category CRUD. `CreateCategoryForm` validates the name. Deleting
  a category that **still has products** is blocked (`409`) with a visible error.
- **`/dashboard/orders`**: read-only overview of every user's orders, with a User column.
- **Toasts** (`react-hot-toast`) for success and error on every CRUD and cart action.

### Route guards
| Wrapper | Rule | Used by |
| --- | --- | --- |
| `ProtectedRoute` | logged in, otherwise `/login` | `/cart`, `/checkout`, `/orders`, `/orders/[id]` |
| `AdminRoute` | logged in (`/login`) **and** admin (`/`) | `/dashboard`, `/dashboard/categories`, `/dashboard/orders` |
| `PublicOnlyRoute` | logged out, otherwise `/dashboard` or `/` | `/login`, `/register` |

Guards redirect with `router.replace()` inside `useEffect` and wait for `isReady`, the flag that
`AuthProvider` sets after reading `localStorage`, so a refresh never bounces a logged-in user.
Client-side guards are UX only. The API is still the real gatekeeper.

# Technologies used

Next.js 16 (App Router, Turbopack) · React 19 · TypeScript · Tailwind CSS v4 · react-hot-toast ·
Playwright · `next/navigation` (`useRouter`, `usePathname`, `useSearchParams`) · React Context ·
`localStorage` for session and cart persistence · `NEXT_PUBLIC_API_BASE_URL` · Vercel.
Backend: Flask + SQLAlchemy + JWT, deployed on Vercel with a Supabase PostgreSQL database.

# Folder structure

```
app/                         Pages (App Router). Server Components unless marked (client)
├── layout.tsx               <Providers> → Header + page + Footer, title template
├── page.tsx, products/, categories/          Checkpoint 2 pages (server-rendered from the API)
├── login/, register/        PublicOnlyRoute + LoginForm / RegisterForm
├── cart/, checkout/         ProtectedRoute + CartTable / CheckoutForm
├── orders/ , orders/[id]/   ProtectedRoute + CustomerOrders / OrderDetail
└── dashboard/               layout (tabs) + page / categories / orders, each in AdminRoute
components/                  UI: Header, NavBar, Card, ProductCard, AddToCartButton, CartTable,
                             CheckoutForm, RegisterForm, LoginForm, FormField, AddProductForm,
                             CreateProductForm, EditProductForm, CreateCategoryForm,
                             DashboardProducts, DashboardCategories, AdminOrders, OrdersTable,
                             ProtectedRoute, AdminRoute, PublicOnlyRoute, Providers, …
context/                     AuthContext (user + JWT), CartContext (items, totals), both persisted
hooks/                       useAuth(), useCart(), useApiData() (client fetch + reload)
lib/
├── api.ts                   Server-side fetch helpers (Server Components)
├── apiClient.ts             Browser fetch helpers: auth, CRUD, orders (Bearer token)
├── errors.ts                ApiError (status-aware) + getErrorMessage()
├── types.ts                 API types + form types
└── utils.ts                 formatting + Tailwind class helpers
e2e/                         Playwright specs + helpers (screenshots.spec.ts captures the README images)
playwright.config.ts
```

# Run it locally

**Requirements:** Node.js 20+, npm. Optional: the Flask API locally (otherwise use the live one).

```bash
git clone https://github.com/Revou-FSSE-Jun26/module-3-Sann02.git
cd module-3-Sann02
cp .env.example .env.local     # Windows: copy .env.example .env.local
npm install
npm run dev                    # http://localhost:3000
```

In `.env.local`, point `NEXT_PUBLIC_API_BASE_URL` at either API:
- the live one: `https://module-2-sann02-hvbj.vercel.app`
- a local Flask server: `http://localhost:5000` (see the Module 2 README: `flask db upgrade`,
  `python -m helper.seed`, `python -m helper.seed_order`, `flask run`)

Demo admin from the seed data: `admin@email.com` / `admin123`. Customers can register themselves.

| Script | What it does |
| --- | --- |
| `npm run dev` | Dev server (Turbopack) |
| `npm run build` / `npm start` | Production build / serve |
| `npm run lint` / `npm run typecheck` | ESLint / `next typegen` + `tsc --noEmit` |
| `npm run test:e2e` | Playwright E2E suite |
| `npm run test:e2e:report` | Open the last HTML report |

| Env key | Purpose |
| --- | --- |
| `NEXT_PUBLIC_API_BASE_URL` | API base URL (inlined at build time; set it in Vercel too). No hardcoded URLs |
| `API_SECRET_KEY` | Server-only secret, sent as `X-API-Key` when set |
| `API_DEMO_DELAY_MS` | Optional delay to show `loading.tsx` (default `0`) |

# Run the E2E test suite (Playwright)

```bash
npx playwright install chromium       # once (or set PLAYWRIGHT_CHANNEL=msedge / chrome)

# A) Against the live Vercel deployment
PLAYWRIGHT_BASE_URL=https://revoshop-frontend.vercel.app npm run test:e2e
#    PowerShell: $env:PLAYWRIGHT_BASE_URL="https://revoshop-frontend.vercel.app"; npm run test:e2e

# B) Locally: build first; the config starts `next start` on port 3100
npm run build
npm run test:e2e
```

There are 23 tests in 4 specs. They run against the real API and create their own uniquely named
users, products and categories. Products and categories are deleted again at the end.

| Spec | Covers |
| --- | --- |
| `public.spec.ts` | home, search + URL, dynamic product title, categories, guest Add to Cart → `/login`, every protected route → `/login` |
| `auth.spec.ts` | RegisterForm's 4 validation rules, register → `/` + refresh keeps the username, duplicate email, login 401 banner, empty-field errors, customer → `/` and blocked from `/dashboard*` |
| `cart-checkout.spec.ts` | empty state + disabled checkout, add with quantity, persistence across refresh, remove, checkout → `POST /orders` → cart cleared → `/orders` → order detail |
| `admin.spec.ts` | admin → `/dashboard`, product validate/create/edit/delete, **409 delete guard**, category validate/create/edit/delete + **linked-products guard**, all-orders overview |

**Production run** against `https://revoshop-frontend.vercel.app` (Chromium/Edge, 1 worker):
**23 passed**, 0 failed. The product CRUD test passed on its automatic retry after a slow
first API response, which the config's `retries: 2` absorbs. The 3 skipped tests are
`screenshots.spec.ts`, which only runs with `CAPTURE_SCREENSHOTS=1`.

The guard tests use seeded data: "Keyboard Mechanical" is in a pending order, and "Elektronik"
has products. Override them with `E2E_GUARDED_PRODUCT` / `E2E_GUARDED_CATEGORY` and the admin
login with `E2E_ADMIN_EMAIL` / `E2E_ADMIN_PASSWORD`.

# Screenshots (`screenshots/checkpoint-3/`)

Regenerate with `CAPTURE_SCREENSHOTS=1 npx playwright test e2e/screenshots.spec.ts`.

| Home | Product listing | Product detail |
| --- | --- | --- |
| ![Home](screenshots/checkpoint-3/01-home.png) | ![Products](screenshots/checkpoint-3/02-products.png) | ![Product detail](screenshots/checkpoint-3/03-product-detail.png) |

| Categories | Register (validation) | Login (401) |
| --- | --- | --- |
| ![Categories](screenshots/checkpoint-3/04-categories.png) | ![Register](screenshots/checkpoint-3/05-register-validation.png) | ![Login](screenshots/checkpoint-3/06-login-error.png) |

| Cart | Checkout | My orders |
| --- | --- | --- |
| ![Cart](screenshots/checkpoint-3/07-cart.png) | ![Checkout](screenshots/checkpoint-3/08-checkout.png) | ![My orders](screenshots/checkpoint-3/09-my-orders.png) |

| Admin dashboard | Admin categories | Admin orders |
| --- | --- | --- |
| ![Dashboard](screenshots/checkpoint-3/10-admin-dashboard.png) | ![Categories admin](screenshots/checkpoint-3/11-admin-categories.png) | ![Orders admin](screenshots/checkpoint-3/12-admin-orders.png) |

### Known limitations
- The Flask API does not check the admin role on write endpoints or on `GET /orders` without a
  token, so the admin pages are protected in the UI only (the Module 2 rubric made JWT optional).
- The JWT is stored in `localStorage`. That's fine for this course, but a production app would
  prefer an httpOnly cookie.

---

# Checkpoint 2: Next.js + Flask API

Checkpoint 3 changed two things here. The cart moved from `ProductGrid` state into
`CartContext`, and `AddProductForm` became the shared, validated form behind
`CreateProductForm` and `EditProductForm`. Everything else below still applies.

- **Server vs Client.** Catalog pages are Server Components that `await` the API (`lib/api.ts`).
  Client Components handle interactivity.
- **Search + category filter.** Typing filters the list instantly. After a short pause,
  `router.push()` writes `?search=` / `?category=` to the URL, and the `/products` Server
  Component refetches `GET /products?search=&category_id=`.
- **Errors / loading.** `lib/api.ts` checks `res.ok` and throws. Each route has `loading.tsx`
  (skeleton) and `error.tsx` (friendly message + **Try again**).
- **Nested layout.** `app/products/layout.tsx` stays mounted between `/products` and
  `/products/[id]` (see `[products/layout] mounted` in the console).
- Evidence screenshots: `screenshots/checkpoint-2/`.

---

# Checkpoint 1 — Frontend Fundamentals

These exercises cover the building blocks that React, Next.js and Tailwind sit on top of:
semantic HTML, CSS layout, vanilla JavaScript, TypeScript, and Tailwind.

## 📁 Folder structure

```
revoshop-frontend/
├── html-css/              Semantic profile page + contact form
│   ├── index.html         header/main/section/article/footer, full contact form
│   ├── styles.css         all 5 selector types, box model, CSS Grid, responsive
│   └── profile.js         prevents default form submit + status message
│
├── javascript/            Plain JS, runnable directly in the browser
│   ├── dom-events.html     ┐ DOM manipulation + event handling exercise
│   ├── dom-events.js       ┘ select/update, toggle class, create/remove elements
│   ├── array-processing.html ┐ forEach / map / filter / reduce exercise
│   └── array-processing.js   ┘ iterate, transform, aggregate typed array data
│
├── typescript-tailwind/   tsc + tsconfig + Tailwind CSS project
│   ├── package.json        build scripts + dev dependencies
│   ├── tsconfig.json       strict TypeScript config, compiles to dist/
│   ├── index.html          the catalog page (loads compiled JS + built CSS)
│   ├── src/
│   │   ├── types.ts        interfaces, type aliases, union types
│   │   ├── products.ts     the typed product data (array of typed objects)
│   │   ├── main.ts         render + live search + add-to-cart counter
│   │   └── input.css       Tailwind entry (@import "tailwindcss")
│   └── dist/               build output (generated; git-ignored)
│
├── screenshots/           profile (desktop + mobile) + catalog with live search
└── README.md
```

---

## ▶️ How to run each part locally

### 1. `html-css/` — profile page

No build step. Just open the file in a browser:

- Double-click `html-css/index.html`, **or**
- Right-click → *Open with Live Server* in VS Code.

Resize the window across **768px** to see the layout switch from a single
stacked column (mobile) to a two-column grid (content + sidebar) with the header
and footer spanning the full width.

### 2. `javascript/` — the two exercises

No build step. Open either file in a browser and open DevTools (F12) to see the
console output:

- `javascript/dom-events.html` — click the counter, toggle the highlight class,
  add/remove tasks.
- `javascript/array-processing.html` — results of `forEach`, `map`, `filter`,
  and `reduce` are rendered on the page **and** logged to the console.

### 3. `typescript-tailwind/` — typed product catalog

This one compiles TypeScript with `tsc` and builds CSS with the Tailwind CLI.

```bash
cd typescript-tailwind
npm install        # install typescript + @tailwindcss/cli
npm run build      # compiles src/*.ts -> dist/*.js  AND  builds dist/output.css
```

Then open `typescript-tailwind/index.html` in a browser (Live Server
recommended so ES module imports load over http).

Useful scripts:

| Script              | What it does                                        |
| ------------------- | --------------------------------------------------- |
| `npm run build`     | one-shot: compile TS **and** build Tailwind CSS     |
| `npm run build:ts`  | compile TypeScript only (`tsc`)                     |
| `npm run build:css` | build the Tailwind stylesheet only                  |
| `npm run watch:ts`  | recompile TS on change                              |
| `npm run watch:css` | rebuild CSS on change                               |

> **Note:** `dist/` is git-ignored (build artifacts aren't committed). After
> cloning, run `npm install && npm run build` before opening `index.html`.

---

## ✅ What each rubric item maps to

**HTML, CSS & JavaScript fundamentals**

- Semantic HTML5 (`header`, `nav`, `main`, `section`, `article`, `aside`,
  `footer`) — `html-css/index.html`.
- Contact form with labelled inputs of every type (text, email, number, radio
  group, checkbox, select, textarea, submit) — `html-css/index.html`.
- Box model + all five selector types (element, class, ID, descendant,
  pseudo-class) — `html-css/styles.css`.
- Two-dimensional CSS Grid layout + Flexbox inside a cell — `html-css/styles.css`.
- Responsive `@media (min-width: 768px)` breakpoint (mobile-first) —
  `html-css/styles.css`.
- Variables with correct types, operators, functions — the JS files.
- Select/update DOM, toggle classes, create/remove elements, event handlers with
  `preventDefault` — `javascript/dom-events.js` and `html-css/profile.js`.
- `forEach` / `map` / `filter` / `reduce` — `javascript/array-processing.js`.

**TypeScript & Tailwind CSS**

- `tsc` + configured `tsconfig.json`, compiles without errors —
  `typescript-tailwind/tsconfig.json`.
- Interfaces + type aliases + union types — `src/types.ts`
  (`Product`/`CartItem` interfaces, `Rupiah`/`Catalog` aliases,
  `Category`/`StockStatus` union types).
- Nested objects and arrays typed correctly — `src/products.ts` (an array of
  typed `Product` objects, each with a nested `tags` array).
- Tailwind installed & configured, utility-first classes instead of custom CSS —
  `src/input.css` + `index.html`.
- Typography utilities + mobile-first responsive layout
  (`grid-cols-1 sm:grid-cols-2 lg:grid-cols-3`) — `index.html`.
- Typed data mapped to conditional Tailwind class strings (badge colour derived
  from the union-typed `status` field) — `src/main.ts` (`statusBadgeClasses`).
- Fully typed interactive catalog: renders from data, **live search** filters by
  name/tag, **add-to-cart counter** updates on click — `src/main.ts`.

---

## 📸 Screenshots

See the `screenshots/` folder:

- Profile page at a **desktop** width.
- Profile page at a **mobile** width.
- The typed product catalog showing **live search** and Tailwind styling in
  action.
