# **PETORA — Sistem Manajemen Terpadu Petshop & Petcare**
## **Dokumen Spesifikasi Teknis Lengkap**

---

## **I. DESKRIPSI PROYEK & VISI**

**PETORA** adalah aplikasi web enterprise yang dirancang sebagai **sistem manajemen terintegrasi penuh** untuk bisnis petshop dan petcare (klinik hewan, grooming salon, pet hotel) dengan cakupan operasional dari manajemen pelanggan, perawatan hewan, hingga point-of-sale dan manajemen keuangan.

### **Tujuan Utama**

Petora dibangun untuk menyelesaikan **tiga masalah utama** dalam industri petcare Indonesia:

1. **Fragmentasi operasional** — Operasi petshop, klinik, dan pet hotel dikelola dengan sistem terpisah. Petora menyatukan semuanya dalam satu ekosistem.

2. **Ketidakefisienan data pasien/pelanggan** — Riwayat medis, pemeliharaan hewan, dan preferensi pelanggan tersebar di berbagai tempat. Petora menciptakan single source of truth.

3. **Kurangnya insights bisnis** — Pemilik sulit melacak profitabilitas dan performa. Petora menyediakan dashboard dan laporan analitik real-time.

### **Visi Akhir**

Menjadi operating system untuk petcare Indonesia dengan kemampuan multi-lokasi, terintegrasi dengan payment gateway, WhatsApp, dan business intelligence tools.

---

## **II. ARSITEKTUR SISTEM**

### **A. Dua Permukaan Aplikasi**

Petora memiliki **dua interface berbeda** untuk dua audience:

#### **1. Staff Dashboard** (`/app/*`)
- **Target user**: Owner, Admin, Dokter, Kasir
- **Karakteristik**: Desktop-first, sidebar navigation, data-dense, keyboard shortcuts
- **Tujuan**: Manajemen operasional penuh bisnis
- **Aksesibilitas**: Hanya untuk staff yang login dengan PIN
- **Fitur utama**: Semua modul bisnis (CRM, appointments, POS, inventory, reports, dll)

#### **2. Customer Portal** (`/portal/*`)
- **Target user**: Pelanggan (pet owners)
- **Karakteristik**: Mobile-first, bottom navigation, simple & friendly
- **Tujuan**: Self-service untuk pelanggan
- **Aksesibilitas**: Terbuka untuk pelanggan yang sudah register
- **Fitur utama**: View riwayat hewan, booking appointments, lihat invoices, loyalty points, shop produk

### **B. Arsitektur Teknis**

**Frontend Layer**
- Framework: SolidJS 1.8 dengan TypeScript strict mode
- Build tool: Vite 5 dengan HMR
- Routing: @solidjs/router 0.10 dengan file-based routing dan lazy loading
- State management: SolidJS Signals (fine-grained reactivity), Stores (nested state), Resources (async data), Context (global state)
- Data fetching: @tanstack/solid-query 5 untuk server state management dengan automatic caching dan background refetch
- Styling: Tailwind CSS v4 dengan utility-first approach
- UI components: @kobalte/core untuk headless accessible components, custom wrapper dengan Tailwind
- Icons: lucide-solid
- Forms: @modular-forms/solid dengan Zod validation
- Charts: solid-charts untuk visualisasi data
- Notifications: Custom toast system dengan in-app notifications

**Backend Layer**
- Platform: Supabase (managed PostgreSQL + Auth + Edge Functions + Storage)
- Database: PostgreSQL 15+ dengan Row Level Security (RLS)
- API: PostgREST (auto-generated REST API) + Supabase Client (type-safe queries)
- Edge Functions: Deno runtime untuk custom business logic
- Realtime: WebSocket subscriptions untuk live updates
- Storage: S3-compatible untuk file uploads

**External Services**
- Payment gateway: Midtrans (QRIS, bank transfer, e-wallet, credit card)
- WhatsApp API: Fonnte (appointment reminders, invoices, notifications)
- Email: Resend (invoices, confirmations)
- Monitoring: Sentry (error tracking, performance monitoring)

**Deployment**
- Frontend: Vercel (auto-deploy dari GitHub)
- Backend: Supabase Cloud (managed PostgreSQL + serverless functions)
- Database: Supabase PostgreSQL (encrypted, backed up)
- Files: Supabase Storage (S3-compatible)
- Monitoring: Sentry (error tracking), Lighthouse (performance)

---

## **III. STACK TEKNOLOGI LENGKAP**

### **Frontend Stack**

**Core Framework**
- SolidJS 1.8 — Reactive UI library dengan fine-grained reactivity, tanpa virtual DOM, performa tinggi
- TypeScript 5.3 — Strict mode, type safety penuh, no implicit any
- Vite 5 — Build tool dengan fast dev server, HMR, optimized production build

**Routing & Navigation**
- @solidjs/router 0.10 — File-based routing, nested routes, lazy loading, route guards
- Konvensi: Route definitions di `src/routes/` dengan struktur folder yang merepresentasikan hierarchy

**State Management**
- Signals (`createSignal`) — Primitive reactive values untuk state sederhana
- Stores (`createStore`) — Complex nested reactive objects untuk state terstruktur
- Resources (`createResource`) — Async data fetching dengan automatic caching, refetch, dan error handling
- Context (`createContext`) — Global state sharing untuk auth, theme, user preferences

**Data Fetching**
- @tanstack/solid-query 5 — Server state management, smart caching, background refetch, optimistic updates
- Supabase Client — Direct database queries dengan RLS enforcement, type-safe

**Styling**
- Tailwind CSS v4 — Utility-first CSS, JIT compilation
- tailwind-merge — Conditional class merging untuk avoid conflicts
- clsx — Conditional class names dengan syntax yang clean
- Konvensi: Utility classes langsung di JSX, component variants via props

**UI Components**
- @kobalte/core — Headless accessible components (dialogs, menus, tabs, dropdowns, tooltips)
- Custom components di `src/components/ui/` — Wrapper dengan styling Tailwind, consistent API
- Icons: Lucide Solid (`lucide-solid`) — Icon system yang comprehensive

**Forms & Validation**
- @modular-forms/solid — Form management dengan SolidJS integration, type-safe
- Zod — Schema validation (runtime + TypeScript inference), reusable schemas
- Konvensi: Schema di `src/lib/validations/`, reusable di frontend & backend

**Charts & Visualization**
- solid-charts — Charting library native untuk SolidJS
- Recharts alternative — Chart.js dengan wrapper SolidJS jika diperlukan

**Utilities**
- date-fns — Date manipulation, formatting, parsing
- zod — Runtime validation
- nanoid — Unique ID generation
- qrcode — QR code generation untuk ID card hewan
- html2canvas — Screenshot/PDF generation untuk receipts dan reports

**Notifications**
- Custom toast system — In-app notifications dengan auto-dismiss
- Notification center — Bell icon dengan dropdown untuk view all notifications

### **Backend Stack**

**Platform**
- Supabase — Managed PostgreSQL + Auth + Edge Functions + Storage
- PostgreSQL 15+ — Database engine dengan full SQL support
- Deno 1.40+ — Edge Functions runtime, secure by default

**API Layer**
- PostgREST — Auto-generated REST API dari database schema, zero-code CRUD
- Supabase Client — Type-safe database queries, automatic TypeScript types
- Edge Functions — Custom business logic, integrations, complex workflows

**Authentication**
- Supabase Auth — Session management, JWT tokens, HTTP-only cookies
- Custom PIN-based auth — Edge Function untuk validasi PIN dengan bcrypt
- RLS (Row Level Security) — Database-level access control, enforce di setiap query

### **External Services**

**Payment Gateway**
- Midtrans — QRIS, bank transfer, e-wallet, credit card
- Webhook handler — Edge Function untuk payment status updates, auto-confirm invoices

**Communication**
- Fonnte — WhatsApp API (appointment reminders, invoices, notifications, pet hotel updates)
- Resend — Email service (invoices, confirmations, feedback requests)

**Monitoring & Analytics**
- Sentry — Error tracking, performance monitoring, release management
- Supabase Analytics — Database query performance, slow query detection

### **Development Tools**

**Build & Bundle**
- Vite — Dev server & build tool dengan optimized chunks
- TypeScript — Type checking dengan strict mode
- ESLint — Code linting dengan custom rules
- Prettier — Code formatting dengan consistent style

**Testing**
- Vitest — Unit testing untuk SolidJS components dan utilities
- @testing-library/solid — Component testing dengan user-centric approach
- Playwright — E2E testing untuk critical workflows

**Git & CI/CD**
- Husky — Git hooks (pre-commit, pre-push) untuk enforce quality
- lint-staged — Run linters on staged files only
- GitHub Actions — CI/CD pipeline (lint, test, build, deploy to Vercel)

---

## **IV. STRUKTUR PROYEK FRONTEND**

### **Root Directory Structure**

```
petora/
├── src/
│   ├── app/                    # Staff Dashboard
│   │   ├── routes/             # Route definitions (file-based)
│   │   ├── layouts/            # Layout components (sidebar, header)
│   │   ├── pages/              # Page components (Dashboard, CustomerList, dll)
│   │   └── components/         # Dashboard-specific components
│   │
│   ├── portal/                 # Customer Portal
│   │   ├── routes/             # Route definitions
│   │   ├── layouts/            # Layout components (bottom nav)
│   │   ├── pages/              # Page components (Home, MyPets, dll)
│   │   └── components/         # Portal-specific components
│   │
│   ├── components/             # Shared components
│   │   ├── ui/                 # UI primitives (Button, Input, Dialog, Tabs, dll)
│   │   ├── forms/              # Form components (FormField, FormError, FormSelect)
│   │   ├── data-display/       # Data display (Table, Card, Badge, List, Timeline)
│   │   ├── feedback/           # Feedback (Alert, Toast, Modal, ConfirmationDialog)
│   │   └── layout/             # Layout helpers (Container, Stack, Grid, Divider)
│   │
│   ├── lib/                    # Core utilities
│   │   ├── supabase/           # Supabase client configuration & helpers
│   │   ├── services/           # Business logic services (customerService, petService)
│   │   ├── validations/        # Zod schemas (customerSchema, petSchema)
│   │   ├── hooks/              # Custom SolidJS hooks (useAuth, useDebounce)
│   │   ├── utils/              # Utility functions (format, calculate, validate)
│   │   ├── constants/          # Constants & enums (roles, statuses, categories)
│   │   └── types/              # TypeScript type definitions
│   │
│   ├── stores/                 # Global state (Context + Stores)
│   │   ├── auth.ts             # Authentication state (user, session, login/logout)
│   │   ├── theme.ts            # Theme state (light/dark mode)
│   │   └── user.ts             # User preferences (language, notifications)
│   │
│   ├── styles/                 # Global styles
│   │   ├── globals.css         # Tailwind imports, base styles
│   │   └── theme.css           # Theme variables (colors, spacing)
│   │
│   ├── App.tsx                 # Root app component
│   ├── index.tsx               # Entry point
│   └── router.tsx              # Router configuration
│
├── public/                     # Static assets
│   ├── favicon.ico
│   └── images/
│
├── supabase/                   # Supabase configuration
│   ├── functions/              # Edge Functions (validate-pin, midtrans-webhook)
│   ├── migrations/             # Database migrations (SQL files)
│   └── seed.sql                # Seed data untuk development
│
├── tests/                      # Test files
│   ├── unit/                   # Unit tests (services, utils, hooks)
│   ├── integration/            # Integration tests (components, flows)
│   └── e2e/                    # E2E tests (Playwright)
│
├── .env                        # Environment variables (local)
├── .env.example                # Environment template
├── package.json                # Dependencies & scripts
├── tsconfig.json               # TypeScript config (strict mode)
├── vite.config.ts              # Vite config (plugins, aliases)
├── tailwind.config.ts          # Tailwind config (theme, plugins)
├── eslint.config.js            # ESLint config (rules, plugins)
└── README.md                   # Project documentation
```

### **Konvensi Penamaan File**

**Component files**: kebab-case (customer-list.tsx) atau PascalCase (CustomerList.tsx) — konsisten dalam satu proyek
**Custom hooks**: camelCase dengan prefix 'use' (useCustomer.ts, useAuth.ts)
**Service files**: camelCase dengan suffix 'Service' (customerService.ts, petService.ts)
**Validation schemas**: camelCase dengan suffix 'Schema' (customerSchema.ts, petSchema.ts)
**Type definitions**: kebab-case dengan suffix '.types' (customer.types.ts)
**Test files**: kebab-case dengan suffix '.test' (customer.test.ts)

### **Konvensi Penamaan Komponen**

**Component names**: PascalCase (CustomerList, PetCard, AppointmentCalendar)
**Props interfaces**: PascalCase dengan suffix 'Props' (CustomerListProps, PetCardProps)
**Event handlers**: camelCase dengan prefix 'on' (onCustomerSelect, onPetClick)
**Constants**: UPPER_SNAKE_CASE (MAX_CUSTOMERS_PER_PAGE, DEFAULT_SORT_ORDER)

---

## **V. KONVENSI & STANDAR KODE**

### **SolidJS Patterns**

**Signals (Primitive Reactive Values)**
Digunakan untuk simple reactive values seperti counters, toggles, input values. Signals adalah primitive yang paling efisien di SolidJS, hanya re-render component yang menggunakan signal tersebut.

**Derived Signals (Computed Values)**
Digunakan untuk computed values yang bergantung pada signals lain. Automatically update saat dependencies berubah. Contoh: fullName dari firstName dan lastName.

**Effects (Side Effects)**
Digunakan untuk side effects seperti logging, API calls, subscriptions. Automatically re-run saat dependencies berubah. Harus digunakan dengan hati-hati untuk avoid infinite loops.

**Stores (Complex Nested State)**
Digunakan untuk complex nested objects seperti shopping cart, form data, configuration. Stores memungkinkan update partial tanpa re-create entire object.

**Resources (Async Data Fetching)**
Digunakan untuk async data fetching dengan automatic caching, refetch, dan error handling. Resources automatically fetch data saat component mount dan re-fetch saat dependencies berubah.

**Context (Global State)**
Digunakan untuk global state sharing seperti authentication, theme, user preferences. Context harus digunakan sparingly, prefer signals dan stores untuk local state.

### **Routing Patterns**

**File-based routing**: Route definitions di `src/routes/` dengan struktur folder yang merepresentasikan hierarchy. Contoh: `src/routes/app/customers/[id].tsx` → `/app/customers/:id`

**Lazy loading**: Semua page components di-lazy load untuk code-splitting dan faster initial load. Gunakan `lazy()` dari SolidJS.

**Route guards**: Authentication guards untuk protect routes. Redirect ke `/login` jika user belum login. Role-based guards untuk restrict access berdasarkan role.

**Nested routes**: Layout components untuk shared UI (sidebar, header). Child routes render di `<Outlet />` component.

### **Service Layer Pattern**

**Single responsibility**: Setiap service bertanggung jawab untuk satu domain (customerService, petService, appointmentService).

**Type-safe**: Semua service methods menggunakan TypeScript types. Input dan output harus didefinisikan dengan jelas.

**Error handling**: Service methods harus throw errors yang descriptive. Frontend components catch errors dan display user-friendly messages.

**Validation**: Input validation dengan Zod schemas sebelum send ke database. Validate di frontend dan backend untuk defense in depth.

**Pagination**: Semua list endpoints support pagination dengan page dan limit parameters. Return total count untuk pagination UI.

**Filtering & sorting**: Support filtering by multiple fields dan sorting by any column. Default sort order harus didefinisikan.

### **Validation Pattern (Zod)**

