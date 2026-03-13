# Starlife Advert Bot → Website Deployment (Netlify + Firebase + Admin Panel)

Yes — you can turn this into a website. The clean setup is:

- **Netlify** = host the frontend (website)
- **Firebase Auth** = user/admin login
- **Firestore** = app data (users, investments, withdrawals)
- **Cloud Functions** = secure admin actions + financial logic

---

## Direct answer to your question

> “So how should I deploy it, like a website using Netlify with admin panel?”

Use this model:

1. Build a frontend app (React/Vite recommended).
2. Deploy frontend to Netlify.
3. Use Firebase for auth/data/functions.
4. Add an `/admin` route in frontend.
5. Restrict admin access using Firebase **custom claims** + Firestore rules.

`index.js` in this repo is backend/bot code; it is **not** your website HTML file.

---

## Recommended architecture

### Frontend (Netlify)
- Public pages: `/`, `/login`, `/register`, `/dashboard`
- Admin page: `/admin`
- Frontend calls Firebase SDK and/or Cloud Functions

### Backend (Firebase)
- **Auth**: Email/password sign-in
- **Firestore**: user/investment/withdrawal records
- **Cloud Functions**:
  - approve/reject investments
  - approve/reject withdrawals
  - calculate referral payouts
  - write admin audit logs

---

## Step-by-step deployment guide

## 1) Create Firebase project

In Firebase Console:
- Create project
- Enable **Authentication → Email/Password**
- Create **Firestore** database
- Enable **Functions** (Blaze plan usually required)

Install CLI and init:

```bash
npm i -g firebase-tools
firebase login
firebase init
```

Choose:
- Firestore
- Functions
- Hosting (optional; if using only Netlify frontend, Hosting can be skipped)

---

## 2) Build website frontend

Recommended quick start:

```bash
npm create vite@latest web -- --template react
cd web
npm install
```

Install Firebase SDK in frontend:

```bash
npm install firebase
```

Create pages/routes:
- `/login`
- `/dashboard`
- `/admin`

---

## 3) Create admin panel access control (important)

Do **not** rely only on hiding UI buttons.
Use Firebase custom claims and backend checks.

### A) Set admin claim (one-time script in functions/admin setup)

Example (Node Admin SDK idea):
- `setCustomUserClaims(uid, { admin: true })`

### B) Protect `/admin` route in frontend

- Check logged-in user token claims
- If `admin !== true`, redirect away from `/admin`

### C) Enforce server-side admin checks

Every admin Cloud Function must verify auth token + `admin` claim before writing approvals.

---

## 4) Firestore collections (starter)

- `users/{uid}`
- `investments/{investmentId}`
- `withdrawals/{withdrawalId}`
- `transactions/{txId}`
- `audit_logs/{logId}`

Suggested fields:
- `status` (`pending`, `approved`, `rejected`)
- `createdAt`, `updatedAt`
- `approvedBy`, `approvedAt`

---

## 5) Security rules baseline

Use rules so:
- users can read/write only their own documents
- only admins can read all records or approve/reject

Pseudo-rule intent:
- `request.auth.uid == resource.data.userId` for user-owned docs
- `request.auth.token.admin == true` for admin operations

---

## 6) Deploy functions (secure backend)

From Firebase project root:

```bash
firebase deploy --only functions
```

Use callable HTTPS functions or HTTPS endpoints for admin actions.

---

## 7) Deploy frontend to Netlify

Push frontend to GitHub, then on Netlify:
- **New site from Git**
- Build command: `npm run build`
- Publish directory: `dist` (Vite) or your framework output

Add environment variables in Netlify Site Settings:
- `VITE_FIREBASE_API_KEY`
- `VITE_FIREBASE_AUTH_DOMAIN`
- `VITE_FIREBASE_PROJECT_ID`
- `VITE_FIREBASE_STORAGE_BUCKET`
- `VITE_FIREBASE_MESSAGING_SENDER_ID`
- `VITE_FIREBASE_APP_ID`

Then deploy.

---

## 8) SPA routing support on Netlify

For React/Vue SPA routes (`/admin`, `/dashboard`), add `web/public/_redirects`:

```txt
/* /index.html 200
```

This prevents 404 on page refresh.

---

## 9) Map existing bot logic safely

From current `index.js`, migrate business logic into Cloud Functions/services:
- investment creation/approval
- withdrawal fee + payout logic
- referral bonus logic
- notification triggers (email)

Never trust client-side amounts/status changes.

---

## 10) Production checklist

- [ ] Admin users have custom claim
- [ ] Admin actions validated server-side
- [ ] Firestore rules tested
- [ ] Audit logging enabled for admin operations
- [ ] Rate limits / abuse protection in place
- [ ] Environment variables configured in Netlify and Firebase

---

## Quick practical flow

1. Keep this repo as backend reference/business rules.
2. Build a separate `web/` frontend app.
3. Deploy frontend on Netlify.
4. Deploy secure actions via Firebase Functions.
5. Launch `/admin` with custom-claim authorization.

That gives you a real website + protected admin panel.
