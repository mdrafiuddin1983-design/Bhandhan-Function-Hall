# Running this locally, and showing it to your client — a realistic guide

## What's actually real right now
- **Website (`index.html`)**: fully working, published live at the artifact link Claude gave you. Open it on any laptop, phone, or tablet — nothing to install. This is the fastest thing to show your client today.
- **Backend (`server/`)**: real code, but it has never been run — it needs a Postgres database and a machine to run Node on. It is not live anywhere yet.
- **Mobile app**: does not exist yet. Nothing has been built for Android or iOS. See the "About the mobile app" section below before promising your client a date.

---

## Option A — Show your client the website right now (0 setup)
Just send them the published link. It works on their phone's browser already, and if they tap **Share → Add to Home Screen** (iPhone) or the browser's **Install app** prompt (Android Chrome), it adds an app icon to their home screen and opens full-screen like a native app — no App Store needed for this.

This is good for showing the **design and booking flow**. It is NOT connected to a real database yet — the calendar's "already booked" dates and the payment step are simulated in the page itself, not talking to the backend below.

## Option B — Run the real backend + database on your machine
This connects the booking engine, the database, and (once you add real API keys) real payments — but the website above still won't be wired to it until we connect the two (see "What's still needed" below).

**You'll need installed:** [Docker Desktop](https://www.docker.com/products/docker-desktop/) (this runs Postgres and the API for you — you don't need to install Postgres or Node separately).

**Steps:**
1. Unzip the backend package you downloaded, so you have a folder containing `docker-compose.yml`, `db/`, `server/`, `docs/`.
2. Open a terminal in that folder.
3. Run:
   ```
   docker compose up
   ```
4. Wait for it to say `API listening on :4000`. That's it — the database schema and default notification templates are loaded automatically.
5. Test it's alive by opening `http://localhost:4000/api/halls` in a browser — you should get back `[]` (empty list, since no hall has been added yet).

At this point you have a real, running backend on `localhost:4000` — but it has no data in it yet (no halls, no sample bookings), and the website isn't pointed at it. That's genuinely the next piece of work, not a config toggle.

---

## About the mobile app — please don't promise your client an App Store date yet

Nothing has been built for this. Here's the honest path and timeline once we start:

1. **Build the app** (React Native, one codebase for both platforms) — reuses the same backend API. This is real development time, not a few minutes.
2. **Preview it on your client's phone within minutes, with no App Store involved**: using [Expo](https://expo.dev/), once the app exists, your client installs the free "Expo Go" app from their app store once, and then scanning a QR code opens *your* app inside it instantly, updating live as we build. This is the realistic way to "show the client the app on their phone" quickly.
3. **Actually publishing to the App Store / Play Store** requires:
   - **Apple**: an Apple Developer Program account (**$99/year**, needs a real person or company enrolled), app icons/screenshots in Apple's required sizes, a privacy policy URL, and then Apple's review — typically **1–3 days**, sometimes longer if they ask questions.
   - **Google Play**: a Google Play Developer account (**$25 one-time**), similar assets, and Google's review — usually **a few hours to a couple of days**.
   - Both require *your* accounts — I can't create these or submit on your behalf. I can prepare everything the submission needs (icons, screenshots, store listing text, the build itself) once we get there.
4. Realistically: app build → internal testing → store submission → review is **weeks, not days**, the first time through. Please set that expectation with your client now rather than after the fact.

---

## What's still needed to make Option B feel "real" for a demo
- Add at least one hall's data (Bandhan Function Hall itself) into the database — currently the seed only loads notification templates, not any hall/owner/customer records.
- Point the website's calendar and payment step at the live API instead of its built-in sample data.
- Real Razorpay/Cashfree test-mode keys, so the payment step actually completes instead of being simulated.

I can do all three next — just say the word, and let me know if you want the client demo running against fake/sandbox payments (recommended for a demo) or if you already have live payment gateway credentials ready.

## About commission auto-deduction — the honest version

You asked for commission to be deducted automatically, without depending on the owner or manager to pay you. That's a real, standard feature — **Razorpay Route** (Cashfree calls it "Easy Split") — and it's built into the code now (`server/src/routes/payments.ts`, `server/src/routes/ownerAccounts.ts`). Here's exactly how it works and what it needs from you:

1. **You need a Razorpay (or Cashfree) *business* account with Route/marketplace enabled.** This requires your own KYC — PAN, business registration if you have one, bank account. This is unavoidably you, not me — a payment gateway will not let an AI or an unverified party move real money.
2. **Each hall owner completes their own one-time KYC** with the gateway (their PAN, their bank account) — `POST /api/owner-accounts/onboard` starts this. Until an owner's account shows `ACTIVE`, the code deliberately does NOT auto-split — it holds the full amount and logs a warning, rather than guessing or sending 100% to an unverified owner.
3. **Once both are done**, every customer payment is split by Razorpay itself, at the moment of payment: your commission percentage never leaves your account, and only the owner's share is transferred to them. Neither the owner nor a manager ever holds your commission — there's nothing for them to forget or withhold.
4. **Commission percentage** defaults to 5% (`platform_settings` table), overridable per hall. Change it any time — no code change needed.
5. **Auditability**: `GET /api/owner/commission-ledger` shows you every payment, your commission amount, and whether the gateway confirmed the split — so you can see if any owner is still on old-style manual settlement (not yet onboarded) versus fully automatic.

This is genuinely solid once set up — but "once set up" includes real KYC steps that take your gateway 1–3 business days per owner. There's no way to make that instant; it's the tradeoff for the money being provably safe.
