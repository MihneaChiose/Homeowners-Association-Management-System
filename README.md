# Homeowners Association Management System

A full-stack web application for managing homeowners associations. It brings apartment administration, utility meter readings, monthly maintenance calculations, invoices, online payments, issue reporting, and resident communication into one platform.

The application provides separate workspaces for **association administrators** and **residents**, with role-based access to their respective data and workflows.

## Features

### Administrator dashboard

- **Overview:** key metrics for apartments, residents, active issues, monthly income and expenses, and unpaid invoices.
- **Apartment and resident management:** add apartments with their floor, usable area, resident count, and an associated resident account.
- **Utility readings:** monitor submitted meter readings and their history across apartments.
- **Expense management:** record expenses by month, category, amount, and allocation method.
- **Maintenance calculation:** generate apartment-level maintenance invoices from association expenses, review itemized calculation details, and publish monthly maintenance lists.
- **Issue tracking:** view resident reports and move them through `OPEN` → `IN_PROGRESS` → `RESOLVED`.
- **Announcements:** publish, edit, and withdraw notices on the association's digital noticeboard.

### Resident dashboard

- **Personal overview:** apartment information, maintenance amount, outstanding invoices, meter-reading status, and active issues.
- **Meter readings:** submit and review readings for cold water, hot water, gas, and electricity; input is validated against existing readings.
- **Maintenance and invoices:** view published monthly maintenance lists, invoice totals, and an itemized explanation of charges.
- **Online payments:** pay eligible invoices through **Stripe Checkout** and track payment status.
- **Issue reporting:** submit maintenance/administrative issues and follow their progress.
- **Digital noticeboard:** read active announcements from the association administrator.

### Maintenance calculation engine

The calculation service supports expense allocation according to:

| Method | Description |
| --- | --- |
| Per apartment | Equal amount for each apartment |
| Per resident | Proportional to the number of residents |
| By surface area | Proportional to apartment area |
| By consumption | Based on utility consumption determined from meter readings |

Maintenance lists are calculated before publication. Lists in `PUBLISHED` or `CLOSED` status are visible to residents; eligible unpaid invoices can be paid through Stripe.

## Tech stack

| Layer | Technologies |
| --- | --- |
| Frontend | Next.js 16 (App Router), React 19, TypeScript |
| Styling and UI | Tailwind CSS 4, Lucide React |
| Server-side logic | Next.js Server Actions and API Routes |
| Database | PostgreSQL 17 |
| ORM and migrations | Prisma 7, `@prisma/adapter-pg` |
| Authentication | JWT sessions with `jose`, HTTP-only cookies, password hashing with `bcryptjs` |
| Validation | Zod |
| Payments | Stripe Checkout and signed Stripe webhooks |
| Development tooling | Node.js 24, npm, ESLint, Docker Desktop / Docker Engine |

## Project structure

```text
src/
├── app/
│   ├── admin/                # Administrator pages and Server Actions
│   ├── locatar/              # Resident pages and Server Actions
│   ├── api/stripe/webhook/   # Stripe webhook endpoint
│   ├── login/                # Sign in
│   └── register/             # Administrator registration
├── components/              # Shared UI, layouts and invoice details
├── lib/                     # Database, sessions, Stripe and helpers
└── services/
    ├── consumptions/        # Monthly consumption calculations
    └── maintenance/         # Maintenance calculation and distribution rules
prisma/
├── migrations/              # Database migrations
├── schema.prisma            # Database models
└── seed.ts                  # Local demonstration data
prisma.config.ts             # Prisma CLI configuration
```

## Getting started (local development)

These steps were tested with **Windows, PowerShell, Node.js 24, Docker Desktop, PostgreSQL 17, and npm**. Equivalent commands can be used on macOS or Linux.

### Prerequisites