**Schema definitions**: Semua validation schemas di `src/lib/validations/`. Reusable di frontend (form validation) dan backend (Edge Functions).

**Type inference**: Gunakan `z.infer<typeof schema>` untuk generate TypeScript types dari schemas. Avoid duplicate type definitions.

**Custom error messages**: Semua validation errors harus memiliki custom error messages dalam Bahasa Indonesia yang user-friendly.

**Partial validation**: Support partial validation untuk update operations. Gunakan `schema.partial()` untuk make all fields optional.

### **Component Pattern**

**Props splitting**: Gunakan `splitProps()` untuk separate custom props dari native HTML props. Avoid prop drilling.

**Default values**: Gunakan `mergeProps()` untuk define default values. Avoid conditional rendering untuk default props.

**Conditional rendering**: Gunakan `<Show>` component dari SolidJS untuk conditional rendering. Avoid ternary operators untuk complex conditions.

**List rendering**: Gunakan `<For>` component dari SolidJS untuk list rendering. Selalu provide `key` prop untuk efficient updates.

**Event handling**: Event handlers harus defined di component level, bukan inline. Gunakan arrow functions untuk preserve `this` context.

### **Form Pattern**

**Form library**: Gunakan @modular-forms/solid untuk form management. Type-safe, integrated dengan SolidJS.

**Validation**: Validate on submit dan on blur. Show error messages inline di bawah each field.

**Loading state**: Disable submit button saat form is submitting. Show loading spinner di button.

**Success handling**: Call `onSuccess` callback setelah successful submit. Close modal atau navigate ke list page.

**Error handling**: Show error toast jika submit gagal. Keep form data intact untuk user retry.

---

## **VI. MODUL-MODUL BISNIS**

### **Modul 1: Authentication & User Management**

**Tujuan**: Mengamankan aplikasi dan mengelola akses berdasarkan role.

#### **Fitur Utama**

**PIN-Based Login**
- User login dengan username + 6-digit PIN (bukan password)
- Username adalah unique, case-sensitive, alphanumeric
- PIN adalah 6-digit numeric, di-hash dengan bcrypt (salt rounds 12)
- Session token di-store di HTTP-only cookie untuk security
- JWT session validity: 24 jam, auto-refresh sebelum expiry
- Remember me option: extend session to 7 days

**Role-Based Access Control (RBAC)**
5 role berbeda dengan permission matrix:

| Role | Deskripsi | Akses |
|------|-----------|-------|
| `OWNER` | Pemilik bisnis | Akses penuh, bisa reset PIN staff, lihat laporan keuangan, approve expenses |
| `ADMIN` | Manajer bisnis | Kelola customer, staff, inventory, tidak lihat keuangan sensitif |
| `DOKTER` | Dokter/veterinarian | Kelola medical records, appointments, treatment, tidak bisa edit inventory |
| `KASIR` | Kasir/receptionist | POS, checkout, customer service, tidak bisa lihat medical records |
| `CUSTOMER` | Pelanggan | Hanya portal customer, lihat invoice, booking, tidak bisa akses dashboard |

**Permission Matrix Detail**
- OWNER: full access ke semua modul, termasuk financial reports dan expense approvals
- ADMIN: access ke CRM, appointments, inventory, reports (exclude financial), tidak bisa delete data
- DOKTER: access ke appointments, medical records, customer view (read-only), tidak bisa edit products
- KASIR: access ke POS, customer view (read-only), appointments view (read-only), tidak bisa lihat medical records
- CUSTOMER: access ke portal only, bisa view own data, booking appointments, view invoices

**Security Features**
- Failed login lockout: 5 consecutive failed attempts → akun di-lock 15 menit
- Lockout notification: tampilkan pesan "Akun terkunci. Coba lagi dalam X menit"
- Audit logging: semua login attempts tercatat (success, failed, locked)
- Session invalidation: logout dari semua devices saat PIN changed
- IP tracking: record IP address untuk setiap login attempt
- Device fingerprinting: detect suspicious login dari new device

**User Management**
- Create user: Owner bisa create user dengan role ADMIN, DOKTER, KASIR
- Initial PIN: auto-generated 6-digit random PIN, Owner bisa set custom PIN
- Deactivate user: soft delete (user masih ada di DB, tapi is_active = false)
- Reactivate user: Owner bisa reactivate deactivated user
- Change PIN: user bisa change own PIN dengan verify old PIN first
- Reset PIN: Owner bisa reset PIN siapa saja, generate random atau set custom
- Force logout: Owner bisa force logout user dari semua devices

**Audit Trail**
- Semua user actions tercatat: create, update, delete, login, logout
- Audit log fields: user_id, action, entity_type, entity_id, before_values, after_values, timestamp, ip_address
- Audit log immutable: tidak bisa di-edit atau di-delete
- Audit log retention: keep untuk 2 tahun, archive setelah itu

#### **Business Rules**

**Login Flow**
1. User input username + PIN
2. System check username exists dan is_active = true
3. System check akun tidak locked (locked_until < now)
4. System verify PIN dengan bcrypt.compare
5. Jika PIN salah: increment failed_login_attempts, check if >= 5 → lock akun 15 menit
6. Jika PIN benar: reset failed_login_attempts, generate JWT session, record last_login_at
7. Return user data + session token

**PIN Reset Flow**
1. Owner/Admin select user
2. System generate random 6-digit PIN atau Owner input custom PIN
3. System hash PIN dengan bcrypt
4. System update user record
5. System invalidate all active sessions untuk user tersebut
6. System log audit trail
7. Display new PIN ke Owner (one-time view)

**Session Management**
1. Login success → generate JWT dengan payload: user_id, role, exp (24h)
2. JWT stored di HTTP-only cookie (secure, sameSite: strict)
3. Setiap request: validate JWT, check exp, refresh jika needed
4. Logout: clear cookie, invalidate session di database
5. Force logout: invalidate all sessions untuk user

#### **Edge Cases**

**Concurrent login dari multiple devices**
- Allow: user bisa login dari multiple devices simultaneously
- Session tracking: record semua active sessions per user
- Force logout: invalidate specific session atau all sessions

**PIN change saat ada active sessions**
- Invalidate all active sessions setelah PIN change
- User harus login ulang di semua devices
- Notification: email user tentang PIN change

**Locked account recovery**
- Owner bisa unlock akun yang locked
- Reset failed_login_attempts ke 0
- Clear locked_until timestamp
- Log audit trail

**Expired session handling**
- Detect expired session di frontend
- Redirect ke login page
- Preserve current URL untuk redirect back setelah login
- Show message "Session expired. Please login again"

---

### **Modul 2: CRM (Customer Relationship Management)**

**Tujuan**: Kelola database pelanggan dan hubungan jangka panjang.

#### **Fitur Utama**

**Customer Management**

**Create Customer**
- Input fields:
  - Nama lengkap (required, min 2 chars, max 100 chars)
  - Nomor HP (required, format: +62/62/08xx, validate dengan regex)
  - Email (optional, validate format)
  - Alamat (optional, max 500 chars)
  - Kontak darurat (optional, nama + nomor HP)
  - Foto profil (optional, upload image, max 2MB, formats: jpg/png/webp)
  - Catatan khusus (optional, max 1000 chars, untuk alergi/preferensi)
  - Tags (optional, multi-select: VIP, Regular, New, Blacklist)

- Validation rules:
  - Nama: required, unique (case-insensitive), no special characters except space, dash, apostrophe
  - Nomor HP: required, unique, must match Indonesian phone format
  - Email: optional, must be valid email format, unique jika diisi
  - Tags: optional, array of enums, max 5 tags

- Auto-generated fields:
  - Customer ID (UUID)
  - Created at timestamp
  - Updated at timestamp
  - Deleted at timestamp (null initially)

**Customer List View**
- Display: table dengan columns: foto, nama, nomor HP, email, tags, created date, actions
- Search: by nama or nomor HP (case-insensitive, partial match)
- Filter: by tags (multi-select), by created date range
- Sort: by nama, nomor HP, created date (asc/desc)
- Pagination: 50 items per page, show total count
- Bulk actions: select multiple customers → bulk tag, bulk export

**Customer Detail Page**
- Header: foto profil, nama, tags, quick actions (edit, delete, add pet)
- Tabs:
  - Overview: biodata lengkap, kontak info, catatan
  - Pets: list semua hewan peliharaan dengan foto, nama, spesies, umur
  - Transactions: riwayat transaksi (appointments, products, pet hotel, grooming)
  - Loyalty: current points, tier, points history, redemption options
  - Contact history: list semua interactions (calls, WhatsApp, emails)

**Customer Search & Filter**
- Global search: search across nama, nomor HP, email
- Advanced filter:
  - Tags: multi-select (VIP, Regular, New, Blacklist)
  - Created date: date range picker
  - Has pets: yes/no
  - Loyalty tier: BRONZE, SILVER, GOLD, PLATINUM
  - Transaction count: min-max range
- Saved filters: save filter combinations untuk quick access

**Pet Management**

**Create Pet**
- Input fields:
  - Nama (required, min 2 chars, max 50 chars)
  - Spesies (required, enum: Anjing, Kucing, Kelinci, Hamster, Burung, Reptil, Lainnya)
  - Ras (required, free text, max 50 chars)
  - Tanggal lahir (optional, date picker, must be in the past)
  - Gender (required, enum: Jantan, Betina, Tidak Diketahui)
  - Berat badan awal (optional, number, min 0.1 kg, max 200 kg)
  - Foto (optional, upload image, max 2MB)
  - Microchip number (optional, 15-digit numeric)
  - Warna (optional, free text, max 50 chars)
  - Catatan khusus (optional, max 500 chars)

- Auto-generated fields:
  - Pet ID (UUID)
  - Customer ID (link ke owner)
  - Age (calculated from birth date)
  - Created at, updated at, deleted at timestamps

**Pet List View (per customer)**
- Display: card grid dengan foto, nama, spesies, ras, umur
- Sort: by nama, created date, last visit date
- Quick actions: view detail, edit, add appointment, add medical record

**Pet Detail Page**
- Header: foto, nama, spesies, ras, umur, gender, quick actions
- Tabs:
  - Overview: biodata lengkap, foto, microchip, warna, catatan
  - Vaccines: list vaksin yang sudah diberikan + due dates untuk vaksin berikutnya
  - Diseases/Allergies: list penyakit yang diderita, alergi, kondisi khusus
  - Medical Records: timeline semua medical records, sorted by date desc
  - Weight: grafik berat badan dari waktu ke waktu (line chart)
  - ID Card: generate & print ID card hewan dengan QR code

**Pet Medical History Tabs Detail**

**Overview Tab**
- Display: all pet info in structured format
- Photo gallery: upload multiple photos, set one as primary
- Quick stats: total visits, last visit date, next vaccine due date

**Vaccine Tab**
- List semua vaksin yang sudah diberikan:
  - Vaccine name (required, enum: Rabies, Distemper, Parvo, Hepatitis, Leptospirosis, dll)
  - Date given (required, date picker)
  - Batch number (optional)
  - Veterinarian (auto-fill from current user)
  - Next due date (auto-calculate based on vaccine type)
  - Notes (optional)
- Add vaccine: form untuk record new vaccine
- Vaccine schedule: auto-generate schedule berdasarkan spesies dan umur
- Reminder: notify customer via WhatsApp 7 days before due date

**Diseases/Allergies Tab**
- List semua penyakit dan alergi:
  - Type (enum: Disease, Allergy, Condition)
  - Name (required, free text)
  - Severity (enum: Mild, Moderate, Severe)
  - Diagnosed date (required)
  - Status (enum: Active, Resolved, Chronic)
  - Treatment (optional)
  - Notes (optional)
- Add disease/allergy: form untuk record new condition
- Filter: by type, status, severity

**Medical Records Tab**
- Timeline view: semua medical records sorted by date desc
- Each record card:
  - Date
  - Doctor name
  - Chief complaint (truncated)
  - Diagnosis (truncated)
  - Status (OPEN/CLOSED)
  - Click to view full detail
- Filter: by date range, doctor, status

**Weight Tab**
- Line chart: berat badan over time
- Data points: date + weight
- Add weight: quick input untuk record new weight
- Trends: show weight change percentage, alert if significant change (>10% in 1 month)

**ID Card Tab**
- Preview: ID card design dengan foto, nama, spesies, ras, QR code
- QR code: encode pet ID + customer ID untuk quick check-in
- Print: generate PDF untuk print
- Download: save as image (PNG)

**Pet Weight Tracking**
- Record weight setiap kali visit (appointment, grooming, pet hotel check-in)
- Auto-save ke pet_weight_logs table
- Grafik: line chart dengan date di X-axis, weight di Y-axis
- Alerts: notify doctor jika weight loss >10% dalam 1 bulan

#### **Business Rules**

**Customer Creation**
- Nama harus unique (case-insensitive comparison)
- Nomor HP harus unique dan valid format Indonesia
- Email harus unique jika diisi
- Tags default: ['New'] untuk customer baru
- Auto-create loyalty account dengan 0 points

**Pet Creation**
- Satu customer bisa punya multiple pets
- Pet nama tidak harus unique (bisa ada 2 pets bernama "Buddy")
- Birth date harus di masa lalu
- Weight harus positive number
- Microchip number harus 15-digit numeric jika diisi

**Customer-Pet Relationship**
- Satu pet bisa punya multiple customers (ownership bersama)
- Primary owner: customer yang create pet first
- Secondary owners: bisa di-add later
- Semua owners bisa view pet data, tapi hanya primary owner bisa edit

**Soft Delete**
- Delete customer: set deleted_at timestamp, cascade soft-delete semua pets
- Delete pet: set deleted_at timestamp, keep medical records (for audit)
- Restore: Owner bisa restore deleted customer/pet within 30 days
- After 30 days: permanent delete (hard delete)

**Data Validation**
- Frontend validation: instant feedback saat user input
- Backend validation: double-check di Edge Function sebelum save
- Database constraints: unique constraints, foreign keys, check constraints

#### **Edge Cases**

**Duplicate customer detection**
- Saat create customer: check if nama + nomor HP already exists
- Show warning: "Customer dengan nama dan nomor HP yang sama sudah ada. Lanjutkan?"
- Allow user to proceed atau merge dengan existing customer

**Pet ownership transfer**
- Transfer pet dari satu customer ke customer lain
- Keep medical history intact
- Update primary owner
- Log audit trail

**Customer merge**
- Merge 2 customer records yang duplicate
- Combine pets, transactions, loyalty points
- Keep one customer record, soft-delete the other
- Log audit trail

**Bulk operations**
- Bulk tag: apply tags ke multiple customers
- Bulk export: export customer list ke CSV/Excel
- Bulk delete: soft-delete multiple customers (require confirmation)

**Photo upload**
- Max file size: 2MB
- Allowed formats: jpg, png, webp
- Auto-resize: compress image untuk optimize storage
- Generate thumbnails: multiple sizes untuk different views

---

### **Modul 3: Appointments & Medical Records**

**Tujuan**: Manajemen jadwal kunjungan dan rekam medis elektronik hewan.

#### **Fitur Utama**

**Appointment Management**

