# RideRent Vehicle Rental System

This delivery implements the **18 customer HTML pages listed in your latest request**, their CSS and JavaScript, and the PHP/MySQL APIs needed to use them. The requested administrator PHP backend is now implemented; see BACKEND-API.md for its endpoint contracts. The admin login and overview dashboard are implemented. Other admin management HTML screens remain placeholders.

## Run in XAMPP

1. Extract `RideRent.zip` and copy the `RideRent` folder to `C:\xampp\htdocs\RideRent`.
2. Start **Apache** and **MySQL** in XAMPP Control Panel.
3. Open `http://localhost/phpmyadmin/`.
4. Choose **Import**, select `RideRent/database/riderent.sql`, and click **Go**. The file creates `riderent`, its tables, and sample records. Import it once into a new database. It does not erase an existing installation.
5. Visit `http://localhost/RideRent/index.html`.

The folder must contain `index.html` directly; avoid an extra nested `RideRent` folder. Opening HTML using a `file://` URL will not run PHP APIs.

Default connection in `backend/config/database.php`: host `localhost`, database `riderent`, user `root`, blank password. Tested with XAMPP PHP 8.2 and MariaDB 10.4. MySQL 8+ is also targeted. The PDO MySQL extension must be enabled.

## Demo customer accounts

| Email | Password |
| --- | --- |
| `aarav@example.com` | `password` |
| `maya@example.com` | `password` |
| `rohan@example.com` | `password` |

You can also register a new account. Aarav has a completed rental eligible for a review. Maya has a sample review. Demo prices use Nepalese rupees (NPR); contact information, specifications, and vehicle illustrations are demonstration content.

The administrator API accepts `admin@riderent.example`, password `password`. Open `http://localhost/RideRent/admin/login.html` to log in and view the dashboard. Other management screens remain placeholders; BACKEND-API.md documents their backend APIs.

## Implemented customer pages

| File | Behavior |
| --- | --- |
| `index.html` | Home, date/category search, featured vehicles from MySQL, categories, benefits, and rental steps. |
| `vehicles.html` | Search, category/brand/fuel/transmission/price/date filters, sort, and pagination. |
| `vehicle-details.html` | Vehicle data, date/price estimate, booking link, average rating, and reviews. |
| `login.html` | Session login and optional 30-day remember-me cookie. |
| `register.html` | Validated registration and duplicate-email handling. |
| `forgot-password.html` | Request a reset email; a token URL on the same page displays the new-password form. |
| `booking.html` | Authenticated summary and booking confirmation; all prices are recalculated server-side. |
| `booking-confirmation.html` | Owned booking summary and payment records. |
| `my-bookings.html` | Owned bookings, status filters, and eligible cancellation. |
| `booking-details.html` | Customer/rental/payment details, cancellation, and review link. |
| `profile.html` | Current user's profile and booking count. |
| `edit-profile.html` | Update name, phone, address, and licence. Email is not editable. |
| `change-password.html` | Verify current password and save a new password hash. |
| `about.html` | About, mission, vision, and database-backed statistics. |
| `contact.html` | Validated contact form stored in MySQL. |
| `faq.html` | Accessible expandable rental questions. |
| `terms.html` | Academic-project rental terms. |
| `privacy.html` | Academic-project data-handling notice. |

## Architecture and folders

- Root `.html` files contain frontend page markup. They contain no PHP.
- `assets/css/style.css` supplies the responsive customer design system.
- `assets/js/main.js` builds shared navigation/footer and provides fetch, alerts, formatting, escaping, and form helpers.
- `assets/js/auth.js`, `vehicles.js`, `booking.js`, and `profile.js` connect customer pages to their APIs.
- `assets/images/logo/` and `assets/images/vehicles/` contain original local SVG illustrations, so images work offline.
- `backend/auth/`, `vehicles/`, `bookings/`, `users/`, `contact/`, `reviews/`, and `public/` expose narrowly named PHP endpoint files.
- `backend/api.php` dispatches those endpoints to customer backend logic. Keeping route handlers together simplifies this semester project; it never generates HTML.
- `backend/helpers/functions.php` centralizes JSON output, session/CSRF checks, validation, prepared queries, pricing, and availability.
- `backend/config/database.php` creates the reusable PDO connection.
- `database/riderent.sql` creates the eight domain tables and three supporting tables for remember tokens, password resets, and rate limiting.

## API conventions

Responses use `{ "success": true, "message": "...", "data": {} }`. Errors set an appropriate HTTP status and use `success: false`.

Session status is nested inside `data`: `data.loggedIn`, `data.user`, and `data.csrfToken`. GET is used for retrieval. State-changing endpoints accept JSON POST requests and require `X-CSRF-Token`, obtained from `backend/auth/session.php`. JavaScript sends same-origin cookies with `fetch()`.

Customer identity comes from the session, never a submitted user ID. Responses omit password hashes. Output inserted into HTML templates is escaped; text-only UI uses `textContent`. Image URLs are restricted to local vehicle assets.

## Availability and booking rules

Rental intervals use an exclusive return date: `[pickup_date, return_date)`. Existing bookings overlap when `existing.pickup_date < requested.return_date AND existing.return_date > requested.pickup_date`. Pending, Confirmed, and Active bookings block a date range. Adjacent rentals can share the return/pickup boundary.

Booking creation opens an InnoDB transaction and locks the selected vehicle with `SELECT ... FOR UPDATE` before checking overlaps and inserting. Concurrent customers therefore cannot reserve the same vehicle dates. Any future administrator endpoint that creates or reactivates bookings must use the same locking rule. Prices and totals are obtained from the database, not JavaScript. Rentals must be 1–365 days and cannot start in the past.

Bookings begin as Pending with a pending cash payment record. The site does not process money. The new administrator APIs can confirm/start/complete bookings and update payment records; administrator management screens beyond the login/dashboard remain to be implemented. Existing completed sample bookings support review testing. Pending and Confirmed bookings can be cancelled; paid amounts are not automatically refunded.

## Password recovery and configuration

Password-reset tokens are random, hashed in the database, expire after 30 minutes, and can be redeemed once. **Actual reset-email delivery requires configuring PHP `mail()`/SMTP in XAMPP.** The site never displays a reset token in an API response. Request messages are generic to avoid exposing registered email addresses.

Optional server environment variables: `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `APP_URL`, and `APP_TIMEZONE`. `APP_URL` defaults to `http://localhost/RideRent` and must match the real site URL for reset links. The frontend uses Kathmandu time and NPR; update the frontend formatters too if changing locale.

Use demo data for local coursework. Before real deployment, replace seeded credentials and contact details, configure email and HTTPS, and establish real rental/privacy policies. Password changes revoke remember-me tokens but do not forcibly terminate every already-active session on other devices. Session cookies are HttpOnly and SameSite=Lax, and Secure when served over HTTPS.

## Validation and troubleshooting

See `TESTING.md` for executed tests and limitations. The API tests used an isolated temporary database and did not import or change your normal `riderent` database.

- **Database unavailable:** start MySQL, import the SQL, and check PDO credentials.
- **JSON/server error:** use the Apache URL and verify PHP is enabled. Detailed errors go to the PHP error log instead of the browser.
- **Admin access:** use `admin/login.html`. Login and dashboard work; other management HTML pages remain placeholders.
- **No reset email:** configure PHP mail transport and `APP_URL`.
- **Import says table already exists:** the schema has already been imported; do not re-import over existing records.
- **Apache 403/500:** verify local Apache permissions and support for the included `.htaccess` directives. The project disables directory listings and blocks HTTP access to SQL/config folders.
