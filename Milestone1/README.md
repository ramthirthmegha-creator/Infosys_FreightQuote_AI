# Milestone 1 — User Authentication Module

**Infosys Springboard Internship 7.0 · Batch 1**

## What This Milestone Is

This milestone implements a complete **Login · Signup · Forgot Password** authentication
system as a single-page Streamlit application, run inside Google Colab and exposed
publicly through an ngrok tunnel. Sessions are managed with JWT, and password recovery
supports both a security-question flow and an email-based OTP flow sent via Gmail SMTP.

## Architecture

The whole app is one Python script (`app.py`), written to disk by a Colab cell and run
as a background Streamlit process, tunnelled to the public internet by ngrok.

```
┌─────────────┐   writes    ┌──────────┐   runs as    ┌────────────┐   public URL   ┌────────┐
│ Colab Cell  │ ──────────▶ │ app.py   │ ───────────▶ │ Streamlit  │ ─────────────▶ │ ngrok  │
│ (%%writefile)│            │ (script) │              │  process   │                │ tunnel │
└─────────────┘             └──────────┘              └────────────┘                └────────┘
                                  │
                    ┌─────────────┼──────────────┐
                    ▼             ▼               ▼
             ┌────────────┐ ┌───────────┐  ┌──────────────┐
             │  SQLite DB │ │  PyJWT     │  │  Gmail SMTP  │
             │ (users     │ │ (session   │  │ (OTP emails) │
             │  table)    │ │  tokens)   │  │              │
             └────────────┘ └───────────┘  └──────────────┘
```

- **Colab Secrets** feed `JWT_SECRET`, `EMAIL_ADDRESS`, `EMAIL_PASSWORD`, `NGROK_AUTHTOKEN`,
  and `ADMIN_USERNAME`/`ADMIN_PASSWORD` into the environment at launch — nothing sensitive
  is hardcoded in `app.py`.
- **SQLite** (`infosys_portal.db`) stores registered users: username, email, a bcrypt
  password hash, a security question, and a bcrypt-hashed security answer.
- **PyJWT** issues a signed, time-limited token on login, stored in Streamlit's session
  state — this token (not a server-side session) is what keeps a user "logged in."
- **Gmail SMTP** sends the one-time password for the Forgot Password → Email OTP route;
  the OTP itself is never stored in the database, only hashed inside a short-lived JWT.
- **Admin access** is a separate, hardcoded credential check — completely independent of
  the `users` table — so the admin is not a signup account.

## Features Built

- **Login** — sign in with either username or email + password. A single generic error
  is shown on failure so it never reveals whether the username or password was wrong.
- **Signup** — username, email, password, confirm password, security question (from a
  fixed list) and security answer. Usernames and emails must be unique.
- **Forgot Password** — two independent recovery routes on the same page:
  - *Security Question* — enter your username, answer your saved question, set a new password.
  - *Email OTP* — enter your registered email, receive a 6‑digit code by email (5‑minute
    expiry), verify it, then set a new password.
- **JWT session handling** — a signed JWT is issued on successful login and stored in
  Streamlit session state; the Dashboard only renders when a valid, unexpired token is present.
- **Field validation**
  - No form submits with an empty required field.
  - Email must match the pattern `ab@cd.ef` (≥2 letters before `@`, ≥2 letters between `@`
    and the final dot, ≥2 letters after the final dot).
  - Password must be ≥8 characters with at least one uppercase letter, one lowercase
    letter, one number and one special symbol; Confirm Password must match exactly.
- **User Dashboard** — welcome message with the logged-in username/email and a Logout action.
- **Admin Dashboard** — separate admin login (credentials defined in code / Colab Secrets,
  not a signup account) that lists every registered user's username and email — passwords
  are never displayed.
- **Secrets management** — `JWT_SECRET`, `NGROK_AUTHTOKEN`, `EMAIL_ADDRESS`, `EMAIL_PASSWORD`
  (and optional `ADMIN_USERNAME` / `ADMIN_PASSWORD`) are all read from Google Colab Secrets —
  nothing sensitive is hard-coded in the notebook.

## Tech Stack

| Layer | Technology |
|---|---|
| UI / Frontend | Streamlit (custom CSS — yellow/teal neo-brutalist style) |
| Auth / Sessions | PyJWT (JSON Web Tokens) |
| Password hashing | bcrypt |
| Database | SQLite (local file, auto-created on first run) |
| OTP delivery | Gmail SMTP (smtplib) |
| Public tunnel | ngrok (pyngrok) |
| Runtime | Google Colab |

## How to Run

1. Open `Login_Page.ipynb` in Google Colab.
2. Click the **key icon** (Secrets) in the left sidebar and add:
   - `JWT_SECRET` — any long random string
   - `NGROK_AUTHTOKEN` — your ngrok Authtoken
   - `EMAIL_ADDRESS` — the Gmail address that will send OTP emails
   - `EMAIL_PASSWORD` — the Gmail **App Password** for that address (not your normal password)
   - *(optional)* `ADMIN_USERNAME` / `ADMIN_PASSWORD` — override the default admin login
3. Toggle **notebook access ON** for each secret.
4. Run the three cells top to bottom (install → write `app.py` → launch).
5. Open the printed ngrok URL in your browser.
6. Sign up as a user, or use the **Admin** tab to sign in as an administrator.

## Setting Up ngrok

1. Create a free account at [ngrok.com](https://ngrok.com).
2. On your ngrok dashboard, copy your personal **Authtoken**.
3. In Colab, click the **key icon 🔑** (Secrets) in the left sidebar.
4. Add a new secret named `NGROK_AUTHTOKEN`, paste the token as its value, and toggle notebook access **ON**.

## Setting Up a Gmail App Password (for sending OTP emails)

1. Open your Google Account settings for the Gmail address you want to send OTPs from.
2. Turn on **2-Step Verification** — an App Password option won't appear until this is enabled.
3. In the Google Account search bar, search for **"App Passwords"** and open it.
4. Create a new app password and label it (e.g. "Infosys Portal").
5. Click **Create** — Google shows a 16-character password once.
6. Copy it immediately (it can't be viewed again after closing).
7. In Colab Secrets, add `EMAIL_ADDRESS` (your Gmail address) and `EMAIL_PASSWORD` (the 16-character App Password, not your normal Gmail password).

## Screenshots

<p align="center">
  <b>Login page</b><br>
  <img src="screenshots/Login_page.png" width="500"><br><br>
  <b>Signup page</b><br>
  <img src="screenshots/Signup_page.png" width="500"><br><br>
  <b>Signup Validation Error</b><br>
  <img src="screenshots/Signup_validation_error.png" width="500"><br><br>
  <b>Forgot password</b><br>
  <img src="screenshots/Forgot_password.png" width="500"><br><br>
  <b>Forgot Password — Security Question route</b><br>
  <img src="screenshots/Forgot_password-security_question_route.png" width="500"><br><br>
  <b>Forgot Password — Email OTP route</b><br>
  <img src="screenshots/Forgot_password-email_OTP_route.png" width="500"><br><br>
  <b>OTP Email Received</b><br>
  <img src="screenshots/Actual_OTP_email.png" width="500"><br><br>
  <b>User Dashboard</b><br>
  <img src="screenshots/User_Dashboard.png" width="500"><br><br>
  <b>Administrator Login</b><br>
  <img src="screenshots/Administrator_Login.png" width="500"><br><br>
  <b>Admin Dashboard</b><br>
  <img src="screenshots/Admin_Dashboard.png" width="500">
</p>
