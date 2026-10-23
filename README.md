# ExamPilot

Node.js + Express + SQLite (all data is saved in `data/exampilot.db`).

## Run
1. Node.js 18+ installed.
2. In this folder: `npm install`, then copy `.env.example` to `.env` and edit it, then `npm start`.
   After EVERY change to `.env`: stop the server (Ctrl+C) and start it again.
3. Candidates: http://localhost:3000   Admin: http://localhost:3000/admin
The terminal prints whether `.env` was found and the admin password length, so you can check it loaded.

## Admin flow (example: 10 minute form, tickets in 1 minute)
1. Exam centres: add centres (state list, UP district list, seats per shift).
2. Examinations: "Add top 10 exams" (or create your own).
3. Timing and plan: exam date, first shift time, duration, gap, allocation mode and delays
   (minutes), hall ticket delay, date slip days, date-change threshold, fee.
4. Release form: type how many minutes it stays open (it closes by itself; "Close form now" works too).
5. After it closes, the AI allocates seats after your allocation delay (or instantly per registration).
   Hall tickets open after your hall ticket delay. Buttons: open now / hold / automatic.

## Date clashes (multi-day exams)
- Set "Number of exam days" in Timing and plan (example: 3).
- Candidates may enter another exam's expected date (optional), or press "I have another exam on this date" on the date slip.
- If the clashes on one day reach the threshold, ONLY the clashing candidates are moved to another exam day:
  a free seat at the same centre if there is one, otherwise a swap with a non-clashing candidate. Everyone else keeps their date.
- Below the threshold nothing moves. A 1-day exam has no other day, so clashes cannot be moved.

## Payment
After registering, the candidate sees a UPI QR (set UPI_ID in `.env`). Registration does not depend on payment.
Scanning opens the UPI app; payment is not verified automatically. Use "Mark paid" in Candidates, or add a gateway later.

## Keeping the site up
Rate limiting, security headers, compression, async hashing, crash guards, SQLite WAL, /health.
Live: use PM2 (`pm2 start server.js --name exampilot`), Cloudflare in front, HTTPS, TRUST_PROXY=1, back up `data/exampilot.db`.

## Deploy (GitHub + a host)
GitHub only stores the code. Run it on a Node host (Render, Railway, Fly.io, a VPS...) connected to your GitHub repo.
1. Never upload `.env`. It is in `.gitignore`. Put the same values in the host's "Environment Variables" screen:
   ADMIN_USER, ADMIN_PASSWORD, UPI_ID, UPI_NAME, SMTP_*, MAIL_FROM, TRUST_PROXY=1
2. Build command `npm install`, start command `npm start`.
3. The database is a file. Attach a persistent disk/volume to the host and set DATA_DIR to its path (example /var/data).
   Without a persistent disk the data is lost on every restart or redeploy.
4. Open https://your-site/ for candidates and https://your-site/admin for admin.
