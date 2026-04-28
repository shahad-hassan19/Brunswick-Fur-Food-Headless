# 🐾 Brunswick Fur Food (BFF)

*Live Demo:* [https://web.brunswickfurfood.com/](https://web.brunswickfurfood.com/)  
*Deployed on:* Vercel  
*Stack:* Next.js (TypeScript), Tailwind CSS, Zustand, Shopify Storefront API, Shopify Admin API, MongoDB, Cloudinary, Appstle


---

## 🧠 Overview

**Brunswick Fur Food (BFF)** is a modern headless e-commerce platform for a Melbourne-based pet food brand specializing in fresh, human-grade dog food and monthly subscription boxes. Built on top of Shopify's APIs with a fully custom Next.js frontend, it supports product browsing, cart management, customer accounts, subscription lifecycle management, pet profile creation, and order tracking — all delivered through a branded, responsive interface.


---

## 👥 Target Audience

BFF is designed for pet owners who want fresh, quality food delivered to their door on a regular subscription. The platform serves:

- **Dog owners** — subscribe, manage deliveries, and track orders
- **New customers** — browse products, customize selections, and check out
- **Returning subscribers** — pause, skip, resume, or cancel their subscription from a self-service dashboard

There is no age restriction, and the platform is built to be accessible across all devices.


---

## 🧩 Key Features

BFF covers the full customer journey from product discovery to ongoing subscription management:

- **Authentication & Accounts**:
  - Customer registration and login via Shopify Storefront API
  - Password reset flow (forgot-password + reset-password pages)
  - Rate-limited login endpoint (10 attempts per 60s window) with 429 + Retry-After header
  - Session persistence via `localStorage` tokens (access token + customer ID)

- **Product Catalog & Search**:
  - Products fetched from Shopify Storefront API with category filtering by tag: Food, Treats, Grooming, Essentials
  - Predictive search with live results
  - Product detail pages with full variant selection (size, protein/type)
  - Featured collections and recommended products sections
  - Blog articles feed from Shopify blogs

- **Shopping Cart**:
  - Shopify-managed persistent cart (cart ID stored in `localStorage`)
  - Add, remove, and update line items via GraphQL mutations
  - Real-time cart totals calculated by Shopify
  - Floating cart UI with item count badge and order special instructions
  - Zustand cart store with `localStorage` persistence across sessions

- **Checkout & Payments**:
  - Shopify hosted checkout redirect (primary path) — secure, tax/shipping calculated by Shopify
  - Geographic restriction: Melbourne delivery only
  - Flat $10 shipping fee, waived over $50 threshold

- **Subscription Management (Appstle)**:
  - View active subscriptions with billing dates and line items
  - Pause, resume, cancel, and skip next delivery — all from the account dashboard
  - Confirmation modal before destructive subscription actions
  - Multiple subscription support (primary + secondary)

- **Pet Profile Management**:
  - Dog profile creation and editing (name, breed, weight, age, gender)
  - Dog photo upload to Cloudinary with live preview
  - Personalized dashboard greeting with the dog's name
  - Profiles stored in MongoDB (Mongoose `UserDog` model)

- **Order Management**:
  - Full order history with fulfillment status (delivered / shipped / processing)
  - Order detail pages with items, quantities, and shipping address
  - Next delivery date calculation stored in Zustand order store

- **Newsletter**:
  - Email capture via conditional modal
  - Newsletter subscription endpoint with Shopify customer sync


---

## 🧱 Tech Stack Overview

- **Frontend**: Next.js 14 (TypeScript) with App Router, Server Components, and Server Actions
- **Styling**: Tailwind CSS with Tailwind Merge, Tailwind Animate, and Class Variance Authority for component variants
- **State Management**: Zustand — lightweight stores for cart and orders with `localStorage` persistence
- **E-Commerce Backend**: Shopify Storefront API (v2026-04) for products, cart, and customer auth; Shopify Admin API (v2024-01) for orders and server-side operations
- **GraphQL**: GraphQL Request for all Shopify API queries and mutations
- **Subscriptions**: Appstle — third-party Shopify subscription app, integrated via REST API
- **Database**: MongoDB (Mongoose 9.3.0) for pet profiles and local order data
- **Image Storage**: Cloudinary v2 — dog photo uploads with CDN delivery
- **HTTP Client**: Axios with Authorization Bearer token injection and 401 redirect handling
- **UI Components**: Radix UI (Dialog), Swiper (carousel), Lucide React (icons)
- **Notifications**: React Hot Toast
- **Utilities**: date-fns, libphonenumber-js, country-state-city, server-only


---

## 🔐 Authentication & Session Management

Authentication is handled entirely through Shopify's Storefront API customer mutations (login, register, password reset). Tokens are stored in `localStorage` and attached to every API call via an Axios instance:

```typescript
// Axios instance injects token on every request
Authorization: Bearer <bff_customer_token>
X-Customer-Id: <bff_customer_id>
```

The login route is protected by a rate-limiting middleware (`middleware.ts`) that blocks IPs after 10 failed attempts within a 60-second window and returns a `429` with `Retry-After` headers.


---

## 🛍️ Shopify Integration Architecture

BFF is built as a **headless Shopify storefront** — the frontend is fully custom while all commerce logic (products, variants, cart, checkout, customers, orders) is powered by Shopify's APIs.

| Layer | API Used | Purpose |
|---|---|---|
| Product catalog, cart, customer auth | Storefront API (GraphQL) | Client-facing data |
| Orders, admin operations | Admin API (GraphQL) | Server-side retrieval |
| Subscriptions | Appstle REST API | Recurring billing management |

GraphQL queries and mutations are organized under `lib/shopify/queries/` and `lib/shopify/mutations/`. All Shopify data fetching uses a `no-store` cache strategy to keep selling plan and subscription data fresh.


---

## 🐕 Pet Profile & Cloudinary Integration

Customers can create a dog profile from `/account/dog`. The dog's photo is uploaded directly to **Cloudinary** and the CDN URL is stored in MongoDB alongside the profile data:

```
POST /api/account/dog  →  upload image to Cloudinary  →  save to MongoDB (UserDog model)
GET  /api/account/dog  →  retrieve profile by Shopify customerId
```

The account dashboard personalizes the greeting using the stored dog name.


---

## 📦 Subscription Management (Appstle)

Subscription data is fetched from the **Appstle API** using the customer's Shopify ID. The account dashboard surfaces:

- Active subscription status, billing date, and line items with quantities and prices
- Action buttons for: **Pause**, **Resume**, **Cancel**, and **Skip Next Delivery**
- A confirmation modal before any destructive action to prevent accidental cancellations

All subscription mutations route through `/api/shopify/subscriptions/update-status`.


---

## 🚀 Deployment

BFF is deployed in a **private environment** for production use by the Brunswick Fur Food brand. The frontend is a Next.js application connected to a Shopify store, MongoDB Atlas cluster, and Cloudinary account.


---

## 🛠️ Challenges Faced

- **Headless Shopify Integration**:
  Building a fully custom storefront on top of Shopify's Storefront and Admin GraphQL APIs required understanding Shopify's data model deeply — carts, variants, selling plans, customer mutations — and building a clean abstraction layer over GraphQL queries/mutations for reliable, type-safe usage throughout the app.

- **Appstle Subscription API**:
  Appstle has limited public documentation. Integrating pause, resume, cancel, and skip-delivery flows required careful trial-and-error to discover the correct endpoint signatures and map Appstle's subscription contract schema to the frontend state.

- **Token Lifecycle & Rate Limiting**:
  Managing Shopify customer tokens across sessions (no server-side cookies) while also protecting the login endpoint from brute force attacks required implementing both `localStorage`-based session handling and custom rate-limiting middleware in a stateless Next.js environment.

- **Dual Checkout Architecture**:
  Supporting both Shopify's hosted checkout (primary) and a custom on-site checkout path meant maintaining two separate flows without breaking the cart state or duplicating order data between Shopify and MongoDB.


---

## 💡 Lessons Learned

- **Headless Commerce Architecture**: Gained hands-on experience building a production headless storefront — separating the frontend entirely from the commerce backend while consuming GraphQL APIs for all data needs.
- **Shopify Storefront & Admin APIs**: Learned to work with both Shopify API layers — Storefront for client-facing queries and mutations, Admin for privileged server-side operations.
- **Third-Party Subscription Platforms**: Learned to integrate Appstle as a subscription engine on top of Shopify, handling recurring billing state from the frontend.
- **Cloudinary for User-Generated Content**: Implemented file upload pipelines with CDN-backed image storage and preview rendering.
- **Stateless Auth in Next.js**: Built a complete JWT auth flow without server-side sessions, handling token refresh, expiry, and rate limiting entirely in Next.js middleware and API routes.
- **Zustand for E-Commerce State**: Used Zustand stores with `localStorage` persistence to manage cart and order state reliably across page navigations and browser sessions.


---

## 🎯 Future Plans

The core platform scope is complete. Potential future enhancements include:

- Real-time order status updates via Shopify webhooks
- A personalized product recommendation engine based on the dog's breed and weight profile
- In-app chat or support widget for subscription queries
- Mobile app adaptation for broader accessibility


---

Thank you! <br>
**Shahad Hassan** <br>
Full Stack Developer <br>
*Softles*