- [Node.js 24](https://nodejs.org/) and npm (a Node version manager such as [nvm-windows](https://github.com/coreybutler/nvm-windows) is optional)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) with its Linux container engine running, or a compatible local PostgreSQL server
- Git
- A [Stripe account](https://dashboard.stripe.com/) in test/sandbox mode **only if you want to test payments**

### 1. Clone the repository

```powershell
git clone https://github.com/MihneaChiose/Homeowners-Association-Management-System.git
cd Homeowners-Association-Management-System
```

If you use nvm-windows:

```powershell
nvm use 24
```

### 2. Install dependencies

```powershell
npm ci
```

`npm ci` installs the versions recorded in `package-lock.json`. The `dotenv` dependency is included in the updated project; no separate installation is needed.

### 3. Start PostgreSQL

Make sure Docker Desktop is running, then create a PostgreSQL 17 container:

```powershell
docker run -d --name licenta-postgres -e POSTGRES_USER=licenta -e POSTGRES_PASSWORD=licenta_dev_2026 -e POSTGRES_DB=licenta_db -p 5432:5432 -v licenta-pg-data:/var/lib/postgresql/data postgres:17
```

The credentials above are **development-only examples**. Use different, strong credentials outside an isolated local environment.

Verify that the database accepts connections:

```powershell
docker exec licenta-postgres pg_isready -U licenta -d licenta_db
```

If the container already exists from an earlier setup, start it instead:

```powershell
docker start licenta-postgres
```

The named Docker volume `licenta-pg-data` keeps database data between container restarts.

### 4. Configure environment variables

Create a `.env` file in the repository root (next to `package.json` and `prisma.config.ts`):

```dotenv
DATABASE_URL="postgresql://licenta:licenta_dev_2026@localhost:5432/licenta_db?schema=public"
JWT_SECRET="REPLACE_WITH_A_RANDOM_SECRET"
APP_URL="http://localhost:3000"
STRIPE_SECRET_KEY="sk_test_REPLACE_WITH_YOUR_TEST_KEY"
STRIPE_WEBHOOK_SECRET="whsec_REPLACE_WITH_YOUR_LOCAL_WEBHOOK_SECRET"
```

Generate a strong random value for `JWT_SECRET`:

```powershell
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

The Stripe values above are **placeholders**, not working credentials. Replace them before testing payments. Keep `.env` private; it is excluded by `.gitignore` and must **never** be committed.

### 5. Prepare the database

Run the existing migrations, generate Prisma Client, and insert demonstration data:

```powershell
npm run prisma:migrate
npm run prisma:generate
npm run db:seed
```

The seed creates an administrator, a resident, an association, an apartment, and utility meters. **It does not create maintenance invoices or payment records.**

### 6. Start the application

```powershell
npm run dev
```

Open **http://localhost:3000**. The sign-in page is at **http://localhost:3000/login**.

## Demo accounts

The database seed creates these **local-only** accounts:

| Role | Email | Password | Landing page |
| --- | --- | --- | --- |
| Administrator | `admin@test.com` | `admin123` | `/admin/dashboard` |
| Resident | `locatar@test.com` | `locatar123` | `/locatar/dashboard` |

> **Security:** These are intentionally predictable development credentials. Do not deploy seeded accounts to a public environment. The current apartment-creation flow also provisions new resident accounts with a fixed initial password (`locatar123`), which must be replaced with a secure onboarding/password-setup flow before production use.

## How to use the application

1. **Sign in as administrator** using the local demo account, or register a new administrator and association at `/register`.
2. Go to **Apartments** (`/admin/apartamente`) to manage apartment and resident information. The seed already provides one sample apartment.
3. **Sign in as resident** and open **Meter Readings** (`/locatar/contoare`) to submit readings for an eligible period. The administrator can review them under `/admin/contoare`.
4. Back in the administrator account, open **Expenses** (`/admin/cheltuieli`) and enter a monthly expense with an allocation method. For a quick demo without meter readings, choose **Per apartment**.
5. Open **Maintenance** (`/admin/intretinere`), select the same month and year, **calculate** the maintenance list, review the invoices, and **publish** it.
6. Sign in as the resident and open **Maintenance & Payments** (`/locatar/intretinere`) to inspect the published invoice and its charge breakdown.
7. With Stripe test mode configured, select **Pay online** to complete a simulated checkout. The invoice is marked as paid after the application receives the verified Stripe webhook.
8. Explore **Issues** and the **Noticeboard** from both roles to test reporting, status updates, and association announcements.

## Stripe payments: local test setup

The application uses Stripe Checkout with **RON** amounts and processes these webhook events:

- `checkout.session.completed` — confirms a successful payment and updates the payment/invoice records.
- `checkout.session.expired` — marks an expired pending payment attempt as rejected.

1. Obtain your **test secret API key** (`sk_test_...`) from the [Stripe Dashboard](https://dashboard.stripe.com/apikeys), and set `STRIPE_SECRET_KEY` in `.env`.
2. [Install the Stripe CLI](https://docs.stripe.com/stripe-cli) and authenticate in your test environment:

   ```powershell
   stripe login
   ```

3. In a **separate terminal**, forward Stripe events to your local webhook route:

   ```powershell
   stripe listen --events checkout.session.completed,checkout.session.expired --forward-to http://localhost:3000/api/stripe/webhook
   ```

4. Copy the CLI-generated `whsec_...` signing secret into `STRIPE_WEBHOOK_SECRET` in `.env`, then **restart `npm run dev`** so the new environment variable is loaded.
5. Keep both the Next.js development server and `stripe listen` running. Create and publish an unpaid maintenance invoice, then initiate a resident payment in the application.
6. On Stripe Checkout, use the standard **test card** `4242 4242 4242 4242`, any future expiry date, and any three-digit CVC. **No real money is transferred in test mode.**
7. Confirm that the forwarded event receives an HTTP 2xx response and that the invoice status changes to **Paid**.

**Important:** returning to the application from Stripe Checkout does not itself mark an invoice as paid; the signed webhook must be processed successfully. The `whsec_...` value from `stripe listen` can differ from a webhook secret configured in the Dashboard.

## Useful commands

| Command | Purpose |
| --- | --- |
| `npm run dev` | Run the development server |
| `npm run build` | Build the production bundle |
| `npm run start` | Start the compiled application |
| `npm run lint` | Run ESLint |
| `npm run prisma:migrate` | Apply/create development migrations |
| `npm run prisma:generate` | Generate Prisma Client |
| `npm run db:seed` | Insert local demo data |
| `npm run prisma:studio` | Inspect the database with Prisma Studio |
| `npm audit` | Check dependencies for known vulnerabilities |

### Starting again after a reboot

Once the initial setup is complete, you normally only need to start Docker Desktop and run:

```powershell
docker start licenta-postgres
nvm use 24    # if using nvm-windows
npm run dev
```

Launch `stripe listen` in a second terminal **only when testing payments**. You do not need to repeat the migrations and seed on every startup.

## Security and deployment notes

- Passwords are hashed with `bcryptjs`; sessions are JWT-based and stored in HTTP-only cookies.
- Sensitive actions check the authenticated user's role and associated records.
- Stripe webhook signatures are verified before payment status is updated.
- Keep `DATABASE_URL`, `JWT_SECRET`, `STRIPE_SECRET_KEY`, and `STRIPE_WEBHOOK_SECRET` out of source control.
- The demo credentials, fixed resident initial password, and `/test-db` diagnostic route are intended for **local development** and should be addressed before any public deployment.
- Run `npm audit`, `npm run lint`, and `npm run build` after changing dependencies.
