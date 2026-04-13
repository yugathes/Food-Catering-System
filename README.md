
# Food Catering Ordering System

## Overview

A web-based food catering management platform that digitises the end-to-end ordering workflow for cafeteria or catering services. The system handles multi-role user access (Admin, Staff, Customer), a categorised menu catalogue, a cart-based checkout flow, bank-transfer payment verification via receipt upload, and automated email notifications at every order status transition. It targets institutional or small-to-medium catering operations that need a lightweight, self-hosted solution with no external SaaS dependency.

---

## Features

- **Role-based access control** — Separate portals and permissions for Admin, Staff, and Customer roles.
- **Categorised menu management** — Create, edit, and delete menu items across six categories: Rice, Curry, Meat, Vegetables, Sides, and Drinks.
- **Cart & checkout** — Session-based cart with quantity control, collection type selection (dine-in / delivery), collection date-time scheduling, and delivery address capture.
- **Order lifecycle management** — Four-stage pipeline: *Waiting for Payment → Waiting for Approval → Approved / Declined*, with status-filtered views.
- **Payment receipt upload** — Customers upload bank transfer receipts (JPG/PNG); Admins verify and approve or decline orders.
- **Automated email notifications** — PHPMailer (SMTP/Gmail) dispatches HTML emails to customers on every status change, including payment instructions and food-collection token.
- **Token-based collection** — Unique collection token generated on order approval for counter pickup.
- **PDF sales reporting** — On-demand A4 PDF reports (FPDF) covering approved orders with itemised pricing and total revenue.
- **Admin dashboard** — Live statistics: total staff, customers, orders, approved/declined/pending counts, and overall sales.
- **User management** — Admin CRUD for user accounts with role assignment and password reset.
- **Staff portal** — Subset of admin capabilities scoped to menu and order management.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Backend** | PHP 7.4 / 8.x (procedural, session-based) |
| **Database** | MySQL / MariaDB 10.4 |
| **Frontend** | HTML5, CSS3, Vanilla JavaScript |
| **Email** | PHPMailer (SMTP via Gmail) |
| **PDF Generation** | FPDF |
| **Local Server** | XAMPP / WAMPP (Apache + MySQL) |

---

## Architecture

The application follows a **role-partitioned MVC-lite structure** with three independently scoped modules:

```
/
├── Auth/          # Authentication — Login, Register, session bootstrap, DB connection
├── Admin/         # Admin panel — Dashboard, menu CRUD, order approval, user management, reports
├── Staff/         # Staff panel — Scoped menu and order views
├── Customer/      # Customer portal — Menu browse, cart, checkout, order history, profile
├── css/           # Shared stylesheets and button assets
├── MenuIMG/       # Uploaded menu item images
├── fpdf/          # FPDF library for PDF generation
├── catering.sql   # Database schema + seed data
└── index.php      # Entry point — redirects to Auth/Login.php
```

**Session management** gates every protected route; unauthenticated requests are redirected to the login page with a 5-second countdown. Database interactions use `mysqli` with direct SQL queries. Email dispatch is triggered server-side via `mail.php` after each order status transition.

**Database schema (key tables):**

| Table | Purpose |
|-------|---------|
| `users` | Stores all user accounts with role (`Admin`, `Staff`, `Customer`) |
| `menu` | Menu catalogue with name, description, price, image, category |
| `orders` | Orders with serialised `menuID` and `quantity` (pipe-delimited), status, receipt, collection metadata |
| `shop` | Shop/vendor registry |

---

## Setup & Installation

### Prerequisites

- XAMPP or WAMPP (Apache 2 + MySQL / MariaDB)
- PHP 7.4 or higher

### Steps

```bash
# 1. Clone or download the repository
git clone https://github.com/yugathes/Food-Catering-System.git

# 2. Move the project to your web server root
cp -r Food-Catering-System/ /xampp/htdocs/Food-Catering-System
# or on Linux/WAMPP: /var/www/html/Food-Catering-System

# 3. Start Apache and MySQL via the XAMPP/WAMPP control panel

# 4. Import the database
#    Open http://localhost/phpmyadmin
#    Create a new database named `catering`
#    Import catering.sql into it

# 5. Verify the database connection
#    Edit Auth/connection.php if your MySQL credentials differ from the defaults:
#      Host: localhost | User: root | Password: (empty) | DB: catering

# 6. Open the application
open http://localhost/Food-Catering-System
```

### Default Credentials

| Role | Username | Password |
|------|----------|----------|
| Admin | `Admin` | `1234` |
| Staff | `Piravin` | `123` |
| Customer | `Yuga` | `1234` |

> **Note:** Change all default passwords before deploying to any non-local environment.

### Email Notifications (Optional)

To enable order status emails, update the SMTP credentials in `mail.php`:

```php
$mail->Username = 'your-email@gmail.com';
$mail->Password = 'your-app-password';       // Use a Gmail App Password
$mail->setFrom('your-email@gmail.com', 'YourCafeteria');
```

---

## Key Workflows

| Endpoint / File | Method | Description |
|-----------------|--------|-------------|
| `Auth/Login.php` | POST | Authenticate user and initialise role session |
| `Auth/Register.php` | POST | Register new Customer account |
| `Customer/CheckOut.php` | POST | Place order — inserts into `orders` with status `0` |
| `Admin/Approve.php` | POST | Update order status (Approve → 2 / Decline → 3) |
| `Admin/Report.php` | GET | Generate and stream PDF sales report |
| `Admin/Menu.php` | GET | List menu items filtered by category |
| `mail.php` | GET | Dispatch HTML email notification for a given order status |

---

## Deployment

The application is designed for **XAMPP/WAMPP self-hosting**. For production deployment:

1. **Apache VHost** — Configure a named virtual host pointing to the project root.
2. **MySQL hardening** — Replace the `root` / no-password connection with a dedicated DB user with least-privilege grants.
3. **HTTPS** — Terminate TLS at Apache or an upstream reverse proxy (e.g., Nginx).
4. **File permissions** — Restrict write access to `Customer/Upload/` and `MenuIMG/` only.
5. **Environment variables** — Move DB credentials and SMTP secrets out of source files into `.env` or Apache `SetEnv` directives.

---

## Future Improvements

- Migrate raw `mysqli` queries to **PDO with prepared statements** to eliminate SQL injection surface.
- Introduce a **PHP framework** (Laravel / Slim) for routing, ORM, and middleware layers.
- Replace pipe-delimited order items with a normalised **`order_items` junction table**.
- Add **JWT or OAuth2** for stateless API authentication.
- Implement a **real-time order status dashboard** via WebSockets or SSE.
- Containerise the stack with **Docker Compose** (PHP-FPM + Nginx + MySQL) for portable deployments.
- Extend to a **mobile-responsive SPA** (Vue.js / React) consuming a RESTful PHP backend.

---

## Author

**Yugathes Subramaniam**  
[github.com/yugathes](https://github.com/yugathes)