**Create Appointment**
- Input fields:
  - Customer (required, search/select dari existing customers)
  - Pet (required, select from customer's pets)
  - Doctor (required, select from active doctors)
  - Date (required, date picker, must be today or future)
  - Time (required, time picker, 30-minute slots, 08:00-17:00)
  - Service type (required, enum: Consultation, Vaccination, Check-up, Surgery, Grooming, dll)
  - Complaint/keluhan (optional, max 500 chars)
  - Catatan khusus (optional, max 500 chars, misal: "hewan takut jarum")
  - Duration (auto-calculate based on service type, editable)
  - Is from portal (auto-fill: true jika booking dari customer portal)

- Validation rules:
  - Date must be today or future
  - Time must be within business hours (08:00-17:00)
  - Doctor must be available at selected date/time
  - No overlapping appointments untuk same doctor
  - Duration must be multiple of 30 minutes

- Auto-generated fields:
  - Appointment ID (UUID)
  - Queue number (auto-assign based on time slot)
  - Status (default: WAITING)
  - Created at, updated at timestamps

**Appointment List View**
- Display: table atau calendar view
- Table columns: queue number, time, customer name, pet name, doctor, service type, status, actions
- Calendar view: daily/weekly/monthly view, color-coded by doctor
- Filter: by date, doctor, status, service type
- Sort: by time, queue number, created date
- Search: by customer name, pet name

**Appointment Calendar View**
- Daily view: time slots dari 08:00-17:00, each slot 30 minutes
- Weekly view: 7 days, each day shows appointments
- Monthly view: calendar grid, each day shows appointment count
- Color coding: different colors untuk different doctors
- Drag & drop: reschedule appointments (change date/time)
- Click appointment: open detail modal

**Virtual Queue**
- Display: real-time queue untuk hari ini
- Columns: queue number, customer name, pet name, doctor, scheduled time, status, estimated wait time
- Status colors: WAITING (yellow), IN_PROGRESS (blue), DONE (green), CANCELLED (red)
- Auto-update: realtime subscription ke database
- Actions: call next, start appointment, mark as done, cancel

**Appointment Status Workflow**
- WAITING: appointment created, customer belum datang
- IN_PROGRESS: customer datang, dokter sedang periksa
- DONE: appointment selesai, medical record created
- CANCELLED: appointment dibatalkan (oleh customer atau staff)
- NO_SHOW: customer tidak datang tanpa cancel

**Status Transitions**
- WAITING → IN_PROGRESS: staff click "Start" atau customer check-in
- IN_PROGRESS → DONE: doctor complete examination, create medical record
- WAITING → CANCELLED: customer cancel atau staff cancel
- IN_PROGRESS → CANCELLED: rare, hanya jika emergency
- WAITING → NO_SHOW: customer tidak datang setelah 30 menit dari scheduled time

**Queue Management**
- Auto-assign queue number berdasarkan time slot
- Queue number format: Q001, Q002, Q003, dst
- Priority queue: VIP customers bisa di-prioritaskan (optional)
- Wait time estimation: calculate based on average appointment duration
- Real-time update: queue number dan status update live

**Medical Records**

**Create Medical Record**
- Input fields:
  - Appointment (auto-link ke appointment yang sedang IN_PROGRESS)
  - Pet (auto-fill dari appointment)
  - Customer (auto-fill dari pet)
  - Doctor (auto-fill dari current user)
  - Chief complaint (required, max 500 chars)
  - History (optional, max 1000 chars, riwayat penyakit, durasi, gejala)
  - Physical exam (optional, max 1000 chars, hasil pemeriksaan fisik)
  - Vital signs:
    - Berat badan (required, number, kg, min 0.1, max 200)
    - Suhu (optional, number, °C, min 30, max 45)
    - Heart rate (optional, number, bpm, min 20, max 300)
    - Respiratory rate (optional, number, breaths/min, min 5, max 100)
  - Diagnosis (required, max 500 chars)
  - Treatment (required, max 1000 chars, tindakan yang dilakukan)
  - Prescription (optional, max 1000 chars, resep obat atau anjuran)
  - Lab results (optional, max 1000 chars, hasil pemeriksaan lab)
  - Attachments (optional, upload files, max 5 files, max 5MB each)
  - Notes (optional, max 500 chars, catatan tambahan)

- Auto-generated fields:
  - Medical record ID (UUID)
  - Record number (auto-generate: MR-YYYYMMDD-XXX)
  - Status (default: OPEN)
  - Created at, updated at timestamps

- Validation rules:
  - Chief complaint dan diagnosis required
  - Berat badan required (untuk track weight over time)
  - Vital signs optional tapi recommended
  - Attachments: max 5 files, max 5MB each, allowed formats: jpg, png, pdf

**Medical Record List View**
- Display: table atau timeline view
- Table columns: record number, date, pet name, customer name, doctor, diagnosis, status, actions
- Timeline view: sorted by date desc, grouped by pet
- Filter: by date range, pet, doctor, status
- Sort: by date, record number
- Search: by record number, pet name, diagnosis

**Medical Record Detail View**
- Header: record number, date, pet name, customer name, doctor name
- Sections:
  - Chief complaint
  - History
  - Physical exam
  - Vital signs (with visual indicators for abnormal values)
  - Diagnosis
  - Treatment
  - Prescription
  - Lab results
  - Attachments (downloadable)
  - Notes
- Actions: edit (jika status OPEN), print, download PDF

**Medical Record Status**
- OPEN: medical record sedang dibuat atau belum finalized
- CLOSED: medical record sudah finalized, tidak bisa di-edit
- Status transition: OPEN → CLOSED saat appointment status DONE
- Edit restriction: hanya bisa edit jika status OPEN
- After CLOSED: bisa view tapi tidak bisa edit (audit trail)

**Medical History Timeline**
- Display: timeline semua medical records untuk satu pet
- Sorted by date desc
- Each entry: date, doctor name, diagnosis (truncated), status badge
- Click entry: open full detail modal
- Filter: by date range, doctor
- Export: download timeline sebagai PDF

#### **Business Rules**

**Appointment Creation**
- Customer harus exist dan active
- Pet harus belong ke customer yang dipilih
- Doctor harus active dan punya role DOKTER
- Date harus today atau future
- Time harus within business hours (08:00-17:00)
- No overlapping: doctor tidak bisa punya 2 appointments di waktu yang sama
- Duration default: 30 minutes untuk consultation, 60 minutes untuk surgery
- Queue number auto-assign berdasarkan time slot

**Appointment Rescheduling**
- Only if status WAITING
- Check doctor availability di new date/time
- Update queue number jika perlu
- Notify customer via WhatsApp tentang perubahan
- Log audit trail

**Appointment Cancellation**
- Can cancel if status WAITING atau IN_PROGRESS
- Require reason (optional tapi recommended)
- Update status ke CANCELLED
- Notify customer via WhatsApp
- Log audit trail

**Medical Record Creation**
- Harus linked ke appointment yang IN_PROGRESS
- Auto-fill data dari appointment (pet, customer, doctor)
- Doctor harus be the one who handle appointment
- Berat badan auto-save ke pet_weight_logs
- Status default: OPEN

**Medical Record Finalization**
- Status berubah ke CLOSED saat appointment status DONE
- Setelah CLOSED, tidak bisa edit
- Edit hanya bisa jika status OPEN
- Semua edits logged di audit trail

**Attachment Management**
- Max 5 files per medical record
- Max 5MB per file
- Allowed formats: jpg, png, pdf
- Auto-generate thumbnails untuk images
- Store di Supabase Storage
- Access control: only doctor and customer can view

#### **Edge Cases**

**Overlapping appointments**
- System check doctor availability sebelum create appointment
- If overlap: show error "Dokter sudah punya appointment di waktu tersebut"
- Suggest alternative time slots

**No-show handling**
- If customer tidak datang within 30 menit dari scheduled time
- Staff bisa mark as NO_SHOW
- Status berubah ke NO_SHOW
- Notify customer via WhatsApp
- Log audit trail

**Emergency appointment**
- Walk-in customer tanpa booking
- Staff create appointment dengan time = now
- Priority queue (optional)
- Queue number auto-assign

**Medical record edit after CLOSED**
- Tidak bisa edit jika status CLOSED
- Workaround: create new medical record dengan reference ke old record
- Or: Owner bisa reopen medical record (change status back to OPEN)
- Log audit trail untuk reopen action

**Attachment upload failure**
- If upload gagal (network error, file too large)
- Show error message
- Allow retry
- Keep form data intact

**Concurrent medical record creation**
- Two doctors try to create medical record untuk same appointment
- First one wins (database constraint)
- Second one get error "Medical record already exists for this appointment"

---

### **Modul 4: Pet Hotel & Grooming**

**Tujuan**: Manajemen booking dan check-in/out pet hotel & grooming salon.

#### **Fitur Utama**

**Pet Hotel**

**Room Management**
- Room master data:
  - Room name/number (required, unique)
  - Type (required, enum: Suite, Standard, Budget)
  - Capacity (required, number, max pets per room)
  - Price per night (required, number, min 0)
  - Description (optional, max 500 chars)
  - Photo (optional, upload image)
  - Status (required, enum: AVAILABLE, RESERVED, OCCUPIED, MAINTENANCE, INACTIVE)
  - Cleanliness status (enum: CLEAN, DIRTY, UNDER_CLEANING)

- Room list view:
  - Display: card grid atau table
  - Card view: photo, name, type, status badge, current occupant (if any)
  - Table view: name, type, capacity, price, status, cleanliness, actions
  - Filter: by type, status, cleanliness
  - Actions: edit, change status, view bookings

**Room Status Workflow**
- AVAILABLE: room kosong, siap untuk booking
- RESERVED: room sudah di-book untuk tanggal tertentu
- OCCUPIED: room sedang ditempati pet
- MAINTENANCE: room sedang diperbaiki
- INACTIVE: room tidak digunakan (nonaktif)

**Cleanliness Status Workflow**
- CLEAN: room sudah dibersihkan, siap untuk occupant baru
- DIRTY: room perlu dibersihkan (setelah check-out)
- UNDER_CLEANING: room sedang dalam proses pembersihan
- Transition: OCCUPIED → check-out → DIRTY → UNDER_CLEANING → CLEAN → AVAILABLE

**Booking System**
- Input fields:
  - Customer (required, search/select)
  - Pet (required, select from customer's pets)
  - Room (optional, auto-assign based on availability atau manual select)
  - Check-in date (required, date picker, must be today or future)
  - Check-out date (required, date picker, must be after check-in)
  - Special notes (optional, max 500 chars, makanan, obat, behavior)
  - Price (auto-calculate: nights × price_per_night)

- Validation rules:
  - Check-in date must be today or future
  - Check-out date must be after check-in date
  - Room must be AVAILABLE atau RESERVED untuk tanggal tersebut
  - No overlapping bookings untuk same room
  - Pet must belong to customer

- Auto-generated fields:
  - Booking ID (UUID)
  - Booking number (auto-generate: PH-YYYYMMDD-XXX)
  - Status (default: BOOKED)
  - Total price (auto-calculate)
  - Created at, updated at timestamps

**Booking List View**
- Display: table atau calendar view
- Table columns: booking number, customer, pet, room, check-in, check-out, status, total price, actions
- Calendar view: timeline view, each row = room, each column = date
- Filter: by date range, room, status, customer
- Sort: by check-in date, booking number
- Search: by booking number, customer name, pet name

**Booking Status Workflow**
- BOOKED: booking created, belum check-in
- CHECKED_IN: pet sudah check-in, room OCCUPIED
- CHECKED_OUT: pet sudah check-out, room AVAILABLE (setelah cleaning)
- CANCELLED: booking dibatalkan

**Status Transitions**
- BOOKED → CHECKED_IN: staff check-in pet pada hari check-in
- CHECKED_IN → CHECKED_OUT: staff check-out pet pada hari check-out
- BOOKED → CANCELLED: customer cancel atau staff cancel
- CHECKED_IN → CANCELLED: rare, hanya jika emergency

**Check-in Process**
- Staff select booking yang status BOOKED
- Verify pet identity (scan QR code atau input pet ID)
- Record actual_check_in_at timestamp
- Update booking status ke CHECKED_IN
- Update room status ke OCCUPIED
- Input initial condition (optional, max 500 chars)
- Input customer instructions (optional, max 500 chars)
- Take photo of pet (optional)
- Notify customer via WhatsApp tentang check-in success

**Check-out Process**
- Staff select booking yang status CHECKED_IN
- Verify pet identity
- Record actual_check_out_at timestamp
- Input final condition (optional, max 500 chars)
- Calculate final price (jika ada additional charges)
- Update booking status ke CHECKED_OUT
- Update room status ke DIRTY
- Generate invoice
- Process payment
- Notify customer via WhatsApp tentang check-out success

**Daily Care Logs**
- Staff catat aktivitas daily untuk setiap pet yang stay:
  - Log type (enum: Feeding, Medicine, Walk, Play, Observation, Other)
  - Time (required, time picker)
  - Description (required, max 500 chars)
  - Photo (optional, upload image)
  - Staff (auto-fill dari current user)

- Log list view:
  - Display: timeline view, sorted by time desc
  - Each entry: time, type badge, description, photo thumbnail, staff name
  - Filter: by type, date
  - Actions: edit, delete (only within 1 hour of creation)

- Customer view:
  - Customer bisa view all care logs di portal
  - Real-time update: logs muncul otomatis setelah staff input
  - Photo gallery: view all photos dari stay

**Grooming**

**Grooming Service Management**
- Service master data:
  - Service name (required, unique)
  - Description (optional, max 500 chars)
  - Duration (required, number, minutes)
  - Price (required, number, min 0)
  - Category (required, enum: Bath, Haircut, Nail Trim, Ear Clean, Teeth Clean, Full Grooming, Other)
  - Photo (optional, upload image)
  - Status (required, enum: ACTIVE, INACTIVE)

- Service list view:
  - Display: table
  - Columns: name, category, duration, price, status, actions
  - Filter: by category, status
  - Actions: edit, toggle status

**Grooming Booking**
- Input fields:
  - Customer (required, search/select)
  - Pet (required, select from customer's pets)
  - Service (required, select from active services)
  - Groomer (optional, select from staff dengan role KASIR atau specific groomer role)
  - Date (required, date picker, must be today or future)
  - Time (required, time picker, 30-minute slots)
  - Special notes (optional, max 500 chars)
  - Price (auto-fill from service)

- Validation rules:
  - Date must be today or future
  - Time must be within business hours
  - Groomer must be available at selected date/time
  - No overlapping bookings untuk same groomer
  - Duration auto-fill from service

- Auto-generated fields:
  - Booking ID (UUID)
  - Booking number (auto-generate: GR-YYYYMMDD-XXX)
  - Status (default: BOOKED)
  - Created at, updated at timestamps

**Grooming Booking List View**
- Display: table atau calendar view
- Table columns: booking number, customer, pet, service, groomer, date, time, status, actions
- Calendar view: daily view, time slots, color-coded by groomer
- Filter: by date, groomer, status, service
- Sort: by date, time
- Search: by booking number, customer name, pet name

**Grooming Status Workflow**
- BOOKED: booking created, belum mulai
- IN_PROGRESS: grooming sedang berlangsung
- DONE: grooming selesai
- CANCELLED: booking dibatalkan

**Status Transitions**
- BOOKED → IN_PROGRESS: groomer start grooming
- IN_PROGRESS → DONE: groomer finish grooming
- BOOKED → CANCELLED: customer cancel atau staff cancel
- IN_PROGRESS → CANCELLED: rare, hanya jika emergency

**Grooming Queue**
- Display: real-time queue untuk hari ini
- Columns: queue number, customer name, pet name, service, groomer, scheduled time, status
- Status colors: BOOKED (yellow), IN_PROGRESS (blue), DONE (green), CANCELLED (red)
- Auto-update: realtime subscription
- Actions: start grooming, mark as done, cancel

**Before/After Photos**
- Groomer take photo before grooming
- Groomer take photo after grooming
- Upload ke grooming booking
- Customer bisa view di portal
- Optional: share ke social media (future feature)

#### **Business Rules**

**Room Management**
- Room name harus unique
- Price per night harus positive number
- Capacity minimal 1
- Status default: AVAILABLE
- Cleanliness default: CLEAN

**Pet Hotel Booking**
- Customer harus exist dan active
- Pet harus belong ke customer
- Check-in date harus today atau future
- Check-out date harus setelah check-in date
- Room harus AVAILABLE untuk tanggal tersebut
- Auto-calculate total price: nights × price_per_night
- Nights calculation: (check-out date - check-in date) in days

**Check-in/Check-out**
- Check-in hanya bisa jika status BOOKED
- Check-out hanya bisa jika status CHECKED_IN
- Actual check-in/out timestamp recorded
- Room status auto-update
- Invoice auto-generate saat check-out

**Daily Care Logs**
- Hanya bisa create log jika booking status CHECKED_IN
- Log harus linked ke booking
- Photo optional tapi recommended
- Staff auto-fill dari current user
- Edit hanya bisa within 1 hour of creation

**Grooming Service**
- Service name harus unique
- Duration harus positive number (minutes)
- Price harus positive number
- Status default: ACTIVE

**Grooming Booking**
- Customer harus exist dan active
- Pet harus belong ke customer
- Service harus ACTIVE
- Date harus today atau future
- Time harus within business hours
- Groomer harus available (no overlapping)
- Price auto-fill from service

#### **Edge Cases**

**Room double booking**
- System check room availability sebelum create booking
- If overlap: show error "Room sudah di-book untuk tanggal tersebut"
- Suggest alternative rooms atau dates

**Early check-in**
- Customer datang sebelum check-in date
- Staff bisa allow early check-in jika room AVAILABLE
- Update actual_check_in_at ke actual time
- Adjust price jika perlu (optional)

**Late check-out**
- Customer tidak ambil pet pada check-out date
- Staff contact customer
- Extend booking jika customer setuju
- Update check-out date dan recalculate price
- Generate new invoice

**Pet sickness during stay**
- Pet sakit selama stay di pet hotel
- Staff notify customer immediately via WhatsApp
- Suggest bring pet to clinic (create appointment)
- Log incident di daily care logs
- Take photo untuk documentation

**Grooming no-show**
- Customer tidak datang untuk grooming appointment
- Staff mark as NO_SHOW setelah 30 menit
- Status berubah ke CANCELLED
- Notify customer via WhatsApp

**Groomer unavailability**
- Groomer sakit atau tidak masuk
- Reassign bookings ke groomer lain
- Notify customer tentang perubahan groomer
- Allow customer reschedule atau cancel

**Photo upload failure**
- If upload gagal (network error, file too large)
- Show error message
- Allow retry
- Keep form data intact

---

### **Modul 5: Petshop & Inventory**

**Tujuan**: Manajemen produk dan stok barang (makanan, obat, aksesoris, alat, dll).

#### **Fitur Utama**

**Product Master Data**

**Create Product**
- Input fields:
  - Nama produk (required, min 2 chars, max 100 chars)
  - SKU (required, unique, alphanumeric, max 50 chars)
  - Barcode (optional, unique, numeric, max 50 chars)
  - Kategori (required, enum: Makanan, Obat, Aksesoris, Alat, Mainan, Kandang, Lainnya)
  - Sub-kategori (optional, free text, max 50 chars)
  - Supplier (optional, select from suppliers)
  - Purchase price (required, number, min 0)
  - Selling price (required, number, min purchase price)
  - Unit (required, enum: Piece, Kilogram, Liter, Box, Pack)
  - Stock quantity (required, number, min 0)
  - Min stock level (required, number, min 0, trigger untuk purchase order)
  - Max stock level (optional, number, min min stock level)
  - Stock location (optional, free text, max 100 chars, tempat fisik di toko)
  - Expiry date (optional, date picker, must be future)
  - Description (optional, max 1000 chars)
  - Photo (optional, upload image)
  - Status (required, enum: ACTIVE, ARCHIVED)

- Validation rules:
  - Nama: required, unique (case-insensitive)
  - SKU: required, unique, alphanumeric only
  - Barcode: optional, unique jika diisi
  - Selling price harus >= purchase price
  - Min stock level harus >= 0
  - Max stock level harus >= min stock level jika diisi
  - Expiry date harus future jika diisi

- Auto-generated fields:
  - Product ID (UUID)
  - Created at, updated at, deleted at timestamps

**Product List View**
- Display: table atau card grid
- Table columns: photo, nama, SKU, kategori, stock, purchase price, selling price, status, actions
- Card view: photo, nama, SKU, stock badge, selling price
- Filter: by kategori, status, stock level (low stock, out of stock)
- Sort: by nama, SKU, stock, selling price, created date
- Search: by nama, SKU, barcode
- Pagination: 50 items per page

**Low Stock Alert**
- Auto-detect products dengan current stock <= min stock level
- Display badge "Low Stock" di product list
- Notification: alert staff di dashboard
- Suggestion: create purchase order untuk restock

**Out of Stock Alert**
- Auto-detect products dengan current stock = 0
- Display badge "Out of Stock" di product list
- Notification: alert staff di dashboard
- Hide dari POS jika out of stock (optional)

**Product Detail Page**
- Header: photo, nama, SKU, barcode, status badge
- Sections:
  - Basic info: kategori, sub-kategori, supplier, unit, description
  - Pricing: purchase price, selling price, margin
  - Stock: current stock, min level, max level, location
  - Expiry: expiry date, days until expiry
  - Stock movements: history semua stock movements
  - Sales history: list invoices yang include this product

**Stock Management**

**Stock Levels**
- Current stock: jumlah stock saat ini
- Min stock level: threshold untuk low stock alert
- Max stock level: maximum stock yang disarankan
- Stock location: tempat fisik di toko (rak, gudang, dll)

**Stock Movements**
- Movement types:
  - IN: pembelian dari supplier atau transfer masuk
  - OUT: penjualan via POS
  - RETURN: produk return dari customer
  - ADJUSTMENT: penyesuaian karena count error
  - DAMAGED: item rusak
  - EXPIRED: kadaluarsa
  - OPNAME: hasil stock opname fisik

- Movement record:
  - Movement ID (UUID)
  - Product ID (required)
  - Type (required, enum: IN, OUT, RETURN, ADJUSTMENT, DAMAGED, EXPIRED, OPNAME)
  - Quantity (required, number, positive untuk IN/RETURN, negative untuk OUT/DAMAGED/EXPIRED)
  - Reason (required, max 500 chars)
  - Reference ID (optional, link ke PO, invoice, dll)
  - Recorded by (auto-fill dari current user)
  - Recorded at (auto-fill timestamp)

- Movement list view:
  - Display: table
  - Columns: date, product, type, quantity, reason, recorded by, reference
  - Filter: by type, date range, product
  - Sort: by date desc
  - Export: download ke CSV/Excel

**Stock History**
- Timeline semua stock movements untuk satu product
- Sorted by date desc
- Each entry: date, type badge, quantity (+/-), reason, recorded by
- Running balance: show stock balance setelah each movement
- Export: download history ke CSV/Excel

**Purchase Order (PO)**

**Create PO**
- Input fields:
  - Supplier (required, select from suppliers)
  - Items (required, array of products):
    - Product (required, select from products)
    - Quantity (required, number, min 1)
    - Purchase price (auto-fill from product, editable)
    - Subtotal (auto-calculate: quantity × price)
  - Notes (optional, max 500 chars)
  - Expected delivery date (optional, date picker)

- Auto-generated fields:
  - PO ID (UUID)
  - PO number (auto-generate: PO-YYYYMMDD-XXX)
  - Total amount (auto-calculate: sum of subtotals)
  - Status (default: DRAFT)
  - Created at, updated at timestamps

**PO List View**
- Display: table
- Columns: PO number, supplier, total amount, status, expected delivery, created date, actions
- Filter: by status, supplier, date range
- Sort: by created date desc, PO number
- Search: by PO number, supplier name

**PO Status Workflow**
- DRAFT: PO dibuat, belum dikirim ke supplier
- SENT: PO sudah dikirim ke supplier
- PARTIAL_RECEIVED: sebagian barang sudah diterima
- RECEIVED: semua barang sudah diterima
- CANCELLED: PO dibatalkan

**Status Transitions**
- DRAFT → SENT: staff send PO ke supplier (via email/WhatsApp)
- SENT → PARTIAL_RECEIVED: sebagian barang arrive
- PARTIAL_RECEIVED → RECEIVED: semua barang arrive
- SENT → RECEIVED: semua barang arrive sekaligus
- DRAFT → CANCELLED: staff cancel PO
- SENT → CANCELLED: staff cancel PO (notify supplier)

**Goods Received Note (GRN)**
- Process saat barang arrive dari supplier:
  - Select PO yang status SENT atau PARTIAL_RECEIVED
  - Verify items received:
    - Product (auto-fill from PO)
    - Quantity ordered (auto-fill from PO)
    - Quantity received (input actual quantity)
    - Discrepancy (auto-calculate: received - ordered)
  - Record discrepancies (lebih/kurang/rusak)
  - Update PO status:
    - If all items received → RECEIVED
    - If partial → PARTIAL_RECEIVED
  - Create stock movements (type: IN) untuk each item received
  - Update current stock untuk each product
  - Record GRN timestamp

**Supplier Management**
- Supplier master data:
  - Nama (required, unique)
  - Contact person (optional)
  - Phone (optional)
  - Email (optional)
  - Address (optional)
  - Notes (optional)

- Supplier list view:
  - Display: table
  - Columns: nama, contact person, phone, email, total POs, actions
  - Search: by nama, contact person
  - Actions: edit, view POs

#### **Business Rules**

**Product Creation**
- Nama harus unique (case-insensitive)
- SKU harus unique dan alphanumeric only
- Barcode harus unique jika diisi
- Selling price harus >= purchase price
- Min stock level harus >= 0
- Max stock level harus >= min stock level jika diisi
- Stock quantity default: 0
- Status default: ACTIVE

**Stock Management**
- Current stock tidak bisa negative
- Stock movement harus record reason
- Stock movement auto-update current stock
- IN/RETURN: add to current stock
- OUT/DAMAGED/EXPIRED: subtract from current stock
- ADJUSTMENT: bisa positive atau negative
- OPNAME: set current stock ke actual count

**Purchase Order**
- PO harus punya minimal 1 item
- Total amount auto-calculate dari items
- Status default: DRAFT
- PO number auto-generate dengan format PO-YYYYMMDD-XXX

**Goods Received**
- Quantity received harus >= 0
- Discrepancy auto-calculate
- If received < ordered: record discrepancy, keep PO status PARTIAL_RECEIVED
- If received = ordered: update PO status RECEIVED
- If received > ordered: record discrepancy, update PO status RECEIVED
- Stock auto-update setelah GRN

**Expiry Tracking**
- Expiry date harus future jika diisi
- Alert: notify staff 30 days before expiry
- Alert: notify staff 7 days before expiry
- Alert: notify staff on expiry date
- Auto-mark as EXPIRED setelah expiry date (optional, require manual confirmation)

#### **Edge Cases**

**Duplicate product detection**
- Saat create product: check if nama or SKU already exists
- Show warning: "Produk dengan nama/SKU yang sama sudah ada. Lanjutkan?"
- Allow user to proceed atau cancel

**Negative stock**
- System prevent stock from going negative
- If POS transaction would cause negative stock: show error "Stok tidak cukup"
- Workaround: allow negative stock dengan warning (optional, require Owner approval)

**Stock adjustment**
- Staff adjust stock karena count error
- Require reason (mandatory)
- Require Owner approval jika adjustment > 10% of current stock
- Log audit trail

**PO partial receive**
- Supplier kirim sebagian barang
- Staff record received items
- PO status → PARTIAL_RECEIVED
- Staff bisa receive sisa barang later
- Each receive create stock movement

**Expiry date passed**
- Product sudah expired tapi masih ada stock
- Alert: notify staff
- Staff bisa mark as EXPIRED (create stock movement type EXPIRED)
- Stock auto-update
- Product masih bisa di-view tapi tidak bisa di-sell

**Barcode scan failure**
- Barcode tidak ada di database
- Show error "Produk tidak ditemukan"
- Allow manual input
- Suggest create new product

**Supplier inactive**
- Supplier sudah tidak aktif tapi ada PO lama
- Keep PO history intact
- Prevent create new PO dengan supplier inactive
- Show warning di PO list

---

### **Modul 6: Point of Sale (POS) & Billing**

**Tujuan**: Transaksi penjualan real-time untuk produk, services, pet hotel, dan grooming.

#### **Fitur Utama**

**POS Dashboard**

**Layout**
- Three-column layout:
  - Left (50%): product grid + search
  - Middle (25%): cart with items
  - Right (25%): checkout panel

**Product Selection**
- Search bar: search by nama, SKU, atau barcode
- Barcode scanner: integrate dengan USB barcode scanner (auto-focus)
- Product grid: display products dengan photo, nama, price, stock
- Category filter: filter by kategori (Makanan, Obat, Aksesoris, dll)
- Quick add: click product untuk add ke cart
- Stock indicator: show stock quantity, highlight if low/out of stock
- Out of stock: hide atau disable products yang out of stock

**Cart Management**
- Display: list items di cart
- Each item:
  - Nama produk
  - Price per unit
  - Quantity picker (+/- buttons, manual input)
  - Subtotal (price × quantity)
  - Remove button
- Cart summary:
  - Subtotal: sum of all item subtotals
  - Discount: total discount applied
  - Tax: PPN 11% (configurable)
  - Total: subtotal - discount + tax
- Actions:
  - Clear cart: remove all items
  - Hold cart: save cart untuk later (multi-customer handling)
  - Apply voucher/promo code

**Checkout Process**
- Customer selection:
  - Search existing customer atau create new guest
  - Auto-fill customer info jika existing
  - Link to pet jika applicable (pet hotel checkout, grooming)

- Service selection (jika applicable):
  - Link completed appointment (medical fee)
  - Link checked-out pet hotel booking
  - Link completed grooming booking
  - Auto-sum dengan products di cart

- Payment method:
  - CASH: input amount → calculate change
  - QRIS: generate QR code → customer scan → verify payment
  - TRANSFER: show bank account + reference code → manual confirmation
  - E_WALLET: direct payment link
  - CREDIT_CARD: card reader integration (future)
  - MIXED: multiple payment methods

- Invoice generation:
  - Auto-generate invoice number (format: INV-YYYYMMDD-XXX)
  - Print receipt (kasir + customer copy)
  - Email invoice ke customer (jika ada email)
  - WhatsApp invoice ke customer (jika ada nomor HP)

**Invoice Management**

**Invoice Record**
- Invoice number (unique, auto-generated)
- Customer name, pet name (jika applicable)
- Invoice type: POS, CLINICAL, PET_HOTEL, GROOMING, MIXED
- Items: product/service name, quantity, price, subtotal
- Summary: subtotal, discount, tax, total
- Payment method + status
- Timestamps: created, paid
- Staff who processed

**Invoice Status**
- UNPAID: belum dibayar
- PARTIAL_PAYMENT: bayar sebagian
- PAID: lunas
- CANCELLED: dibatalkan (ada audit trail)

**Status Transitions**
- UNPAID → PARTIAL_PAYMENT: customer bayar sebagian
- PARTIAL_PAYMENT → PAID: customer bayar sisa
- UNPAID → PAID: customer bayar lunas
- UNPAID/PARTIAL → CANCELLED: staff cancel invoice

**Refund/Return**
- Staff process return dengan reason
- Partial atau full refund
- Refund status tracked
- Stock auto-update saat return di-process
- Generate refund receipt

**Cash Shift Management**

**Opening Shift**
- Opening cash (uang awal kasir)
- Recorded at timestamp
- Staff who opened

**Closing Shift**
- Count actual cash
- Sum dari POS transactions
- Variance (actual - expected)
- Notes jika ada discrepancy
- Status: PENDING APPROVAL → APPROVED / REJECTED
- Approval by: Owner/Admin

#### **Business Rules**

**POS Transaction**
- Customer harus exist (atau create guest)
- Cart harus punya minimal 1 item
- Stock harus cukup untuk semua items
- Payment method harus dipilih
- Invoice auto-generate setelah payment success
- Stock auto-deduct setelah invoice created
- Loyalty points auto-earn setelah transaction

**Payment Processing**
- CASH: input amount >= total → calculate change
- QRIS: generate QR → wait for payment confirmation (max 15 menit)
- TRANSFER: show bank info → staff manual confirm setelah customer transfer
- E_WALLET: redirect ke payment gateway → wait for callback
- MIXED: split total ke multiple payment methods

**Invoice Generation**
- Invoice number unique per day
- Format: INV-YYYYMMDD-XXX (XXX = sequential number)
- Include all items, discounts, taxes
- Print receipt otomatis setelah payment
- Email/WhatsApp invoice jika customer data available

**Stock Update**
- Auto-deduct stock setelah invoice created
- If stock insufficient: prevent transaction
- Record stock movement (type: OUT)
- Link stock movement ke invoice

**Loyalty Points**
- Auto-calculate points earned (1 point per 1000 rupiah)
- Update customer loyalty balance
- Record loyalty transaction (type: EARN)
- Notify customer via WhatsApp

#### **Edge Cases**

**Stock insufficient during checkout**
- System check stock sebelum process payment
- If insufficient: show error "Stok tidak cukup untuk [product name]"
- Allow customer remove item atau reduce quantity
- Prevent transaction jika stock masih insufficient

**Payment timeout (QRIS/Transfer)**
- QRIS: timeout setelah 15 menit
- Transfer: manual confirmation required
- If timeout: mark invoice as CANCELLED
- Allow retry dengan new payment

**Partial payment**
- Customer bayar sebagian
- Mark invoice as PARTIAL_PAYMENT
- Record payment amount
- Remaining balance tracked
- Customer bisa bayar sisa later

**Invoice cancellation after payment**
- Only Owner bisa cancel paid invoice
- Require reason (mandatory)
- Process refund jika applicable
- Restore stock jika applicable
- Log audit trail

**Cash shift discrepancy**
- Actual cash != expected cash
- Show variance amount
- Require notes dari kasir
- Owner review dan approve/reject
- Log audit trail

**Concurrent POS transactions**
- Two kasir process transactions bersamaan
- Stock check real-time
- First transaction wins jika stock limited
- Second transaction get error jika stock insufficient

**Barcode scan error**
- Barcode tidak recognized
- Show error "Produk tidak ditemukan"
- Allow manual search
- Suggest similar products

---

### **Modul 7: Loyalty & Promotions**

**Tujuan**: Engagement pelanggan dan repeat business.

#### **Fitur Utama**

**Loyalty Program**

**Membership Tiers**
- BRONZE: 0-100 points
- SILVER: 101-500 points
- GOLD: 501-2000 points
- PLATINUM: 2000+ points
- Auto-tier upgrade based on points accumulated
- Tier benefits: discount %, free services, early access

**Points Earning**
- Automatic per rupiah spent: 1 point per 1000 rupiah (configurable)
- Bonus points untuk produk tertentu (configurable per product)
- Birthday points: special offer di bulan ulang tahun
- Referral points: refer teman yang register dan transact

**Points Redemption**
- Tukar points dengan: discount, free product, free service
- Catalog of redemption options (owner set)
- Transaction type: EARN, REDEEM, EXPIRE, ADJUST
- Points expire setelah 1 tahun tidak ada aktivitas
- Notify customer sebelum points expire

**Tier Benefits**
- Discount percentage per tier (BRONZE: 0%, SILVER: 5%, GOLD: 10%, PLATINUM: 15%)
- Special birthday offer (free grooming untuk GOLD+)
- Early access ke promo (SILVER+)
- Free services per bulan (GOLD: 1x, PLATINUM: 2x)

**Promotions**

**Promotion Types**
- PERCENTAGE: discount X% (misal: 20% off)
- FIXED: discount fix amount (misal: Rp 50.000 off)
- BUNDLE: buy X qty get Y qty free (misal: buy 3 get 1 free)
- HAPPY_HOUR: special price pada jam tertentu
- BIRTHDAY: special offer untuk customer yang ulang tahun

**Promotion Rules**
- Applicable to: products, services, atau both
- Valid date range (start date - end date)
- Max usage per customer (optional)
- Min purchase amount (optional)
- Applicable to: all customers atau specific tier / tags
- Stackable: bisa digabung dengan promo lain atau tidak

**Promotion Management**
- Create/edit/deactivate promotions
- Track redemption count
- A/B test different promo (hidden feature)
- Auto-expire setelah end date

**Customer Feedback**

**Feedback Collection**
- Post-transaction: rate experience 1-5 stars
- Comment field (optional)
- Photo upload (misal: grooming result photos)
- Link to order/appointment

**Feedback Display**
- Portal customer: bisa lihat feedback yang pernah dibuat
- Staff dashboard: bisa lihat semua feedback (untuk improve service)
- Owner: bisa track NPS (Net Promoter Score)

#### **Business Rules**

**Points Calculation**
- 1 point per 1000 rupiah spent (configurable)
- Round down (1500 rupiah = 1 point, 2500 rupiah = 2 points)
- Exclude tax dan discount dari calculation
- Bonus points added on top

**Tier Upgrade**
- Auto-upgrade saat points reach threshold
- Notify customer via WhatsApp
- Update tier di customer profile
- Apply new benefits immediately

**Points Expiry**
- Points expire setelah 1 tahun tidak ada aktivitas
- Activity: earn atau redeem points
- Notify customer 30 days before expiry
- Notify customer 7 days before expiry
- Auto-expire on expiry date

**Promotion Application**
- Check promotion validity (date range, usage limit)
- Check customer eligibility (tier, tags)
- Check min purchase amount
- Apply discount ke cart
- Record promotion redemption

**Feedback Collection**
- Invite customer setelah transaction completed
- Reminder setelah 24 jam jika belum fill
- Max 1 feedback per transaction
- Anonymous feedback allowed

#### **Edge Cases**

**Points calculation with discount**
- Customer apply discount voucher
- Points calculated dari final amount (after discount)
- Exclude discount value dari points calculation

**Tier downgrade**
- Points expire → balance drop below tier threshold
- Auto-downgrade ke lower tier
- Notify customer tentang tier change
- Update benefits

**Promotion stack**
- Jika customer apply multiple promotions: system check stackability rules
- Default: tidak bisa stack (hanya 1 promo terbaik yang berlaku)
- Owner bisa set specific promotions sebagai stackable
- Priority order: BIRTHDAY > BUNDLE > PERCENTAGE > FIXED > HAPPY_HOUR
- Display breakdown: tampilkan masing-masing discount di invoice

**Birthday promotion**
- Auto-detect customer dengan ulang tahun bulan ini
- Auto-apply birthday discount saat checkout
- Notify customer via WhatsApp tentang birthday offer
- Valid hanya di bulan ulang tahun
- Max 1x usage per customer per tahun

**Happy hour promotion**
- Berlaku pada jam tertentu (misal: 14:00-16:00)
- Auto-apply jika transaksi pada jam tersebut
- Display countdown timer di POS dashboard
- Staff bisa override (manual apply) jika sistem tidak auto-detect

**Bundle promotion**
- Buy X qty get Y qty free
- System auto-detect eligible products di cart
- Auto-add free items ke cart
- Display "Free!" badge di cart items
- Track redemption count per customer

**Promotion expiry**
- Auto-deactivate setelah end date
- Notify staff 7 days before expiry
- Show warning di POS jika promo akan expired
- Keep redemption history untuk reporting

**Customer feedback abuse**
- Max 1 feedback per transaction
- Prevent duplicate feedback (check transaction_id)
- Allow edit feedback within 24 jam
- After 24 jam: read-only
- Flag abusive feedback (profanity filter, optional)

---

### **Modul 8: Financial Management & Expenses**

**Tujuan**: Track pengeluaran operasional dan manajemen keuangan bisnis.

#### **Fitur Utama**

**Expense Tracking**

**Expense Categories**
- Utility: listrik, air, internet, telepon
- Maintenance: perbaikan fasilitas, perawatan gedung, AC service
- Supplies: kebutuhan operasional (kertas, tinta, alat kebersihan)
- Salary: gaji karyawan, bonus, tunjangan
- Marketing: iklan, promosi, social media, event
- Rent: sewa tempat
- Insurance: asuransi bisnis, asuransi hewan
- Professional fees: konsultan, akuntan, legal
- Other: pengeluaran lain-lain

**Create Expense**
- Input fields:
  - Category (required, select dari categories)
  - Amount (required, number, min 1, max 1.000.000.000)
  - Description (required, max 500 chars)
  - Expense date (required, date picker, default today)
  - Receipt attachment (optional, upload image/PDF, max 5MB)
  - Paid by (required, select dari staff)
  - Payment method (required, enum: CASH, TRANSFER, E_WALLET, CREDIT_CARD)
  - Reference number (optional, untuk transfer/e-wallet)
  - Notes (optional, max 500 chars)

- Validation rules:
  - Amount harus positive number
  - Description required, minimal 10 chars
  - Expense date tidak bisa future (kecuali Owner override)
  - Receipt optional tapi recommended untuk amount > 500.000

- Auto-generated fields:
  - Expense ID (UUID)
  - Expense number (auto-generate: EXP-YYYYMMDD-XXX)
  - Status (default: PENDING)
  - Created at, updated at timestamps
  - Created by (auto-fill dari current user)

**Expense List View**
- Display: table
- Columns: expense number, date, category, description, amount, paid by, status, actions
- Filter: by category, status, date range, paid by
- Sort: by date desc, amount, category
- Search: by expense number, description
- Pagination: 50 items per page
- Summary cards: total pending, total approved, total rejected (di atas table)

**Expense Detail View**
- Header: expense number, status badge, date
- Sections:
  - Basic info: category, amount, description, expense date
  - Payment info: paid by, payment method, reference number
  - Attachment: receipt preview/download
  - Approval info: approved by, approved at, notes
  - Audit trail: semua perubahan status

**Expense Approval Workflow**
- Status workflow:
  - PENDING: expense baru dibuat, menunggu approval
  - APPROVED: expense disetujui Owner/Admin
  - REJECTED: expense ditolak, perlu revisi

- Approval rules:
  - Amount < 1.000.000: Admin bisa approve
  - Amount >= 1.000.000: hanya Owner bisa approve
  - Expense yang dibuat oleh Owner: auto-approved
  - Expense yang dibuat oleh Admin: perlu Owner approval

- Approval actions:
  - Approve: set status APPROVED, record approver, timestamp
  - Reject: set status REJECTED, require rejection reason, notify creator
  - Request revision: set status REJECTED dengan notes untuk revisi

- Notification:
  - Notify Owner/Admin saat ada expense baru (PENDING)
  - Notify creator saat expense approved/rejected
  - Notify via in-app notification + WhatsApp (optional)

**Financial Reports**

**Daily Report**
- Total revenue dari POS (products sales)
- Total service fees (appointments, pet hotel, grooming)
- Total loyalty points redeemed (discount value)
- Total cash collected
- Total non-cash payments (QRIS, transfer, e-wallet)
- Discrepancies (cash shift variance)
- Top 5 products sold hari ini
- Top 5 services hari ini
- Export: PDF, Excel

**Monthly Report**
- Revenue by category:
  - Products sales
  - Clinical services (appointments)
  - Pet hotel revenue
  - Grooming revenue
  - Other revenue
- Revenue by service type:
  - Consultation
  - Vaccination
  - Surgery
  - Grooming (per service)
  - Pet hotel (per room type)
- Top 10 customers by spending
- Top 10 products by quantity sold
- Top 10 products by revenue
- Total expenses by category
- Profit margin (revenue - expenses)
- Monthly trend (line chart: revenue vs expenses)
- Year-over-year comparison
- Export: PDF, Excel

**Expense Report**
- Total expense by category (pie chart)
- Breakdown per category (bar chart)
- Trend over time (line chart)
- Top expense items
- Expense vs budget (jika budget di-set)
- Export: PDF, Excel

**Cash Flow Report**
- Cash in: semua pemasukan (POS, services, dll)
- Cash out: semua pengeluaran (expenses, PO payments)
- Net cash flow: cash in - cash out
- Daily cash flow trend
- Monthly cash flow trend
- Export: PDF, Excel

**Profit & Loss Report**
- Total revenue
- Total COGS (Cost of Goods Sold)
- Gross profit
- Total expenses
- Net profit
- Profit margin percentage
- Breakdown by month
- Export: PDF, Excel

#### **Business Rules**

**Expense Creation**
- Creator harus active staff (OWNER, ADMIN, DOKTER, KASIR)
- Category harus valid
- Amount harus positive number
- Description minimal 10 chars
- Expense date tidak bisa future (kecuali Owner override)
- Status default: PENDING (kecuali Owner yang create → auto APPROVED)

**Expense Approval**
- Hanya OWNER dan ADMIN bisa approve
- Amount threshold: < 1.000.000 (Admin), >= 1.000.000 (Owner only)
- Approver tidak bisa approve expense yang dibuat sendiri (self-approval prevention)
- Rejection reason mandatory
- Approval/rejection immutable (tidak bisa diubah setelah di-approve/reject)

**Financial Reports**
- Report data real-time (tidak ada caching)
- Date range filter: max 1 tahun
- Export format: PDF (dengan branding), Excel (raw data)
- Report access: hanya OWNER dan ADMIN
- Report generation: async untuk large data (show loading indicator)

**Currency**
- Default currency: IDR (Indonesian Rupiah)
- Format: Rp 1.000.000 (dengan thousand separator)
- No decimal places untuk IDR
- Exchange rate: tidak support multi-currency (future feature)

#### **Edge Cases**

**Expense edit setelah approved**
- Tidak bisa edit expense yang sudah APPROVED
- Workaround: create new expense, cancel old expense
- Require Owner approval untuk cancel approved expense
- Log audit trail

**Expense delete**
- Soft delete: set deleted_at timestamp
- Hanya Owner bisa delete expense
- Delete hanya bisa jika status PENDING
- Approved/Rejected expenses tidak bisa di-delete
- Restore: Owner bisa restore within 30 days

**Duplicate expense detection**
- Saat create: check jika ada expense dengan amount + date + category yang sama
- Show warning: "Expense serupa sudah ada. Lanjutkan?"
- Allow user to proceed atau cancel

**Expense date manipulation**
- Staff input expense date di masa lalu
- System allow (untuk catch-up expenses)
- Require Owner approval jika date > 7 days ago
- Log audit trail dengan note "Backdated expense"

**Negative amount**
- System prevent negative amount
- If staff input negative: show error "Amount harus positive"
- Workaround untuk refund: create separate expense dengan type "REFUND"

**Receipt attachment failure**
- Upload gagal (network error, file too large)
- Show error message
- Allow retry
- Allow submit expense tanpa receipt (dengan warning)
- Mark expense sebagai "Missing receipt" untuk follow-up

**Approval timeout**
- Expense PENDING > 7 days: auto-notify Owner
- Expense PENDING > 30 days: escalate ke Owner (jika Admin yang approve)
- Display aging di expense list (warna berbeda untuk old pending)

**Concurrent approval**
- Two approvers try to approve same expense
- First one wins (database constraint)
- Second one get error "Expense sudah di-approve"
- Show current status

---

### **Modul 9: Settings & Administration**

**Tujuan**: Konfigurasi sistem dan manajemen user.

#### **Fitur Utama**

**User Management**

**Create User**
- Input fields:
  - Username (required, unique, alphanumeric, min 3 chars, max 20 chars)
  - Full name (required, min 2 chars, max 100 chars)
  - Role (required, enum: ADMIN, DOKTER, KASIR)
  - Initial PIN (optional, auto-generate 6-digit random jika kosong)
  - Email (optional, untuk notifikasi)
  - Phone (optional, untuk WhatsApp notifications)
  - Photo (optional, upload image)

- Validation rules:
  - Username: unique, alphanumeric only, no spaces
  - Full name: required, no special characters except space, dash, apostrophe
  - Role: harus salah satu dari enum
  - PIN: 6-digit numeric, auto-generate jika kosong
  - Email: valid format jika diisi
  - Phone: Indonesian format jika diisi

- Auto-generated fields:
  - User ID (UUID)
  - PIN hash (bcrypt, salt rounds 12)
  - Status (default: ACTIVE)
  - Created at, updated at timestamps
  - Failed login attempts (default: 0)

**User List View**
- Display: table
- Columns: photo, username, full name, role, status, last login, actions
- Filter: by role, status
- Sort: by username, full name, last login
- Search: by username, full name
- Actions: edit, reset PIN, deactivate/reactivate, force logout

**User Detail View**
- Header: photo, username, full name, role badge, status badge
- Sections:
  - Basic info: username, full name, email, phone
  - Security: last login, failed attempts, locked until
  - Activity: recent actions (dari audit log)
  - Sessions: active sessions (device, IP, last active)

**Deactivate User**
- Soft delete: set is_active = false
- User tidak bisa login lagi
- History tetap ter-audit
- Existing data (medical records, invoices) tetap intact
- Reactivate: Owner bisa reactivate dengan set is_active = true
- Require new PIN saat reactivate (optional)

**Change PIN (Self-Service)**
- User input old PIN → verify
- User input new PIN (6-digit)
- User confirm new PIN
- System hash new PIN dengan bcrypt
- System invalidate all active sessions
- User harus login ulang
- Log audit trail

**Reset PIN (Owner/Admin)**
- Owner bisa reset PIN siapa saja
- Admin bisa reset PIN customer saja
- Generate random 6-digit PIN atau set custom PIN
- Display new PIN one-time (copy to clipboard)
- Invalidate all active sessions
- Notify user via WhatsApp (optional)
- Log audit trail

**Force Logout**
- Owner bisa force logout user dari semua devices
- Invalidate all active sessions
- User harus login ulang
- Notify user tentang force logout
- Log audit trail

**System Settings**

**Business Information**
- Business name (required)
- Address (required)
- Phone (required)
- Email (required)
- Logo (optional, upload image)
- Tax ID (optional, untuk invoice)
- Business hours (required, per day: open time, close time)
- Timezone (required, default: Asia/Jakarta)

**Payment Settings**
- Midtrans merchant ID (untuk payment gateway)
- Midtrans server key (encrypted)
- Midtrans client key
- QRIS enabled/disabled (toggle)
- Bank accounts (multiple):
  - Bank name
  - Account number
  - Account holder name
  - Branch (optional)
- E-wallet providers:
  - GoPay (enabled/disabled)
  - OVO (enabled/disabled)
  - Dana (enabled/disabled)
  - ShopeePay (enabled/disabled)
- Credit card (enabled/disabled)
- Payment terms (default: immediate)

**Notification Settings**
- WhatsApp:
  - Fonnte API key (encrypted)
  - Fonnte sender number
  - Templates:
    - Appointment reminder
    - Invoice sent
    - Pet hotel check-in
    - Pet hotel daily update
    - Pet hotel check-out
    - Grooming reminder
    - Promotion
    - Loyalty tier upgrade
  - Enable/disable per template
  - Timing:
    - Appointment reminder: 24 jam sebelum (configurable)
    - Invoice sent: immediate
    - Pet hotel update: real-time
- Email:
  - Resend API key (encrypted)
  - Sender email
  - Sender name
  - Templates:
    - Appointment confirmation
    - Invoice
    - Feedback request
    - Password reset (future)
  - Enable/disable per template
- In-app notifications:
  - Enable/disable per notification type
  - Sound on/off
  - Desktop notifications (browser)

**Loyalty Settings**
- Points ratio: berapa rupiah per 1 point (default: 1000)
- Tier thresholds:
  - BRONZE: 0-100 points
  - SILVER: 101-500 points
  - GOLD: 501-2000 points
  - PLATINUM: 2000+ points
- Tier benefits:
  - Discount percentage per tier
  - Free services per month per tier
  - Birthday offer per tier
- Points expiry: berapa bulan tidak aktif sebelum expire (default: 12)
- Bonus points:
  - Birthday bonus (multiplier)
  - Referral bonus (fixed points)
  - Product-specific bonus (per product)

**Tax Settings**
- PPN (PPN) enabled/disabled (toggle)
- PPN rate (default: 11%)
- Service tax enabled/disabled (toggle)
- Service tax rate (default: 0%)
- Tax inclusive/exclusive (default: exclusive)
- Tax ID (untuk invoice)

**Appointment Settings**
- Business hours (per day)
- Appointment duration default (per service type):
  - Consultation: 30 minutes
  - Vaccination: 15 minutes
  - Check-up: 30 minutes
  - Surgery: 60 minutes
  - Grooming: 45 minutes
- Buffer time between appointments (default: 15 minutes)
- Max appointments per doctor per day (default: 20)
- Auto-assign queue number (enabled/disabled)
- No-show threshold (minutes, default: 30)
- Reminder timing (hours before, default: 24)

**Pet Hotel Settings**
- Room types (customizable):
  - Suite, Standard, Budget
  - Price per night per type
  - Capacity per type
- Check-in time (default: 14:00)
- Check-out time (default: 12:00)
- Late check-out fee (per hour)
- Early check-in fee (per hour)
- Daily care log types (customizable):
  - Feeding, Medicine, Walk, Play, Observation, Other
- Photo upload requirement (mandatory/optional)

**Grooming Settings**
- Service categories (customizable):
  - Bath, Haircut, Nail Trim, Ear Clean, Teeth Clean, Full Grooming
- Default duration per service
- Groomer roles (customizable)
- Before/after photo requirement (mandatory/optional)

**Inventory Settings**
- Low stock threshold (default: 10)
- Auto-create PO saat low stock (enabled/disabled)
- Stock opname frequency (weekly, monthly, quarterly)
- Expiry alert timing (days before expiry):
  - 30 days before
  - 7 days before
  - On expiry date
- Barcode format (EAN-13, Code-128, QR)

**POS Settings**
- Receipt printer (enabled/disabled)
- Receipt format (thermal A6, A5, A4)
- Auto-print receipt setelah payment (enabled/disabled)
- Cash shift required (enabled/disabled)
- Opening cash amount (default: 500.000)
- Mixed payment allowed (enabled/disabled)
- Invoice number format (customizable prefix)

**Security Settings**
- PIN length (default: 6, max: 8)
- Failed login attempts before lockout (default: 5)
- Lockout duration (minutes, default: 15)
- Session duration (hours, default: 24)
- Remember me duration (days, default: 7)
- Password complexity (untuk future password-based auth)
- 2FA (enabled/disabled, untuk future)
- IP whitelist (optional, untuk restrict access)

#### **Business Rules**

**User Creation**
- Username harus unique (case-sensitive)
- Username tidak bisa sama dengan reserved words (admin, owner, system)
- PIN auto-generate jika tidak di-input
- PIN di-hash dengan bcrypt sebelum save
- Role harus valid enum
- Status default: ACTIVE

**PIN Management**
- PIN harus 6-digit numeric
- PIN di-hash dengan bcrypt (salt rounds 12)
- PIN change: invalidate all sessions
- PIN reset: generate random atau custom
- PIN tidak bisa sama dengan 5 PIN terakhir (history check)

**System Settings**
- Settings hanya bisa diubah oleh OWNER
- Beberapa settings require restart aplikasi (rare)
- Settings change logged di audit trail
- Settings validation: check valid values sebelum save

**Notification Settings**
- API keys di-encrypt di database
- API keys tidak bisa di-view setelah save (hanya edit)
- Test notification: send test message sebelum save
- Template variables: {{customer_name}}, {{pet_name}}, {{date}}, dll

#### **Edge Cases**

**Username change**
- Username change: invalidate all sessions
- User harus login ulang dengan username baru
- Update semua references (audit log, etc)
- Notify user tentang username change

**Role change**
- Role change: invalidate all sessions
- Update permissions immediately
- Log audit trail
- Notify user tentang role change

**Last Owner standing**
- Prevent deactivate last OWNER
- Show error: "Tidak bisa deactivate OWNER terakhir"
- Require transfer ownership ke user lain dulu

**Settings reset**
- Owner bisa reset settings ke default
- Require confirmation
- Log audit trail
- Notify all users tentang settings change

**Concurrent settings edit**
- Two Owners edit settings bersamaan
- Last write wins (dengan warning)
- Show "Settings diubah oleh [user] pada [time]"
- Allow override atau cancel

**API key validation**
- Test API key sebelum save
- Show error jika API key invalid
- Allow save tanpa test (dengan warning)
- Store encrypted API key

---

### **Modul 10: Reports & Analytics (Dashboard)**

**Tujuan**: Business intelligence untuk Owner dan Management.

#### **Fitur Utama**

**Executive Dashboard (Owner View)**

**KPI Cards (Key Performance Indicators)**
- Total revenue today (vs yesterday, % change)
- Total revenue this month (vs last month, % change)
- Total revenue this year (vs last year, % change)
- Total transactions today
- Average transaction value
- Top customer (by spending, this month)
- Conversion rate (booking → completion)
- Staff performance (appointments completed per staff)
- Low stock items count
- Pending expenses count

**Charts**
- Revenue trend (line chart: daily, weekly, monthly)
- Revenue by category (pie chart: products, services, pet hotel, grooming)
- Top products (bar chart: by quantity sold)
- Top services (bar chart: by revenue)
- Customer acquisition (line chart: new customers per month)
- Expense breakdown (pie chart: by category)
- Profit margin trend (line chart: monthly)
- Cash flow (bar chart: cash in vs cash out)

**Quick Actions**
- Create appointment
- Create invoice
- View low stock items
- View pending expenses
- Export report

**Service Reports**

**Appointment Report**
- Total appointments (completed, cancelled, no-show)
- Appointments by doctor (bar chart)
- Average appointment duration
- Doctor utilization rate (% of available slots filled)
- Appointment types breakdown (pie chart)
- No-show rate (percentage)
- Cancellation rate (percentage)
- Top complaints (word cloud)
- Date range filter
- Export: PDF, Excel

**Pet Hotel Report**
- Total bookings (by status)
- Occupancy rate (percentage)
- Revenue (by room type)
- Average stay duration (days)
- Room utilization (by room)
- Peak season (monthly trend)
- Cancellation rate
- Daily care logs count
- Date range filter
- Export: PDF, Excel

**Grooming Report**
- Total grooming services
- Revenue (by service type)
- Popular services (bar chart)
- Groomer utilization (by groomer)
- Average grooming duration
- Before/after photos count
- Cancellation rate
- Date range filter
- Export: PDF, Excel

**Customer Reports**

**Customer Insights**
- Total customers (active, inactive, guest)
- Customer acquisition trend (line chart)
- Customer churn rate (percentage)
- Repeat purchase rate (percentage)
- Customer lifetime value (average)
- Top customers (by spending, table)
- Customer segmentation (by tier, pie chart)
- Geographic distribution (map, future feature)
- Date range filter
- Export: PDF, Excel

**Loyalty Report**
- Total loyalty members (by tier)
- Points issued vs redeemed
- Points expiry forecast
- Tier upgrade/downgrade trend
- Top redeemers
- Redemption catalog popularity
- Date range filter
- Export: PDF, Excel

**Inventory Reports**

**Stock Report**
- Low stock items (table, sorted by urgency)
- Out of stock items
- Stock value (total inventory value)
- Fast-moving items (top 20 by quantity sold)
- Slow-moving items (bottom 20 by quantity sold)
- Expiry tracking (items expiring in 30/60/90 days)
- Stock turnover ratio
- Dead stock (no movement in 90 days)
- Export: PDF, Excel

**Purchase Order Report**
- Total POs (by status)
- Total PO value
- Average PO processing time
- Top suppliers (by PO value)
- PO discrepancies (ordered vs received)
- Supplier performance (on-time delivery rate)
- Date range filter
- Export: PDF, Excel

**Financial Reports**

**Revenue Report**
- Total revenue (by category, by service type)
- Revenue trend (daily, weekly, monthly)
- Revenue by payment method
- Revenue by staff (kasir)
- Revenue by customer segment
- Year-over-year comparison
- Date range filter
- Export: PDF, Excel

**Expense Report**
- Total expenses (by category)
- Expense trend (monthly)
- Top expense items
- Expense vs budget (jika budget di-set)
- Expense approval rate
- Average approval time
- Date range filter
- Export: PDF, Excel

**Profit & Loss Report**
- Total revenue
- Total COGS
- Gross profit
- Total expenses
- Net profit
- Profit margin percentage
- Breakdown by month
- Year-over-year comparison
- Date range filter
- Export: PDF, Excel

**Custom Reports**
- Build custom report dengan drag-and-drop
- Select fields, filters, grouping
- Save report template
- Schedule report (daily, weekly, monthly)
- Auto-email report ke Owner/Admin
- Export: PDF, Excel, CSV

#### **Business Rules**

**Dashboard Access**
- Hanya OWNER dan ADMIN bisa akses dashboard
- KASIR bisa akses limited dashboard (own performance only)
- DOKTER bisa akses limited dashboard (own appointments only)
- Data real-time (auto-refresh setiap 5 menit)

**Report Generation**
- Report data aggregated dari database
- Date range filter: max 1 tahun
- Large reports: async generation (show progress)
- Report caching: cache untuk 1 jam (configurable)
- Export format: PDF (dengan branding), Excel (raw data), CSV

**Data Privacy**
- Customer data di-report: anonymized jika export ke external
- Financial data: hanya OWNER bisa lihat full detail
- Staff performance: hanya OWNER dan ADMIN bisa lihat
- Audit trail: semua report access logged

**Report Scheduling**
- Schedule report: daily, weekly, monthly
- Auto-email ke recipients (Owner, Admin)
- Report format: PDF attachment
- Schedule timezone: follow business timezone
- Pause/resume schedule

#### **Edge Cases**

**Large dataset reports**
- Report dengan > 100.000 rows
- Async generation dengan progress indicator
- Pagination di UI (jika view online)
- Export ke Excel (limit 1.000.000 rows)
- Show warning jika report akan lama

**Date range invalid**
- Start date > end date: show error
- Date range > 1 tahun: show warning, allow proceed
- Future date: prevent selection
- No data in range: show "No data available"

**Report export failure**
- Export gagal (network error, file too large)
- Show error message
- Allow retry
- Offer smaller date range
- Provide download link (async)

**Concurrent report generation**
- Two users generate same report bersamaan
- Share cached result (jika available)
- Otherwise: generate separately
- Show "Report sedang di-generate oleh [user]"

**Real-time data staleness**
- Dashboard data stale > 5 menit
- Show "Last updated: [time]"
- Manual refresh button
- Auto-refresh on focus (saat user kembali ke tab)

---

### **Modul 11: Customer Portal**

**Tujuan**: Self-service interface untuk pelanggan.

#### **Fitur Utama**

**Home Page**
- Welcome message (dengan customer name)
- Upcoming appointments (next 7 days):
  - Date, time, doctor, pet name
  - Status badge
  - Quick actions: reschedule, cancel
- Pet hotel bookings (current/upcoming):
  - Check-in/check-out dates
  - Room info
  - Status badge
  - Quick actions: view care logs
- Loyalty points balance + tier
- Quick links:
  - Book appointment
  - Browse shop
  - View invoices
  - Contact us

**My Pets**
- List semua pets milik customer
- Pet card dengan:
  - Photo
  - Name
  - Species, breed
  - Age
  - Quick stats: last visit, next vaccine due
- Tap pet untuk lihat detail:
  - Overview: biodata lengkap
  - Medical history: timeline medical records
  - Vaccination schedule: upcoming vaccines
  - Weight chart: grafik berat badan
  - ID card: view/download ID card

**Appointments**
- Upcoming appointments:
  - List dengan date, time, doctor, pet, status
  - Actions: reschedule, cancel, view detail
- Past appointments:
  - List dengan date, doctor, diagnosis (truncated)
  - Tap untuk lihat full medical record
- Book new appointment:
  - Step 1: Select pet
  - Step 2: Select service type (consultation, vaccination, check-up)
  - Step 3: Select doctor (optional, atau "Any available")
  - Step 4: Select date (calendar view, show available slots)
  - Step 5: Select time (time slots, show availability)
  - Step 6: Add complaint/notes
  - Step 7: Confirm booking
  - Auto-send WhatsApp confirmation

**Pet Hotel Bookings**
- Upcoming bookings:
  - Check-in/check-out dates
  - Room info
  - Special notes
  - Status badge
- Current stays:
  - Daily care logs (timeline)
  - Photos dari staff
  - Real-time updates
- Past bookings:
  - History dengan dates, room, total price
  - Download invoice
- Book new stay:
  - Step 1: Select pet
  - Step 2: Select check-in date
  - Step 3: Select check-out date
  - Step 4: Select room type (auto-suggest available)
  - Step 5: Add special notes (makanan, obat, behavior)
  - Step 6: Confirm booking
  - Auto-send WhatsApp confirmation

**Invoices & Payments**
- List all invoices:
  - Invoice number, date, total, status
  - Filter: by status (pending, paid, partial)
  - Sort: by date desc
- Invoice detail:
  - Items list
  - Summary (subtotal, discount, tax, total)
  - Payment info
  - Download PDF
- Pay online (jika unpaid):
  - Select payment method (QRIS, transfer, e-wallet)
  - Process payment via Midtrans
  - Auto-update invoice status
  - Send receipt via email/WhatsApp
- Payment history:
  - List semua payments
  - Date, amount, method, reference

**Loyalty**
- Points balance
- Tier (Bronze, Silver, Gold, Platinum)
- Points history:
  - List semua transactions (earn, redeem, expire)
  - Date, description, points (+/-)
  - Filter: by type, date range
- Upcoming tier benefits:
  - Points needed untuk next tier
  - Benefits preview
- Redemption options:
  - Catalog of rewards
  - Points required per reward
  - Redeem button (confirm modal)
- Referral program:
  - Referral code
  - Share via WhatsApp
  - Track referrals

**Shop (E-commerce Minimal)**
- Browse products:
  - Grid view dengan photo, name, price
  - Filter: by category
  - Search: by name
  - Sort: by price, popularity
- Product detail:
  - Photo gallery
  - Description
  - Price
  - Stock availability
  - Add to cart button
- Shopping cart:
  - List items
  - Quantity picker
  - Subtotal
  - Checkout button
- Checkout:
  - Select delivery method (pickup, delivery)
  - Delivery address (jika delivery)
  - Payment method
  - Apply voucher (optional)
  - Confirm order
- Order confirmation:
  - Order number
  - Items list
  - Total
  - Estimated delivery/pickup time
  - Track order status

**Notifications**
- Notification center (bell icon):
  - List semua notifications
  - Unread badge
  - Mark as read
- Notification types:
  - Appointment reminder
  - Invoice ready
  - Payment received
  - Promotion/offer
  - Pet hotel update
  - Loyalty tier upgrade
  - Points expiring soon
- Push notifications (browser):
  - Enable/disable
  - Per type toggle
- WhatsApp notifications:
  - Same types as in-app
  - Enable/disable per type

**Profile**
- Edit personal info:
  - Name
  - Email
  - Phone
  - Address
  - Photo
- Change PIN:
  - Input old PIN
  - Input new PIN
  - Confirm new PIN
- Login history:
  - List semua logins
  - Date, time, device, IP
  - Active sessions
- Notification preferences:
  - Toggle per notification type
  - WhatsApp, Email, In-app
- Logout:
  - Confirm modal
  - Invalidate session

#### **Business Rules**

**Portal Access**
- Hanya registered customers bisa akses portal
- Login: username (email/phone) + PIN
- Session: 7 days (remember me)
- Role: CUSTOMER (limited access)

**Appointment Booking**
- Customer bisa book untuk own pets only
- Check doctor availability real-time
- Prevent double booking
- Auto-send confirmation via WhatsApp
- Allow cancel/reschedule (dengan rules):
  - Cancel: H-1 sebelum appointment (free)
  - Reschedule: H-1 sebelum appointment (free)
  - Late cancel: charge fee (optional)

**Pet Hotel Booking**
- Customer bisa book untuk own pets only
- Check room availability real-time
- Auto-calculate price (nights × price per night)
- Auto-send confirmation via WhatsApp
- Allow cancel (dengan rules):
  - Cancel: H-3 sebelum check-in (free)
  - Late cancel: charge fee (optional)

**Invoice Payment**
- Customer bisa pay own invoices only
- Payment via Midtrans (QRIS, transfer, e-wallet)
- Auto-update invoice status setelah payment success
- Auto-send receipt via email/WhatsApp
- Payment timeout: 15 menit (QRIS), manual (transfer)

**Loyalty Redemption**
- Customer bisa redeem own points only
- Check points balance sebelum redeem
- Auto-deduct points setelah redeem
- Auto-send confirmation via WhatsApp
- Points tidak bisa transfer ke customer lain

**Shop Checkout**
- Customer bisa checkout untuk own cart only
- Check stock availability real-time
- Auto-calculate total (items + delivery fee)
- Payment via Midtrans
- Auto-send order confirmation via WhatsApp
- Order status: PENDING → PROCESSING → READY → COMPLETED

#### **Edge Cases**

**Portal login failure**
- Wrong PIN: show error, increment failed attempts
- Account locked: show message, countdown timer
- Session expired: redirect to login, preserve URL
- Network error: show error, allow retry

**Appointment booking conflict**
- Doctor sudah booked di waktu tersebut
- Show error "Dokter tidak available di waktu tersebut"
- Suggest alternative time slots
- Suggest other doctors

**Pet hotel booking conflict**
- Room sudah booked di tanggal tersebut
- Show error "Room tidak available"
- Suggest alternative rooms
- Suggest alternative dates

**Payment failure**
- Payment gateway error
- Show error message
- Allow retry
- Keep invoice as UNPAID
- Notify staff tentang failed payment

**Out of stock during checkout**
- Product out of stock saat customer checkout
- Show error "Product [name] sudah habis"
- Allow remove dari cart
- Suggest alternative products

**Concurrent booking**
- Two customers book same slot bersamaan
- First one wins (database constraint)
- Second one get error "Slot sudah di-book"
- Suggest alternative slots

**Photo upload failure**
- Upload gagal (network error, file too large)
- Show error message
- Allow retry
- Compress image sebelum upload

---

## **VII. FITUR CROSS-CUTTING (HORIZONTAL)**

### **A. Authentication & Authorization**

**PIN-Based Login**
- 6-digit PIN, bcrypt hashed (salt rounds 12)
- Username + PIN login flow
- Session: JWT token, 24h validity, HTTP-only cookie
- Remember me: extend to 7 days

**RBAC (Role-Based Access Control)**
- 5 roles: OWNER, ADMIN, DOKTER, KASIR, CUSTOMER
- Permission matrix: define akses per role per modul
- Enforce di frontend (hide UI) + backend (RLS)

**RLS (Row Level Security)**
- Database-level enforcement
- Policies per table per role
- Prevent data leak (customer A tidak bisa lihat data customer B)
- Audit: semua query logged

**Session Management**
- JWT token di HTTP-only cookie
- Auto-refresh sebelum expiry
- Force logout: invalidate all sessions
- Active sessions list (device, IP, last active)

**Failed Login Lockout**
- 5 failed attempts → lock 15 menit
- Show countdown timer
- Owner bisa unlock manual
- Log audit trail

**Audit Logging**
- Semua actions tercatat: create, update, delete, login, logout
- Fields: user_id, action, entity_type, entity_id, before_values, after_values, timestamp, IP
- Immutable: tidak bisa edit/delete
- Retention: 2 tahun, archive setelah itu

### **B. Realtime Features**

**Appointment Queue**
- Live update saat status berubah (WAITING → IN_PROGRESS → DONE)
- WebSocket subscription ke appointments table
- Auto-refresh queue UI
- Sound notification saat ada perubahan

**Pet Hotel Logs**
- Staff post foto → auto-notify customer (WhatsApp + in-app)
- Customer portal: real-time update care logs
- WebSocket subscription ke pet_hotel_logs table

**Notification Bell**
- Real-time notifications (in-app)
- Unread badge counter
- Mark as read (single, all)
- WebSocket subscription ke notifications table

**Shared Dashboard**
- Multiple staff bisa view same data real-time
- Auto-refresh saat ada perubahan
- Conflict resolution: last write wins (dengan warning)

**POS Stock Update**
- Real-time stock update saat ada transaksi
- Prevent overselling (concurrent transactions)
- WebSocket subscription ke stock_levels table

### **C. Notifications**

**WhatsApp (Fonnte)**
- Appointment reminder (H-1)
- Invoice sent
- Pet hotel check-in/out
- Pet hotel daily update (dengan foto)
- Grooming reminder
- Promotion/offer
- Loyalty tier upgrade
- Template-based (customizable)
- Enable/disable per template

**Email (Resend)**
- Invoice
- Appointment confirmation
- Feedback request
- Password reset (future)
- Template-based (customizable)
- Enable/disable per template

**In-App Notifications**
- Bell icon dengan unread badge
- Notification center (list)
- Types: appointment, invoice, promotion, loyalty, system
- Mark as read (single, all)
- Real-time (WebSocket)

**Push Notifications (Browser)**
- Enable/disable
- Per type toggle
- Service worker untuk background notifications
- Click action: navigate to related page

**SMS (Optional)**
- Critical notifications only (payment failure, system alert)
- Provider: Twilio (future)
- Enable/disable

### **D. Search & Global Search**

**Local Search**
- Per page search (customers, products, appointments)
- Debounced input (300ms)
- Case-insensitive, partial match
- Highlight matching text

**Global Search (Cmd+K / Ctrl+K)**
- Search across: customers, pets, appointments, products, invoices
- Fuzzy search (typo-tolerant)
- Quick jump to entity (navigate to detail page)
- Recent searches (localStorage)
- Keyboard-friendly (arrow keys, Enter, Esc)
- Search results grouped by type

**Search Filters**
- Filter by type (customer, pet, appointment, product)
- Filter by date range
- Filter by status
- Save search presets

**Search Performance**
- Full-text search (PostgreSQL tsvector)
- Indexes untuk search columns
- Pagination untuk large results
- Caching untuk frequent searches

### **E. Audit Trail**

**Every Change Logged**
- User action: create, update, delete
- Before + after values (JSON)
- Timestamp (UTC)
- Performed by (user_id)
- IP address
- User agent (browser, device)

**Soft Deletes**
- deleted_at timestamp
- Nothing truly deleted (kecuali hard delete oleh Owner)
- Restore: within 30 days
- After 30 days: permanent delete (scheduled job)

**Compliance**
- Audit log untuk internal audit
- Audit log untuk regulatory compliance (jika perlu)
- Export audit log: CSV, PDF
- Retention policy: 2 tahun

**Audit Log View**
- List semua audit logs
- Filter: by user, action, entity, date range
- Sort: by timestamp desc
- Detail view: before/after values (diff view)
- Export: CSV, PDF

### **F. File Management**

**Photo Uploads**
- Customer photo
- Pet photo
- Medical attachments (lab results, scan)
- Grooming before/after
- Expense receipts
- Product photos

**File Specifications**
- Max file size: 5MB (configurable)
- Allowed formats: jpg, png, webp, pdf
- Auto-resize: compress image (max 1920x1080)
- Generate thumbnails: multiple sizes (100x100, 300x300, 600x600)
- EXIF data: strip untuk privacy

**Storage**
- Supabase Storage (S3-compatible)
- Bucket per entity type (customers, pets, medical, grooming, expenses, products)
- Access control: private per user/customer (via RLS)
- CDN: auto-cache di edge

**File Operations**
- Upload: drag-and-drop atau click to select
- Preview: before upload
- Progress: upload progress bar
- Delete: soft delete (mark as deleted)
- Download: original atau thumbnail
- Replace: upload new version (keep history)

### **G. Validation & Error Handling**

**Schema Validation (Zod)**
- Frontend: instant feedback saat user input
- Backend: double-check di Edge Function
- Shared schemas: reusable di frontend & backend
- Type inference: generate TypeScript types dari schemas

**Type Safety (TypeScript)**
- Strict mode: no implicit any
- Type checking: compile-time errors
- Type inference: automatic dari Zod schemas
- No type casting (kecuali absolutely necessary)

**Error Messages**
- User-friendly: Bahasa Indonesia
- Actionable: suggest next steps
- Contextual: show which field has error
- Consistent: same error = same message

**Error Logging (Sentry)**
- Capture all errors (frontend + backend)
- Breadcrumbs: user actions before error
- Context: user, device, browser
- Release tracking: version deployment
- Alert: Slack/email untuk critical errors

**Network Error Handling**
- Retry logic: automatic retry (3x, exponential backoff)
- Offline support: queue actions, sync saat online (future)
- Timeout: 30 detik (configurable)
- Fallback: show cached data jika available

### **H. Performance & Caching**

**React Query (Solid Query)**
- Smart caching: cache per query key
- Automatic refetch: on focus, on reconnect, interval
- Optimistic updates: instant UI update, rollback jika error
- Background refetch: stale-while-revalidate
- Cache time: 5 menit (configurable per query)

**Data Table Pagination**
- Lazy load: 50 items per page
- Infinite scroll (optional)
- Virtual scrolling untuk large lists (> 1000 items)
- Sort + filter di server (bukan client)

**Image Optimization**
- Responsive images: srcset untuk different screen sizes
- Lazy loading: load saat visible
- WebP format: smaller file size
- CDN: auto-cache di edge
- Blur placeholder: show blur saat loading

**Bundle Size**
- Target: < 500KB (gzip)
- Code splitting: lazy load pages
- Tree shaking: remove unused code
- Dynamic imports: load on demand

**Lighthouse**
- Target: ≥90 pada semua metrics
- Performance: load time, TTI, TBT
- Accessibility: ARIA labels, keyboard navigation
- Best practices: HTTPS, no console errors
- SEO: meta tags, structured data

### **I. Accessibility (a11y)**

**WCAG 2.1 AA Compliance**
- Semantic HTML: proper heading hierarchy, landmarks
- ARIA labels: untuk custom components
- Keyboard navigation: Tab, Enter, Esc, Arrow keys
- Color contrast: 4.5:1 minimum (text), 3:1 (large text)
- Screen reader support: test dengan NVDA, VoiceOver

**Focus Management**
- Visible focus indicators
- Focus trap di modals/dialogs
- Skip to content link
- Restore focus setelah modal close

**Forms Accessibility**
- Labels: associated dengan inputs
- Error messages: linked dengan inputs (aria-describedby)
- Required fields: marked dengan asterisk + aria-required
- Autocomplete: proper autocomplete attributes

**Testing**
- axe-core: automated accessibility testing
- Manual testing: keyboard-only navigation
- Screen reader testing: NVDA, VoiceOver
- Color contrast checker: WebAIM

### **J. Keyboard Shortcuts**

**Global Shortcuts**
- `Cmd+K` / `Ctrl+K`: Global search
- `Cmd+/` / `Ctrl+/`: Keyboard shortcuts help
- `Cmd+1-9` / `Ctrl+1-9`: Jump to menu item (staff dashboard)
- `Esc`: Close modal/dialog
- `?`: Show keyboard shortcuts help

**Navigation**
- `Tab`: Navigate form fields
- `Shift+Tab`: Navigate backwards
- `Enter`: Submit form / Select item
- `Arrow keys`: Navigate lists/menus
- `Home`: Jump to first item
- `End`: Jump to last item

**POS Shortcuts**
- `F1`: Focus search bar
- `F2`: Add new item
- `F3`: Apply discount
- `F4`: Checkout
- `F5`: Clear cart
- `F12`: Print receipt

**Customization**
- User bisa customize shortcuts (future)
- Save preferences di localStorage
- Export/import shortcuts config

---

## **VIII. DATABASE SCHEMA — OVERVIEW**

### **Auth Domain**
- `users` — staff accounts (OWNER, ADMIN, DOKTER, KASIR) dengan username, PIN hash, role, status
- `audit_logs` — semua perubahan system dengan user_id, action, entity, before/after values, timestamp

### **CRM Domain**
- `customers` — pelanggan/pet owners dengan nama, phone, email, address, tags, photo
- `pets` — hewan peliharaan dengan nama, species, breed, birth_date, gender, weight, microchip
- `pet_vaccines` — vaksin history per pet dengan vaccine name, date given, next due date
- `pet_diseases` — penyakit per pet dengan type, name, severity, status
- `pet_allergies` — alergi per pet dengan allergen, severity, notes
- `pet_weight_logs` — weight tracking per pet dengan date, weight

### **Medical Domain**
- `appointments` — jadwal kunjungan dengan customer, pet, doctor, date, time, status, queue_number
- `medical_records` — EMR dengan appointment, chief_complaint, diagnosis, treatment, prescription
- `procedures` — master data jenis procedure/treatment dengan name, description, price

### **Pet Hotel Domain**
- `rooms` — ruangan/kandang dengan name, type, capacity, price_per_night, status, cleanliness
- `pet_hotel_bookings` — booking history dengan customer, pet, room, check-in/out dates, status
- `pet_hotel_logs` — daily care logs per booking dengan type, description, photo, staff

### **Grooming Domain**
- `grooming_services` — master data jenis grooming dengan name, category, duration, price
- `grooming_bookings` — booking history dengan customer, pet, service, groomer, date, time, status

### **Inventory Domain**
- `products` — master data produk dengan name, SKU, barcode, category, prices, stock, expiry
- `stock_levels` — current stock per product dengan current_stock, min_level, max_level
- `stock_movements` — audit trail stock dengan type, quantity, reason, reference
- `purchase_orders` — PO to suppliers dengan supplier, items, status, total
- `suppliers` — master data supplier dengan name, contact, address

### **POS Domain**
- `invoices` — invoice header dengan customer, type, items, total, payment_status
- `invoice_items` — detail items per invoice dengan product/service, quantity, price
- `invoice_payments` — payment records per invoice dengan method, amount, reference
- `cash_shifts` — daily cash reconciliation dengan opening, closing, variance, status

### **Loyalty Domain**
- `loyalty_accounts` — loyalty per customer dengan current_points, tier
- `loyalty_transactions` — point earn/redeem history dengan type, points, description

### **Promotions Domain**
- `promotions` — promo master data dengan type, rules, valid_dates, status
- `promotion_redemptions` — track redemption per customer dengan promotion, customer, date

### **Finance Domain**
- `expenses` — pengeluaran operasional dengan category, amount, description, status
- `expense_approvals` — approval workflow dengan expense, approver, status, notes

### **Feedback Domain**
- `feedback` — customer feedback/reviews dengan transaction, rating, comment, photos

### **Notifications Domain**
- `notifications` — in-app notifications dengan user, type, title, message, read_status

### **Common Fields (Semua Tables)**
- `id` — UUID primary key
- `created_at` — auto-timestamp (UTC)
- `updated_at` — auto-timestamp (UTC)
- `deleted_at` — soft delete timestamp (null jika active)
- `created_by` — user_id yang create
- `updated_by` — user_id yang terakhir update

### **Database Features**
- **RLS Policies**: enforce di setiap table, per role
- **Indexes**: untuk frequently queried columns (search, filter, sort)
- **Foreign keys**: referential integrity
- **Check constraints**: valid values (enum, range)
- **Unique constraints**: prevent duplicates
- **Triggers**: auto-update timestamps, auto-calculate fields
- **Functions**: reusable logic (calculate age, format currency)
- **Views**: pre-aggregated data untuk reports

---

## **IX. FITUR UNGGULAN YANG MEMBEDAKAN**

1. **Unified Medical Records** — Hewan punya EMR lengkap yang accessible oleh dokter dan customer, dengan timeline view dan search

2. **Realtime Pet Hotel Logs** — Customer dapat update foto/video hewan mereka selama di pet hotel via WhatsApp dan portal

3. **Loyalty Integration** — Points earn otomatis + tier benefits terintegrasi di POS, dengan redemption catalog

4. **Multi-Service Bundling** — Satu invoice bisa berisi appointment + grooming + hotel + products, dengan auto-calculate

5. **WhatsApp Integration** — Invoice, reminder, notifikasi langsung ke customer via WhatsApp dengan customizable templates

6. **Virtual Queue** — Staff lihat antrian real-time, customer tau urutan mereka, dengan estimated wait time

7. **Expense Approval Workflow** — Kontrol pengeluaran dengan approval chain berdasarkan amount threshold

8. **Complete Audit Trail** — Semua aksi tercatat dengan before/after values untuk compliance dan troubleshooting

9. **Portal for Customers** — Customer self-service: booking, lihat medical records, loyalty points, shop, pay invoices

10. **Staff Accessibility** — Keyboard shortcuts + dark mode + realtime updates untuk operasional cepat dan efisien

11. **Fine-Grained Reactivity** — SolidJS signals untuk performa tinggi, tanpa virtual DOM overhead

12. **Type-Safe Full Stack** — TypeScript strict mode + Zod validation di semua layer, dari database sampai UI

13. **Offline-First Ready** — Arsitektur siap untuk offline support (future feature) dengan service worker

14. **Multi-Language Ready** — i18n structure untuk future expansion ke bahasa lain

15. **White-Label Capable** — Business branding (logo, colors) customizable per instance

---

## **X. GOVERNANCE & CONTRACT-DRIVEN DEVELOPMENT**

### **Master Documents**

Petora menggunakan **contract-first development model** dengan 4 master documents sebagai sumber kebenaran tunggal:

**1. master-arsitektur.md — Architecture Law**
- Database schema lengkap (semua tables, columns, types, constraints)
- TypeScript types (shared antara frontend dan backend)
- Validation schemas (Zod)
- RLS policies (per table, per role)
- API contracts (endpoints, request/response shapes)
- Error codes dan messages

**2. master-spesifikasi-frontend.md — Frontend Law**
- UI components library (design tokens, variants, props)
- Layout patterns (sidebar, header, bottom nav)
- Routing structure (file-based routing conventions)
- State management patterns (signals, stores, resources)
- Styling conventions (Tailwind utilities, component patterns)
- Accessibility requirements (WCAG 2.1 AA)

**3. master-spesifikasi-modul.md — Business Logic Law**
- Workflows per modul (state machines, transitions)
- Business rules (validation, calculations)
- Edge cases dan handling
- Permission matrix (per role, per action)
- Notification triggers (kapan, ke siapa, via channel apa)
- Audit requirements (apa yang di-log)

**4. master-test-cidi.md — Quality Law**
- Testing strategy (unit, integration, e2e)
- Quality gates (coverage thresholds, performance budgets)
- CI/CD pipeline (lint, test, build, deploy)
- Code review checklist
- Definition of Done (DoD)
- Release process

### **Development Principles**

**Contract-First**
- Define contracts dulu (types, schemas, APIs)
- Implement berdasarkan contracts
- Contracts immutable (kecuali via RFC process)
- Breaking changes: require major version bump

**Single Source of Truth**
- Master documents adalah sumber kebenaran
- Code harus follow contracts
- Discrepancy: fix code atau update contract (via RFC)

**RFC Process (Request for Comments)**
- Proposal: tulis RFC document
- Review: team review dan discuss
- Approval: Owner + Tech Lead approve
- Implementation: update master documents + code
- Communication: notify all stakeholders

**Code Review**
- Semua PR harus di-review
- Checklist: contracts compliance, tests, accessibility, performance
- Approvals: minimal 1 reviewer + tech lead untuk critical changes
- Merge: squash merge untuk clean history

**Definition of Done (DoD)**
- Code implemented sesuai contracts
- Unit tests: ≥80% coverage
- Integration tests: critical paths covered
- E2E tests: happy paths covered
- Accessibility: axe-core pass
- Performance: Lighthouse ≥90
- Documentation: updated
- Code review: approved
- CI/CD: green

### **Versioning**

**Semantic Versioning**
- Major: breaking changes (API, database schema)
- Minor: new features (backward compatible)
- Patch: bug fixes (backward compatible)

**Database Migrations**
- Sequential numbering (001, 002, 003, ...)
- Up + down migrations (rollback capable)
- Test migrations di local sebelum deploy
- Backup database sebelum migration

**API Versioning**
- URL-based: `/api/v1/`, `/api/v2/`
- Deprecation policy: 6 months notice
- Sunset headers: indicate deprecation
- Migration guide: untuk breaking changes

---

## **XI. KEAMANAN & COMPLIANCE**

### **Data Encryption**
- **In transit**: SSL/TLS 1.3 (HTTPS only)
- **At rest**: Supabase encrypted storage (AES-256)
- **Sensitive fields**: PIN hash (bcrypt), API keys (encrypted)
- **Backup**: encrypted backups, retained 30 days

### **RLS Enforcement**
- Database-level row security
- Policies per table per role
- No bypass allowed (service role only untuk Edge Functions)
- Regular audit: check RLS policies

### **Rate Limiting**
- Login: 5 attempts per 15 minutes per IP
- API: 100 requests per minute per user
- Upload: 10 uploads per minute per user
- Prevent brute force dan DDoS

### **Input Validation**
- Frontend: Zod schemas, instant feedback
- Backend: Edge Functions validate sebelum DB
- Database: check constraints, types
- Defense in depth: validate di semua layer

### **SQL Injection Prevention**
- Parameterized queries (Supabase client)
- No raw SQL (kecuali migrations)
- Input sanitization (escape special characters)
- Regular security audits

### **CSRF Protection**
- SameSite cookies (strict)
- CSRF tokens untuk state-changing operations
- Supabase Auth built-in protection
- Regular security testing

### **XSS Prevention**
- React auto-escapes (no dangerouslySetInnerHTML)
- Content Security Policy (CSP) headers
- Sanitize user inputs (DOMPurify jika perlu)
- Regular security audits

### **HIPAA-Adjacent (Medical Records)**
- Access logging: semua access ke medical records logged
- Encryption: medical records encrypted at rest
- Access control: hanya dokter + customer bisa view
- Audit trail: immutable, 2 tahun retention
- Data minimization: hanya collect necessary data

### **GDPR-Ready (Future)**
- Right to access: export customer data
- Right to deletion: hard delete customer data
- Data portability: export dalam standard format
- Consent management: track consent per customer
- Privacy policy: transparent data usage

### **Security Monitoring**
- Sentry: error tracking, security alerts
- Supabase Analytics: suspicious activity detection
- Regular security audits (quarterly)
- Penetration testing (annual)
- Bug bounty program (future)

### **Compliance Checklist**
- ✅ Data encryption (in transit + at rest)
- ✅ Access control (RBAC + RLS)
- ✅ Audit logging (immutable, 2 tahun)
- ✅ Input validation (all layers)
- ✅ SQL injection prevention
- ✅ XSS prevention
- ✅ CSRF protection
- ✅ Rate limiting
- ✅ Session management (secure cookies)
- ✅ PIN hashing (bcrypt)
- ✅ API key encryption
- ✅ Backup & recovery
- ✅ Incident response plan
- ⏳ HIPAA compliance (if needed)
- ⏳ GDPR compliance (if needed)
- ⏳ PCI DSS compliance (for payments)

---

## **XII. DEPLOYMENT & OPERATIONS**

### **Frontend Deployment (Vercel)**
- Auto-deploy dari GitHub (main branch)
- Preview deployments untuk PRs
- Custom domain: app.petora.id
- CDN: global edge network
- SSL: automatic
- Environment variables: per environment (dev, staging, prod)

### **Backend Deployment (Supabase Cloud)**
- Managed PostgreSQL (auto-backup, auto-scaling)
- Edge Functions: auto-deploy dari GitHub
- Storage: S3-compatible, auto-scaling
- Realtime: WebSocket connections
- Auth: managed service
- Region: Singapore (closest to Indonesia)

### **Environment Management**
- **Development**: local Supabase, Vite dev server
- **Staging**: Supabase Cloud (staging project), Vercel preview
- **Production**: Supabase Cloud (prod project), Vercel main branch
- Environment variables: separate per environment
- Database seeds: different per environment

### **Monitoring & Alerting**
- **Sentry**: error tracking, performance monitoring
- **Supabase Analytics**: database performance, slow queries
- **Vercel Analytics**: frontend performance, web vitals
- **Uptime monitoring**: Pingdom / UptimeRobot
- **Alerting**: Slack / Email untuk critical issues

### **Backup & Recovery**
- Database: automatic daily backups (Supabase)
- Files: Supabase Storage (redundant)
- Code: GitHub (version control)
- Recovery: point-in-time recovery (Supabase)
- RTO: 1 jam, RPO: 24 jam

### **Scaling**
- Frontend: Vercel auto-scaling (serverless)
- Backend: Supabase auto-scaling (managed)
- Database: read replicas (jika needed)
- CDN: global edge network
- Realtime: WebSocket scaling (Supabase)

### **Incident Response**
- **Severity levels**: P1 (critical), P2 (high), P3 (medium), P4 (low)
- **Response time**: P1: 15 menit, P2: 1 jam, P3: 4 jam, P4: 24 jam
- **Communication**: Slack war room, status page, customer notification
- **Post-mortem**: required untuk P1 dan P2
- **Runbooks**: documented untuk common issues

---

Dokumen spesifikasi **PETORA** ini telah mencakup seluruh aspek sistem secara komprehensif, dari arsitektur, stack teknologi, modul-modul bisnis, fitur cross-cutting, database schema, hingga deployment dan operations. Setiap bagian dirancang untuk memberikan kejelasan maksimal tanpa ambiguitas, dengan fokus pada detail fungsional dan business rules yang diperlukan untuk implementasi.
