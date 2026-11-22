# Riy@ Kloset: online shop

A fast, mobile-first shop for Riy@ Kloset (Accra): thrift clothing, bags, shoes and phones.
Static files hosted on GitHub/Cloudflare, with **Supabase** for the database, photo storage and logins.

| File | What it is |
|---|---|
| `index.html` | The shop customers see |
| `admin.html` | Owner's admin panel (add stock, photos, orders, settings) |
| `config.js` | Your Supabase URL + public key (the only file you must edit) |
| `supabase-setup.sql` | Creates the database, security rules and photo storage |
| `seed-sample-products.sql` | Optional: 27 sample items so the shop isn't empty on day one |

---

## Setup (about 20 minutes)

### 1. Create the Supabase project
1. Go to <https://supabase.com> → **New project**. Name: `riya-kloset`. Region: closest to Ghana (e.g. *West EU – London/Ireland*).
2. Save the database password somewhere safe.

### 2. Build the database
1. Supabase → **SQL Editor** → **New query**.
2. Paste all of `supabase-setup.sql` → **Run**. You should see "Success".
3. *(Optional)* New query → paste `seed-sample-products.sql` → **Run**.

### 3. Connect the site
1. Supabase → **Project Settings → API** (or *Data API*).
2. Copy **Project URL** and the **anon / public** key into `config.js`.
3. ⚠️ Never put the `service_role` / secret key in any file.

### 4. Make the owner an admin
1. Supabase → **Authentication → Users → Add user → Create new user**. Enter her email and a password, tick **Auto Confirm User**.
2. SQL Editor → run (with her email):
   ```sql
   insert into public.admins (user_id)
   select id from auth.users where email = 'HER-EMAIL@example.com';
   ```
3. She logs in at `https://YOUR-SITE/admin.html`. Add it to her phone's home screen.

### 5. Customer sign-up settings (Authentication)
- **Authentication → Sign In / Providers → Email**: keep **Email** on.
- **Confirm email**: ON is safer (stops fake accounts), but Supabase's built-in email only sends a few emails per hour.
  For launch, either connect a free SMTP sender (**Authentication → Emails → SMTP Settings**, e.g. Resend or Brevo)
  or turn confirmation OFF until you do. The site handles both.
- **Authentication → URL Configuration**: set **Site URL** to the live address (e.g. `https://riyakloset.com`)
  and add it under **Redirect URLs**, so password-reset and confirmation links come back to the shop.

### 6. Publish
Same as Vazuva: push the folder to a GitHub repository, then deploy with **Cloudflare** (Workers/Pages, connected to the repo) or **GitHub Pages**.
No build step. It's plain HTML.

### 7. First thing in admin
**Settings** tab → her WhatsApp/phone (+233 20 741 1266) is already filled in; check it, then add MoMo number + name, Instagram/TikTok, address, opening hours, delivery fees → **Save**.
Opening hours also control the pickup times customers can book.

---

## How it works

**Customers**
- Browse without an account. To send an order they **sign in / create an account** (name, phone, email, password).
- Checkout: delivery (pickup / Accra / nationwide) → for pickup they **choose a day and time** (only times the shop is open, at least 1 hour ahead) → payment choice → **Send order on WhatsApp** opens a pre-written message to the owner.
- **My account** (person icon, top right): edit name/phone/area, see orders and their status, change email/password, sign out, **delete account**.
- Deleting an account erases the login and profile. Past orders stay for the shop's records with the name and phone removed.

**Owner (admin.html)**
- **Items**: add/edit/delete, up to 8 photos each (shrunk to WebP automatically, ~100–200 KB, good for mobile data), ◀ ▶ to reorder (first = cover), one-tap **Mark sold**.
- **Orders**: *Today's pickups* (sorted by time), *Upcoming pickups*, *Deliveries*, *All*. Set status Confirmed → Ready → Collected, and a **WhatsApp customer** button sends "your order is ready".
- **Settings**: everything shown on the site (contact, MoMo, address, hours, delivery fees).

**Security** (Row Level Security, tested)
- Visitors can only read on-sale items and shop settings.
- Customers can only see/edit their own profile and orders, and can only create orders in their own name.
- Only admins can change products, settings, order status or photos.
- Everything typed in admin is escaped before display (no script injection).
- The anon key in `config.js` is meant to be public. The rules above are what protect the data.

---

## Next steps (not built yet)
- **Online payments**: Paystack or Hubtel for MoMo/Telecel Cash/cards with automatic confirmation. Today she confirms on WhatsApp, then the customer pays manually.
- **Stock reservation**: hold one-of-a-kind items for ~15 min during checkout. Today she confirms availability on WhatsApp.
- **Sign in with Google / phone OTP**: faster sign-up. Phone OTP needs an SMS provider (cost per SMS).
- **Pickup slot limits**: cap how many pickups per time slot if it gets busy.
- **Order spam protection**: if fake orders appear, turn on email confirmation and/or Supabase CAPTCHA (Authentication → Attack Protection).
