# Nirapod Sheba

**Project title:** Design and Development of a Trusted Home Service Platform Using Verified Technicians.

**Submitted by:** ID: 230222001 | 230222012 | 230222029 | 230222030 | 190222022

**GitHub:** https://github.com/sabbirCPabna  

**Date:** 26 September 2026

Nirapod Sheba is a web platform for booking home services. A client can hire a technician directly, or post a job and accept one bid. A technician registers the services he offers and the areas he covers, takes a subscription, and talks to the client from the job. An administrator maintains the service list, coverage areas, subscription plans, and the public pages.

The system is four programs: a customer website, an admin panel, a technician panel, and one API. The three panels are Next.js applications. The API is an Express server. It uses Prisma, MongoDB, and Redis. Chat runs over a WebSocket on the same API. Each program has its own repository and its own hostname.

This repository is the written account of the project. The source code is in the four repositories named below. Those repositories are private. Cloning them needs access to the GitHub account.

## Live system

| Application | Address | Port |
|---|---|---|
| Customer website | https://nirapodsheba.com | 3000 |
| Admin panel | https://admin.nirapodsheba.com | 3002 |
| Technician panel | https://technician.nirapodsheba.com | 3003 |
| API | https://api.nirapodsheba.com | 5003 |

https://www.nirapodsheba.com opens the same customer website. The API health check is https://api.nirapodsheba.com/health.

## Contents

