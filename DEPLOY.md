# Deployment Guide — Excit3D Prints

---

## 1. Collect Payment Credentials

**Stripe**
1. Log in at [stripe.com](https://stripe.com) → **Developers → API Keys**
2. Copy your **Publishable key** (`pk_live_...`) and **Secret key** (`sk_live_...`)

**PayPal**
1. Log in at [developer.paypal.com](https://developer.paypal.com)
2. Go to **My Apps & Credentials** → open or create an app
3. Copy the **Client ID** and **Secret**

> Never put these keys in an email, Google Doc, or anywhere public.

---

## 2. Set Up Google Login (One-Time)

This lets you log in to the admin panel with your Google account.

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Create a new project → **APIs & Services → Credentials**
3. Click **Create Credentials → OAuth client ID → Web application**
4. Under **Authorized redirect URIs**, add:
   ```
   https://excit3dprints.com/api/auth/google/callback
   ```
5. Click **Create** and copy the **Client ID** and **Client Secret** — you'll need these in Step 6

> The redirect URI must match exactly — any difference will break login.

---

## 3. Build the Frontend

Run this once before uploading. If your developer already did this, skip to Step 4.

```bash
cd store && npm run build
```

This produces a `store/dist/` folder containing the compiled site.

---

## 4. Upload Files to Hostinger

Upload the following to your site's root directory via **File Manager** or FTP ([FileZilla](https://filezilla-project.org/)):

```
server/       ← entire folder
store/dist/   ← only this subfolder, not all of store/
```

Do **not** upload `store/src/`, `store/node_modules/`, or any other dev files.

---

## 5. Configure the Node.js Entry Point

1. In the Hostinger control panel, open the **Node.js** section
2. Set the **entry point** to:
   ```
   server/index.js
   ```
3. Save

---

## 6. Install Server Dependencies

1. Open the **SSH Terminal** in the Hostinger control panel
2. Run:
   ```bash
   cd server && npm install --production
   ```

---

## 7. Set Environment Variables

In the Hostinger control panel, find **Environment Variables** (inside the Node.js section) and add the following:

| Variable | Value |
|---|---|
| `DEV_AUTH_BYPASS` | `false` |
| `NODE_ENV` | `production` |
| `STRIPE_SECRET_KEY` | `sk_live_...` from Step 1 |
| `STRIPE_WEBHOOK_SECRET` | From Step 8 below |
| `PAYPAL_CLIENT_ID` | From Step 1 |
| `PAYPAL_CLIENT_SECRET` | From Step 1 |
| `GOOGLE_CLIENT_ID` | From Step 2 |
| `GOOGLE_CLIENT_SECRET` | From Step 2 |
| `GOOGLE_CALLBACK_URL` | `https://excit3dprints.com/api/auth/google/callback` |
| `SESSION_SECRET` | A random 32-character string — generate one at [generate-secret.vercel.app/32](https://generate-secret.vercel.app/32) |
| `FRONTEND_URL` | `https://excit3dprints.com` |

> `DEV_AUTH_BYPASS` must be `false`. If set to `true`, the admin panel is publicly accessible with no login.

---

## 8. Configure the Stripe Webhook

Stripe uses a webhook to notify your server when a payment completes. Without it, orders will not be recorded.

1. In the Stripe dashboard, go to **Developers → Webhooks → Add endpoint**
2. Set the URL to:
   ```
   https://excit3dprints.com/api/stripe-webhook
   ```
3. Under **Events**, select `checkout.session.completed`
4. Save, then copy the **Signing secret**
5. Back in Hostinger, set `STRIPE_WEBHOOK_SECRET` to that value

---

## 9. Restart the App

1. In the Hostinger control panel, go to the **Node.js** section
2. Click **Restart**
3. Visit `https://excit3dprints.com` to confirm the site loads

---

## Verification Checklist

- [ ] Store loads at `https://excit3dprints.com`
- [ ] Admin login works at `/admin` using your Google account
- [ ] Test order completes using Stripe test card `4242 4242 4242 4242` (any future date, any CVC)
- [ ] Test order appears in the admin panel

---

## Troubleshooting

**Site won't load** — Check that the entry point is set to `server/index.js` and the app is running in the Node.js panel.

**Admin login fails** — Verify `GOOGLE_CALLBACK_URL` in environment variables matches the redirect URI in Google Cloud exactly.

**Orders not recording** — `STRIPE_WEBHOOK_SECRET` is likely missing or incorrect. Recheck Step 8.

**Environment variable issues** — After changing any variable, you must restart the app for changes to take effect.
