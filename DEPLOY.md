# Making the demo fully live — step by step

Everything below uses **free tiers** and **Razorpay test mode** (fake cards, no real money, no KYC needed). Total time: roughly 20–30 minutes the first time. I can't click these buttons for you — they create accounts under your name — but every step is exact, so you can follow it without being a developer, or hand it to anyone who can follow instructions.

## Step 1 — Get a Razorpay TEST key (5 min, no KYC)
1. Go to https://dashboard.razorpay.com/signup and sign up with your email.
2. You'll land in **Test Mode** automatically (toggle top-left says "Test Mode").
3. Go to **Settings → API Keys → Generate Test Key**.
4. Copy the **Key Id** (starts `rzp_test_...`) and **Key Secret** — save them somewhere safe.
5. Go to **Settings → Webhooks → Add New Webhook** (you'll come back to fill the URL in Step 3) and note down a **Webhook Secret** you choose (any random string).

## Step 2 — Deploy the database + backend (Railway, free trial credit)
1. Go to https://railway.app and sign up (GitHub login is fastest).
2. Click **New Project → Deploy from GitHub repo**. If you don't have this code in GitHub yet: create a free GitHub account, create a new repository, and upload the contents of the `backend.zip` I gave you (drag-and-drop upload works on github.com, no command line needed).
3. Back in Railway: **New Project → Deploy from GitHub repo**, pick that repository.
4. Railway will detect the `server/Dockerfile` — if it asks for a root directory, set it to `server`.
5. Click **New → Database → Add PostgreSQL** in the same project.
6. Open the Postgres service → **Connect** tab → copy the **Postgres Connection URL**.
7. Open your API service → **Variables** tab → add:
   - `DATABASE_URL` = (the Postgres URL you just copied)
   - `JWT_ACCESS_SECRET` = any random string
   - `JWT_REFRESH_SECRET` = any random string
   - `RAZORPAY_KEY_ID` = your `rzp_test_...` key
   - `RAZORPAY_KEY_SECRET` = your key secret
   - `RAZORPAY_WEBHOOK_SECRET` = the webhook secret you chose in Step 1
8. Open the Postgres service → **Data** tab → **Query** → paste the contents of `db/schema.sql`, run it, then do the same for `db/seed_notification_templates.sql` and `db/seed_demo_data.sql`, in that order.
9. Back on your API service, click **Settings → Networking → Generate Domain**. You'll get a public URL like `https://bandhan-api-production.up.railway.app`.
10. Visit `<that URL>/api/halls` in your browser — you should see one hall (Bandhan Function Hall) come back as JSON. If so, your backend is live.
11. Go back to Razorpay → **Settings → Webhooks**, set the URL to `<that URL>/api/payments/webhook`, select event `payment.captured`, and save.

## Step 3 — Point the website at your live backend
Tell me your Railway URL and your Razorpay **Key Id** (never share the Key *Secret* with me or anyone else — only the backend needs that, and it's already safely in Railway's variables, not in the website). I'll fill in the three config lines at the top of the website's code (`API_BASE_URL`, `HALL_ID` — already fixed — and `RAZORPAY_KEY_ID`) and republish the page. From that point, the calendar, the hold, and the payment are all real, backed by your live database.

## Step 4 — Test it like a customer would
Use Razorpay's published test card: **4111 1111 1111 1111**, any future expiry, any CVV, any OTP screen just click submit. This completes a real (test-mode) checkout end-to-end: order created → checkout → webhook fires → your database marks the booking CONFIRMED with a real commission split recorded (once an owner account is also onboarded per Step 5) → the calendar shows that date as booked for the next visitor.

## Step 5 — (Optional for the demo) Owner payout account
For the demo, you can skip real owner KYC — the code already handles "no linked account yet" by holding the full amount on your platform account rather than guessing. When you're ready to test the actual commission split, `POST /api/owner-accounts/onboard` starts an owner's real Razorpay Route KYC — still test-mode-friendly, Razorpay documents a full test onboarding flow for this too.

---

**When you've done Steps 1–2 and have a URL + key, just send them to me and I'll wire up the site.**