1. [Problem](#1-problem)
2. [Modules](#2-modules)
3. [Architecture](#3-architecture)
4. [Data model](#4-data-model)
5. [Repositories](#5-repositories)
6. [Run locally](#6-run-locally)
7. [Deployment](#7-deployment)

## 1. Problem

A household that needs an electrician, a plumber, or help on short notice usually depends on a personal contact. The price is settled in conversation, and the arrival time is not fixed before the person leaves. There is also no single place where the same client can keep the booking, the payment, and the later review.

Nirapod Sheba gives that work one system. The client either chooses a technician or posts the job and compares bids. Each bid carries a price and an arrival time. The technician works only in the services and areas he has registered. The administrator, not the technician, owns the public catalogue of services and areas.

## 2. Modules

**Client.** The client browses services, books a technician directly, or posts a job for bids. He pays for a booking, writes a review, files a complaint, saves a favourite technician, and messages the technician assigned to the job.

**Technician.** The technician completes registration, chooses services and work areas, and subscribes to a plan. He sees open jobs, submits a bid with a price and an arrival time, follows the jobs assigned to him, reads reviews, and uses the same message thread as the client.

**Administrator.** The administrator manages services, service areas, and subscription plans. Public material, including banners, blog posts, certificates, and contact details, is edited from the same panel.

**API.** The API handles accounts, one-time passwords, both kinds of booking, payments, notifications, and chat. The panels do not talk to the database. They call this API.

## 3. Architecture

A visitor reaches one machine. Nginx receives HTTPS and forwards each hostname to the process that owns it. The three panels and the API are separate processes, so one panel can be rebuilt while the others stay up. MongoDB and Redis accept connections only from that machine.

```text
Browser
  │
  ▼
Nginx  :443
  ├── nirapodsheba.com            →  Next.js   :3000   customer
  ├── admin.nirapodsheba.com      →  Next.js   :3002   admin
  ├── technician.nirapodsheba.com →  Next.js   :3003   technician
  └── api.nirapodsheba.com        →  Express   :5003   API and WebSocket
                                         ├── MongoDB   :27017   replica set rs0
                                         └── Redis     :6379
```

The customer, admin, and technician interfaces differ because the three roles do different work. Keeping them in separate repositories means a change to the technician panel is built and released on its own.

## 4. Data model

A person is stored as a user with a role: client, technician, or admin. A technician also has a profile. The profile records whether the account is active, whether he accepts emergency work, and which subscription is in force.

A service is a kind of work. A service area is a place, stored with its division and district. The technician profile is linked to the services he offers and to the areas he covers.

Booking is stored in two shapes.

* **Offer booking.** The client selects a technician and a service. That technician is assigned at the time of the booking.
* **Bid-work booking.** The client posts the job and may mark it as emergency. Technicians submit bids. A bid has a price and an arrival time in minutes. The client accepts one bid. The accepted bid and the chosen technician are stored on the job.

A payment may belong to either booking. A subscription belongs to the technician. Reviews, complaints, and favourite technicians point at a technician profile. Two users share a chat room, and the messages of that conversation belong to the room. A notification is stored once and marked read for each recipient.

Registration uses a short-lived one-time password. The public site also stores plans, banners, blog posts, certificates, and the contact block shown on the pages.

## 5. Repositories

| Folder | Repository | Dev port | Program |
|---|---|---|---|
| `nirapod-sheba-frontend` | [nirapod-sheba-frontend](https://github.com/sabbirCPabna/nirapod-sheba-frontend) | 3000 | Customer website |
| `nirapod-sheba-admin` | [nirapod-sheba-admin](https://github.com/sabbirCPabna/nirapod-sheba-admin) | 3002 | Admin panel |
| `nirapod-sheba-technician` | [nirapod-sheba-technician](https://github.com/sabbirCPabna/sabbirCPabna-nirapod-sheba-technician) | 3003 | Technician panel |
| `nirapod-sheba-server` | [nirapod-sheba-server](https://github.com/sabbirCPabna/nirapod-sheba-server) | 5003 | API |

The technician repository has a long GitHub name. Clone it into a folder called `nirapod-sheba-technician`.

Stack used in the panels: Next.js, React, and Tailwind CSS. The technician panel keeps session and notification state in Redux Toolkit. The API is Express with Prisma. The database is MongoDB 7, started as replica set `rs0`, because Prisma needs a replica set. Redis holds short-lived server state. Card payments are handled by Stripe. The API exposes `POST /api/v1/payment/webhook` for Stripe.

## 6. Run locally

The four repositories must sit in one parent folder. Start MongoDB and Redis first, then the API, then the three panels. The panels call `http://localhost:5003/api/v1`. If the API is not running, the browser reports that the server cannot be reached.

### 6.1 Tools

* Node.js 20 or newer
* npm
* MongoDB 7, with replica set `rs0`
* Redis

Confirm the tools:

```bash
node -v
npm -v
redis-cli ping
```

`redis-cli ping` should print `PONG`.

If MongoDB is not yet a replica set, open `mongosh` and run:

```javascript
rs.initiate()
```

### 6.2 Clone

```bash
git clone https://github.com/sabbirCPabna/nirapod-sheba-server.git
git clone https://github.com/sabbirCPabna/nirapod-sheba-frontend.git
git clone https://github.com/sabbirCPabna/nirapod-sheba-admin.git
git clone https://github.com/sabbirCPabna/sabbirCPabna-nirapod-sheba-technician.git nirapod-sheba-technician
```

GitHub will ask you to sign in. Without access to these private repositories the clone stops.

### 6.3 API environment

In `nirapod-sheba-server`, copy `.env.example` to `.env`. Set at least these values for a machine on your own computer:

```env
DATABASE_URL="mongodb://127.0.0.1:27017/nirapodseba?replicaSet=rs0"
REDIS_URL="redis://127.0.0.1:6379"
PORT=5003
NODE_ENV=development
BACKEND_BASE_URL="http://localhost:5003"
TECHNICIAN_APP_URL="http://localhost:3003"
CORS_ORIGINS="http://localhost:3000,http://localhost:3002,http://localhost:3003"
JWT_SECRET="replace-with-a-long-random-string"
```

Mail, Stripe, and file-storage keys stay in the same file. Copy them from `.env.example`. Registration email and card payment do nothing useful until those keys are real. Do not commit `.env`.

Create the upload folder the API expects:

```bash
mkdir -p nirapod-sheba-server/uploads
```

### 6.4 Panel environment

Each panel reads the API address from the environment. Create the file before the first `npm run dev`. Next.js inlines `NEXT_PUBLIC_API_URL` at start-up, so a later edit needs a restart.

`nirapod-sheba-frontend/.env`

```env
NEXT_PUBLIC_API_URL=http://localhost:5003/api/v1
```

`nirapod-sheba-admin/.env`

```env
NEXT_PUBLIC_API_URL=http://localhost:5003/api/v1
```

`nirapod-sheba-technician/.env`

```env
NEXT_PUBLIC_API_URL=http://localhost:5003/api/v1
```

### 6.5 Start

Use four terminals.

```bash
cd nirapod-sheba-server
npm install
npx prisma generate
npm run dev
```

The API listens on port 5003. Check it:

```bash
curl http://127.0.0.1:5003/health
```

A healthy process returns JSON with `"ok": true` and `"service": "nirapod-sheba-api"`.

Then, in the other three folders:

```bash
cd nirapod-sheba-frontend
npm install
npx next dev -p 3000
```

```bash
cd nirapod-sheba-admin
npm install
npx next dev -p 3002
```

```bash
cd nirapod-sheba-technician
npm install
npm run dev
```

The technician script already binds port 3003. Open:

* http://localhost:3000 — customer
* http://localhost:3002 — admin
* http://localhost:3003 — technician

`CORS_ORIGINS` on the API must list these three origins. A panel opened on another port will be blocked by the browser.

## 7. Deployment

The live system runs on one Linux server. PM2 keeps the four processes running. Nginx terminates HTTPS and forwards each hostname to the port in the table above. MongoDB and Redis are bound to localhost, so they are not reachable from the public internet. A later update is a pull, install, and build inside the repository that changed, then a restart of that process only.

The customer site, the admin panel, the technician panel, and the API listed at the top of this file are the running copy of this project.
