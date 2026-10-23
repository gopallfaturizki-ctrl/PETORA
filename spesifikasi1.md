# 🐾 Rancangan Lengkap Sistem Pet Shop Enterprise
### Vercel + Supabase | Multi-Role | Single Database

---

## 📋 DAFTAR MODUL LENGKAP (15 Modul Utama)

| No | Modul | Sub-Modul |
|----|-------|-----------|
| 1 | **POS & Kasir** | Transaksi, Retur, Void, Split Payment, Held Orders |
| 2 | **Inventory & Warehouse** | Stok, Stock Opname, Transfer, Batch/Expired, Valuation |
| 3 | **Produk & Katalog** | Master produk, kategori, barcode, harga, variant |
| 4 | **Layanan Grooming** | Booking, workflow groomer, foto before/after |
| 5 | **Pet Hotel** | Booking kamar, check-in/out, daily care log |
| 6 | **Klinik & Rekam Medis** | Appointment, EMR, resep, vaksinasi, lab |
| 7 | **CRM & Customer** | Profil, pet, riwayat, segmentasi, loyalty |
| 8 | **Purchase & Supplier** | PO, GRN, hutang supplier, price list |
| 9 | **Accounting & Finance** | Jurnal, laba-rugi, kas, piutang, pajak |
| 10 | **HRM & Payroll** | Karyawan, absensi, gaji, komisi |
| 11 | **Marketing & Promo** | Voucher, diskon, membership, campaign |
| 12 | **Monitoring & BI** | Dashboard KPI, laporan, forecasting |
| 13 | **Notification Center** | WhatsApp, email, in-app, reminder |
| 14 | **Settings & Configuration** | Outlet, pajak, printer, backup |
| 15 | **Audit & Security** | Log, permission, session, PIN policy |

---

## 1. 🏗️ ARSITEKTUR SISTEM (ENTERPRISE)

```
┌──────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                               │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐   │
│  │ Admin Web  │ │ Kasir POS  │ │ Dokter Web │ │ Customer   │   │
│  │ (Desktop)  │ │ (Tablet)   │ │ (Desktop)  │ │ (Mobile)   │   │
│  └────────────┘ └────────────┘ └────────────┘ └────────────┘   │
│               Next.js 14 + Tailwind + shadcn/ui                  │
│               Deploy: Vercel (Edge + Serverless)                 │
└─────────────────────────────┬────────────────────────────────────┘
                              │ HTTPS + WSS
┌─────────────────────────────▼────────────────────────────────────┐
│                      SUPABASE BACKEND                             │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │ PostgreSQL   │ │ Auth (custom)│ │ Realtime     │            │
│  │ (150+ tables)│ │ username+PIN │ │ (POS, Hotel) │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐            │
│  │ Storage      │ │ Edge Funcs   │ │ Cron Jobs    │            │
│  │ (media,docs) │ │ (WA,invoice) │ │ (reminder)   │            │
│  └──────────────┘ └──────────────┘ └──────────────┘            │
│  ┌──────────────┐ ┌──────────────┐                              │
│  │ RLS Policies │ │ RPC Functions│                              │
│  │ (per role)   │ │ (business)   │                              │
│  └──────────────┘ └──────────────┘                              │
└──────────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────▼────────────────────────────────────┐
│                    EXTERNAL INTEGRATIONS                          │
│  • WhatsApp (Fonnte/Wablas)  • Printer Thermal (WebUSB)          │
│  • Payment Gateway (Midtrans) • Barcode Scanner                  │
│  • Email (Resend)              • Accounting (export)             │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. 🗄️ DATABASE SCHEMA LENGKAP

### 2.1 Core & Authentication

```sql
-- ============ ENUMS ============
create type user_role as enum ('owner','admin','dokter','customer','groomer','kasir');
create type gender_type as enum ('M','F');
create type species_type as enum ('dog','cat','rabbit','bird','fish','hamster','reptile','other');
create type payment_method as enum ('cash','qris','transfer','ewallet','debit','credit','loyalty');
create type transaction_status as enum ('completed','void','refunded','pending','held');
create type appointment_status as enum ('pending','confirmed','in_progress','done','cancelled','no_show');
create type booking_status as enum ('booked','checked_in','checked_out','cancelled','no_show');
create type stock_movement_type as enum ('in','out','adjustment','return','damage','transfer','expired');
create type po_status as enum ('draft','ordered','partial','received','cancelled');
create type service_type as enum ('grooming','hotel','clinic','vaksin','steril','lab','obat');

-- ============ OUTLETS (Multi-cabang support) ============
create table outlets (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,
  name text not null,
  address text,
  phone text,
  email text,
  is_default boolean default false,
  is_active boolean default true,
  tax_number text,
  created_at timestamptz default now()
);

-- ============ USERS & PROFILES ============
create table profiles (
  id uuid primary key references auth.users(id) on delete cascade,
  outlet_id uuid references outlets(id),
  username text unique not null,
  pin_hash text not null,
  pin_salt text not null,
  pin_attempts int default 0,
  pin_locked_until timestamptz,
  full_name text not null,
  role user_role not null,
  phone text,
  email text,
  address text,
  nik text,
  birth_date date,
  join_date date,
  salary_base numeric(12,2),
  commission_rate numeric(5,2) default 0,  -- % komisi untuk kasir/dokter
  avatar_url text,
  signature_url text,                      -- tanda tangan digital
  is_active boolean default true,
  last_login timestamptz,
  created_at timestamptz default now()
);

-- ============ PERMISSIONS (Granular) ============
create table permissions (
  id uuid primary key default gen_random_uuid(),
  role user_role not null,
  module text not null,                    -- 'pos','inventory','medical', dll
  action text not null,                    -- 'create','read','update','delete','void','approve'
  outlet_id uuid references outlets(id),   -- null = semua outlet
  unique(role, module, action, outlet_id)
);
```

### 2.2 Produk & Inventory

```sql
-- ============ KATEGORI ============
create table categories (
  id uuid primary key default gen_random_uuid(),
  parent_id uuid references categories(id),
  name text not null,
  code text unique,
  type text check (type in ('food','accessory','medicine','service','other')),
  description text,
  image_url text,
  is_active boolean default true
);

-- ============ BRAND ============
create table brands (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  country text,
  logo_url text
);

-- ============ PRODUK ============
create table products (
  id uuid primary key default gen_random_uuid(),
  sku text unique not null,
  barcode text,
  name text not null,
  category_id uuid references categories(id),
  brand_id uuid references brands(id),
  unit text not null,                      -- pcs, kg, gram, box
  unit_conversion numeric(10,3) default 1, -- 1 box = 12 pcs
  variant text,                            -- rasa, ukuran, warna
  image_url text,
  description text,
  
  -- Harga
  buy_price numeric(12,2) default 0,
  sell_price numeric(12,2) not null,
  wholesale_price numeric(12,2),           -- harga grosir
  member_price numeric(12,2),              -- harga member
  
  -- Stok
  stock numeric(12,3) default 0,
  min_stock numeric(12,3) default 5,
  max_stock numeric(12,3),
  
  -- Batch & Expired
  has_batch boolean default false,
  has_expired boolean default false,
  
  -- Pajak
  tax_id uuid,                             -- referensi ke tax_rate
  is_taxable boolean default true,
  
  -- Status
  is_active boolean default true,
  is_service boolean default false,
  
  -- Metadata
  weight_gram numeric(10,2),               -- untuk kalkulasi ongkir
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- ============ BATCH & EXPIRED (FEFO) ============
create table product_batches (
  id uuid primary key default gen_random_uuid(),
  product_id uuid references products(id) on delete cascade,
  batch_number text,
  expired_date date,
  qty_initial numeric(12,3),
  qty_remaining numeric(12,3),
  buy_price numeric(12,2),                 -- HPP per batch (FIFO)
  supplier_id uuid,
  received_date date,
  notes text
);

-- ============ STOCK MOVEMENT ============
create table stock_movements (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),
  product_id uuid references products(id),
  batch_id uuid references product_batches(id),
  type stock_movement_type not null,
  qty numeric(12,3) not null,
  before_stock numeric(12,3),
  after_stock numeric(12,3),
  reference_type text,                     -- 'sale','purchase','adjustment','transfer'
  reference_id uuid,
  reference_no text,
  note text,
  user_id uuid references profiles(id),
  created_at timestamptz default now()
);

-- ============ WAREHOUSE / LOKASI STOK ============
create table warehouses (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),
  name text not null,                      -- 'Gudang Utama','Display','Rusak'
  code text,
  is_main boolean default false
);

create table warehouse_stocks (
  id uuid primary key default gen_random_uuid(),
  warehouse_id uuid references warehouses(id),
  product_id uuid references products(id),
  qty numeric(12,3) default 0,
  unique(warehouse_id, product_id)
);

-- ============ SUPPLIER ============
create table suppliers (
  id uuid primary key default gen_random_uuid(),
  code text unique,
  name text not null,
  contact_person text,
  phone text,
  email text,
  address text,
  tax_number text,
  bank_name text,
  bank_account text,
  bank_holder text,
  payment_term_days int default 0,         -- tempo pembayaran
  outstanding_balance numeric(14,2) default 0,
  is_active boolean default true,
  created_at timestamptz default now()
);

-- ============ PURCHASE ORDER ============
create table purchase_orders (
  id uuid primary key default gen_random_uuid(),
  po_number text unique not null,
  outlet_id uuid references outlets(id),
  supplier_id uuid references suppliers(id),
  order_date date not null,
  expected_date date,
  received_date date,
  subtotal numeric(14,2),
  discount numeric(12,2) default 0,
  tax numeric(12,2) default 0,
  total numeric(14,2),
  paid_amount numeric(14,2) default 0,
  status po_status default 'draft',
  payment_status text default 'unpaid',    -- unpaid, partial, paid
  notes text,
  created_by uuid references profiles(id),
  approved_by uuid references profiles(id),
  created_at timestamptz default now()
);

create table purchase_order_items (
  id uuid primary key default gen_random_uuid(),
  po_id uuid references purchase_orders(id) on delete cascade,
  product_id uuid references products(id),
  qty_ordered numeric(12,3),
  qty_received numeric(12,3) default 0,
  buy_price numeric(12,2),
  discount numeric(12,2) default 0,
  subtotal numeric(12,2),
  batch_number text,
  expired_date date
);

-- ============ STOCK OPNAME ============
create table stock_opnames (
  id uuid primary key default gen_random_uuid(),
  opname_number text unique,
  outlet_id uuid references outlets(id),
  opname_date date,
  status text default 'draft',             -- draft, in_progress, finalized
  total_items int,
  matched_items int,
  discrepancy_items int,
  notes text,
  created_by uuid references profiles(id),
  finalized_by uuid references profiles(id),
  finalized_at timestamptz
);

create table stock_opname_items (
  id uuid primary key default gen_random_uuid(),
  opname_id uuid references stock_opnames(id) on delete cascade,
  product_id uuid references products(id),
  system_qty numeric(12,3),
  physical_qty numeric(12,3),
  difference numeric(12,3),
  note text
);
```

### 2.3 Customer, Pet & Loyalty

```sql
-- ============ CUSTOMER ============
create table customers (
  id uuid primary key default gen_random_uuid(),
  profile_id uuid references profiles(id), -- null jika walk-in
  code text unique,
  name text not null,
  phone text,
  email text,
  whatsapp text,
  address text,
  city text,
  birth_date date,
  gender gender_type,
  
  -- Membership
  member_type text default 'regular',      -- regular, silver, gold, platinum
  member_since date,
  member_expired date,
  
  -- Loyalty
  loyalty_points int default 0,
  total_spent numeric(14,2) default 0,
  total_visits int default 0,
  
  -- Preferences
  preferred_vet uuid references profiles(id),
  notes text,
  referral_code text unique,
  referred_by uuid references customers(id),
  
  is_active boolean default true,
  created_at timestamptz default now()
);

-- ============ PET ============
create table pets (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references customers(id) on delete cascade,
  code text unique,
  name text not null,
  species species_type not null,
  breed text,
  gender gender_type,
  birth_date date,
  color text,
  microchip_number text,
  weight_kg numeric(5,2),
  photo_url text,
  
  -- Kesehatan
  is_neutered boolean default false,
  allergies text,                          -- alergi makanan/obat
  chronic_conditions text,                 -- penyakit kronis
  blood_type text,
  last_vaccine_date date,
  next_vaccine_date date,
  
  -- Preferensi
  preferred_groomer uuid references profiles(id),
  preferred_doctor uuid references profiles(id),
  grooming_notes text,                     -- "jangan dicukur habis", dll
  feeding_notes text,                      -- untuk hotel
  
  is_active boolean default true,
  created_at timestamptz default now()
);

-- ============ LOYALTY TIERS ============
create table loyalty_tiers (
  id uuid primary key default gen_random_uuid(),
  name text not null,                      -- Regular, Silver, Gold, Platinum
  min_points int,
  min_spent numeric(14,2),
  point_multiplier numeric(3,2) default 1, -- 1x, 1.5x, 2x
  discount_percent numeric(5,2) default 0,
  benefits jsonb                           -- list benefit
);

-- ============ POINTS HISTORY ============
create table loyalty_transactions (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references customers(id),
  type text check (type in ('earn','redeem','expire','adjust')),
  points int not null,
  balance_after int,
  reference_type text,                     -- 'transaction','redeem','adjustment'
  reference_id uuid,
  notes text,
  created_at timestamptz default now()
);
```

### 2.4 POS & Transaksi

```sql
-- ============ TRANSAKSI ============
create table transactions (
  id uuid primary key default gen_random_uuid(),
  invoice_no text unique not null,
  outlet_id uuid references outlets(id),
  customer_id uuid references customers(id),
  cashier_id uuid references profiles(id),
  
  -- Item
  subtotal numeric(14,2) not null,
  item_discount numeric(12,2) default 0,
  
  -- Diskon
  discount_type text,                      -- percent, nominal
  discount_value numeric(12,2) default 0,
  discount_amount numeric(12,2) default 0,
  voucher_id uuid,
  
  -- Pajak
  tax_rate numeric(5,2) default 0,
  tax_amount numeric(12,2) default 0,
  
  -- Total
  service_charge numeric(12,2) default 0,
  rounding numeric(12,2) default 0,
  grand_total numeric(14,2) not null,
  
  -- Pembayaran
  paid_amount numeric(14,2) not null,
  change_amount numeric(12,2) default 0,
  
  -- Loyalty
  points_earned int default 0,
  points_redeemed int default 0,
  points_discount numeric(12,2) default 0,
  
  -- Status
  status transaction_status default 'completed',
  void_reason text,
  void_by uuid references profiles(id),
  void_at timestamptz,
  
  transaction_date timestamptz default now(),
  business_date date default current_date
);

-- ============ PAYMENT DETAILS (split payment) ============
create table transaction_payments (
  id uuid primary key default gen_random_uuid(),
  transaction_id uuid references transactions(id) on delete cascade,
  method payment_method not null,
  amount numeric(12,2) not null,
  reference_no text,                       -- no. kartu, no. transfer, dll
  card_number text,                        -- 4 digit terakhir
  gateway_response jsonb
);

-- ============ TRANSACTION ITEMS ============
create table transaction_items (
  id uuid primary key default gen_random_uuid(),
  transaction_id uuid references transactions(id) on delete cascade,
  product_id uuid references products(id),
  batch_id uuid references product_batches(id),
  name_snapshot text not null,             -- backup nama jika produk dihapus
  sku_snapshot text,
  qty numeric(10,3) not null,
  unit text,
  price numeric(12,2) not null,
  discount_percent numeric(5,2) default 0,
  discount_amount numeric(12,2) default 0,
  tax_percent numeric(5,2) default 0,
  tax_amount numeric(12,2) default 0,
  subtotal numeric(12,2) not null,
  is_service boolean default false,
  notes text
);

-- ============ HELD TRANSACTIONS (simpan sementara) ============
create table held_transactions (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),
  cashier_id uuid references profiles(id),
  customer_id uuid references customers(id),
  items jsonb not null,                    -- array of cart items
  notes text,
  held_at timestamptz default now(),
  expires_at timestamptz                   -- auto-expire setelah 24 jam
);

-- ============ RETUR ============
create table returns (
  id uuid primary key default gen_random_uuid(),
  return_number text unique,
  original_transaction_id uuid references transactions(id),
  customer_id uuid references customers(id),
  outlet_id uuid references outlets(id),
  return_date timestamptz default now(),
  reason text,
  refund_method payment_method,
  total_refund numeric(12,2),
  status text default 'pending',           -- pending, approved, completed
  approved_by uuid references profiles(id),
  created_by uuid references profiles(id)
);

create table return_items (
  id uuid primary key default gen_random_uuid(),
  return_id uuid references returns(id) on delete cascade,
  product_id uuid references products(id),
  qty numeric(10,3),
  price numeric(12,2),
  subtotal numeric(12,2),
  condition text check (condition in ('good','damaged','expired')),
  note text
);
```

### 2.5 Layanan: Grooming, Hotel, Klinik

```sql
-- ============ SERVICE PACKAGES ============
create table services (
  id uuid primary key default gen_random_uuid(),
  code text unique,
  name text not null,
  type service_type not null,
  species_allowed species_type[],          -- ['dog','cat']
  size_categories text[],                  -- ['small','medium','large','xlarge']
  base_price numeric(12,2) not null,
  price_by_weight boolean default false,
  price_per_kg numeric(10,2),
  duration_minutes int,
  description text,
  includes jsonb,                          -- list included treatments
  is_active boolean default true
);

-- ============ GROOMING ============
create table grooming_bookings (
  id uuid primary key default gen_random_uuid(),
  booking_no text unique,
  outlet_id uuid references outlets(id),
  pet_id uuid references pets(id),
  customer_id uuid references customers(id),
  service_id uuid references services(id),
  groomer_id uuid references profiles(id),
  
  scheduled_at timestamptz not null,
  duration_minutes int,
  actual_start timestamptz,
  actual_end timestamptz,
  
  status appointment_status default 'pending',
  notes text,
  special_request text,
  
  -- Hasil
  weight_before numeric(5,2),
  weight_after numeric(5,2),
  skin_condition text,
  fur_condition text,
  findings text,                           -- kutu, jamur, dll
  treatments_done jsonb,                   -- list treatment aktual
  photo_before_url text,
  photo_after_url text,
  
  -- Biaya
  base_price numeric(12,2),
  additional_charges numeric(12,2) default 0,
  discount numeric(12,2) default 0,
  total numeric(12,2),
  transaction_id uuid references transactions(id),
  
  created_at timestamptz default now()
);

-- ============ PET HOTEL ============
create table hotel_rooms (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),
  code text unique not null,               -- 'A01', 'VIP-01'
  name text,
  type text,                               -- small, medium, large, vip, suite
  capacity int default 1,
  price_per_day numeric(12,2) not null,
  facilities text[],
  photo_url text,
  status text default 'available',         -- available, occupied, maintenance
  notes text
);

create table hotel_bookings (
  id uuid primary key default gen_random_uuid(),
  booking_no text unique,
  outlet_id uuid references outlets(id),
  room_id uuid references hotel_rooms(id),
  pet_id uuid references pets(id),
  customer_id uuid references customers(id),
  
  check_in_date date not null,
  check_out_date date not null,
  actual_check_in timestamptz,
  actual_check_out timestamptz,
  
  price_per_day numeric(12,2),
  total_days int,
  subtotal numeric(12,2),
  additional_services numeric(12,2) default 0,
  discount numeric(12,2) default 0,
  total numeric(12,2),
  
  status booking_status default 'booked',
  
  -- Info khusus
  feeding_schedule text,                   -- "2x sehari, pagi & malam"
  food_brought boolean default false,      -- bawa makanan sendiri?
  food_brand text,
  medicine_schedule text,
  special_notes text,
  emergency_contact text,
  vet_contact text,
  
  deposit_amount numeric(12,2) default 0,
  deposit_status text default 'unpaid',
  
  transaction_id uuid references transactions(id),
  created_at timestamptz default now()
);

-- ============ DAILY CARE LOG (Hotel) ============
create table hotel_daily_logs (
  id uuid primary key default gen_random_uuid(),
  booking_id uuid references hotel_bookings(id) on delete cascade,
  log_date date not null,
  logged_by uuid references profiles(id),
  
  -- Kondisi
  appetite text check (appetite in ('good','fair','poor','refused')),
  stool text check (stool in ('normal','soft','hard','diarrhea','none')),
  activity text check (activity in ('active','normal','lethargic')),
  mood text,
  weight_kg numeric(5,2),
  temperature numeric(4,1),
  
  -- Perawatan
  feeding_done boolean default false,
  feeding_notes text,
  walk_done boolean default false,
  walk_duration_minutes int,
  bath_done boolean default false,
  medicine_given boolean default false,
  medicine_notes text,
  
  -- Foto & video
  photos text[],
  video_url text,
  
  notes text,
  reported_to_owner boolean default false,
  created_at timestamptz default now()
);

-- ============ CLINIC / APPOINTMENT ============
create table appointments (
  id uuid primary key default gen_random_uuid(),
  appointment_no text unique,
  outlet_id uuid references outlets(id),
  pet_id uuid references pets(id),
  customer_id uuid references customers(id),
  doctor_id uuid references profiles(id),
  
  scheduled_at timestamptz not null,
  duration_minutes int default 30,
  actual_start timestamptz,
  actual_end timestamptz,
  
  type text check (type in ('consultation','vaccine','checkup','surgery','emergency','followup')),
  complaint text,                          -- keluhan awal
  status appointment_status default 'pending',
  notes text,
  
  -- Biaya
  consultation_fee numeric(12,2),
  total numeric(12,2),
  transaction_id uuid references transactions(id),
  
  created_at timestamptz default now()
);

-- ============ MEDICAL RECORDS (EMR) ============
create table medical_records (
  id uuid primary key default gen_random_uuid(),
  appointment_id uuid references appointments(id),
  pet_id uuid references pets(id),
  doctor_id uuid references profiles(id),
  record_date timestamptz default now(),
  
  -- Anamnesa
  chief_complaint text,
  history_of_illness text,
  previous_treatment text,
  
  -- Pemeriksaan
  weight_kg numeric(5,2),
  temperature numeric(4,1),
  heart_rate int,
  respiratory_rate int,
  mucous_membrane text,
  capillary_refill_time text,
  body_condition_score numeric(3,1),
  physical_exam jsonb,                     -- detail per organ
  
  -- Diagnosa
  diagnosis text,
  diagnosis_code text,                     -- ICD-10 veterinary
  differential_diagnosis text,
  
  -- Tindakan
  treatment text,
  procedures_done jsonb,                   -- list tindakan
  surgery_notes text,
  
  -- Resep
  prescriptions jsonb,                     -- [{obat, dosis, durasi, instruksi}]
  
  -- Hasil Lab
  lab_results jsonb,
  
  -- Rekomendasi
  recommendations text,
  next_control date,
  
  -- Lampiran
  attachments text[],                      -- URL foto, hasil lab, rontgen
  
  created_at timestamptz default now()
);

-- ============ VACCINATION ============
create table vaccinations (
  id uuid primary key default gen_random_uuid(),
  pet_id uuid references pets(id),
  medical_record_id uuid references medical_records(id),
  doctor_id uuid references profiles(id),
  
  vaccine_name text not null,
  vaccine_brand text,
  batch_number text,
  expired_date date,
  dose_number int,                         -- 1, 2, 3 (booster)
  dose_ml numeric(4,2),
  injection_site text,                     -- SC left, IM right, dll
  
  vaccinated_at timestamptz default now(),
  next_due_date date,
  certificate_url text,
  
  reaction_notes text,                     -- reaksi pasca vaksin
  created_at timestamptz default now()
);

-- ============ PRESCRIPTION ============
create table prescriptions (
  id uuid primary key default gen_random_uuid(),
  medical_record_id uuid references medical_records(id),
  prescription_no text unique,
  pet_id uuid references pets(id),
  doctor_id uuid references profiles(id),
  issued_date timestamptz default now(),
  
  items jsonb not null,                    -- [{product_id, dosis, frekuensi, durasi, instruksi}]
  instructions text,
  notes text,
  
  is_fulfilled boolean default false,
  fulfilled_at timestamptz,
  transaction_id uuid references transactions(id)
);
```

### 2.6 Accounting & Finance

```sql
-- ============ CHART OF ACCOUNTS ============
create table accounts (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,               -- 1-1000 Asset, 2-2000 Liability, dst
  name text not null,
  type text check (type in ('asset','liability','equity','revenue','expense','cogs')),
  parent_id uuid references accounts(id),
  is_postable boolean default true,        -- bisa di-journal-kan
  normal_balance text check (normal_balance in ('debit','credit')),
  is_active boolean default true
);

-- ============ JOURNAL ENTRIES ============
create table journal_entries (
  id uuid primary key default gen_random_uuid(),
  entry_no text unique,
  entry_date date not null,
  description text,
  reference_type text,                     -- 'sale','purchase','adjustment','manual'
  reference_id uuid,
  total_debit numeric(14,2),
  total_credit numeric(14,2),
  status text default 'posted',            -- draft, posted, void
  created_by uuid references profiles(id),
  posted_at timestamptz default now()
);

create table journal_items (
  id uuid primary key default gen_random_uuid(),
  entry_id uuid references journal_entries(id) on delete cascade,
  account_id uuid references accounts(id),
  description text,
  debit numeric(14,2) default 0,
  credit numeric(14,2) default 0
);

-- ============ KAS / CASH REGISTER ============
create table cash_registers (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),
  name text,
  current_balance numeric(14,2) default 0,
  is_active boolean default true
);

-- ============ CASH CLOSING (Setor Kas Harian) ============
create table cash_closings (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),
  cash_register_id uuid references cash_registers(id),
  closing_date date not null,
  opened_by uuid references profiles(id),
  closed_by uuid references profiles(id),
  
  opening_balance numeric(14,2),
  cash_in numeric(14,2),
  cash_out numeric(14,2),
  expected_balance numeric(14,2),
  actual_balance numeric(14,2),
  difference numeric(14,2),
  
  transactions_count int,
  notes text,
  status text default 'open',              -- open, closed, audited
  closed_at timestamptz
);

-- ============ EXPENSES (Pengeluaran Operasional) ============
create table expenses (
  id uuid primary key default gen_random_uuid(),
  expense_no text unique,
  outlet_id uuid references outlets(id),
  category text,                           -- listrik, air, gaji, sewa, dll
  account_id uuid references accounts(id),
  amount numeric(14,2) not null,
  expense_date date not null,
  description text,
  receipt_url text,
  paid_via payment_method,
  status text default 'paid',              -- pending, paid, reimbursed
  approved_by uuid references profiles(id),
  created_by uuid references profiles(id),
  created_at timestamptz default now()
);

-- ============ PIUTANG (Customer Debt) ============
create table receivables (
  id uuid primary key default gen_random_uuid(),
  customer_id uuid references customers(id),
  transaction_id uuid references transactions(id),
  due_date date,
  total_amount numeric(14,2),
  paid_amount numeric(14,2) default 0,
  remaining numeric(14,2),
  status text default 'open'               -- open, partial, paid, overdue
);

-- ============ HUTANG (Supplier Debt) ============
create table payables (
  id uuid primary key default gen_random_uuid(),
  supplier_id uuid references suppliers(id),
  po_id uuid references purchase_orders(id),
  due_date date,
  total_amount numeric(14,2),
  paid_amount numeric(14,2) default 0,
  remaining numeric(14,2),
  status text default 'open'
);
```

### 2.7 HRM & Payroll

```sql
-- ============ ATTENDANCE ============
create table attendance (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  outlet_id uuid references outlets(id),
  date date not null,
  clock_in timestamptz,
  clock_out timestamptz,
  work_hours numeric(4,2),
  overtime_hours numeric(4,2) default 0,
  status text default 'present',           -- present, late, absent, leave, sick
  late_minutes int default 0,
  notes text,
  unique(user_id, date)
);

-- ============ PAYROLL ============
create table payrolls (
  id uuid primary key default gen_random_uuid(),
  payroll_no text unique,
  period_month int,
  period_year int,
  user_id uuid references profiles(id),
  
  -- Income
  base_salary numeric(12,2),
  overtime_pay numeric(12,2) default 0,
  commission numeric(12,2) default 0,
  bonus numeric(12,2) default 0,
  allowance numeric(12,2) default 0,
  
  -- Deductions
  tax numeric(12,2) default 0,
  bpjs numeric(12,2) default 0,
  late_deduction numeric(12,2) default 0,
  other_deduction numeric(12,2) default 0,
  
  -- Total
  gross numeric(12,2),
  net numeric(12,2),
  
  status text default 'draft',             -- draft, approved, paid
  paid_at timestamptz,
  approved_by uuid references profiles(id),
  unique(user_id, period_month, period_year)
);

-- ============ COMMISSION LOG ============
create table commissions (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  type text,                               -- 'sale','service','grooming'
  reference_type text,
  reference_id uuid,
  base_amount numeric(12,2),
  rate numeric(5,2),
  commission_amount numeric(12,2),
  period_month int,
  period_year int,
  created_at timestamptz default now()
);
```

### 2.8 Marketing & Promo

```sql
-- ============ VOUCHERS ============
create table vouchers (
  id uuid primary key default gen_random_uuid(),
  code text unique not null,
  name text,
  type text check (type in ('percent','nominal','free_item','bogo')),
  value numeric(12,2),
  min_purchase numeric(12,2) default 0,
  max_discount numeric(12,2),
  applicable_to text default 'all',        -- all, product, category, service
  applicable_ids uuid[],
  
  quota int,
  used_count int default 0,
  per_user_limit int default 1,
  
  start_date timestamptz,
  end_date timestamptz,
  
  is_active boolean default true,
  created_at timestamptz default now()
);

create table voucher_usages (
  id uuid primary key default gen_random_uuid(),
  voucher_id uuid references vouchers(id),
  customer_id uuid references customers(id),
  transaction_id uuid references transactions(id),
  discount_amount numeric(12,2),
  used_at timestamptz default now()
);

-- ============ PROMOTIONS ============
create table promotions (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  type text check (type in ('discount','bundle','buy_get','cashback','flash_sale')),
  config jsonb not null,                   -- detail rule promo
  applicable_products uuid[],
  applicable_categories uuid[],
  start_date timestamptz,
  end_date timestamptz,
  is_active boolean default true
);

-- ============ CAMPAIGNS ============
create table campaigns (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  channel text check (channel in ('whatsapp','email','sms','in_app')),
  target_audience jsonb,                   -- segmentasi customer
  message_template text,
  scheduled_at timestamptz,
  sent_count int default 0,
  opened_count int default 0,
  converted_count int default 0,
  status text default 'draft'
);
```

### 2.9 Settings & System

```sql
-- ============ TAX RATES ============
create table tax_rates (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  rate numeric(5,2) not null,
  is_default boolean default false,
  is_active boolean default true
);

-- ============ AUDIT LOG ============
create table audit_logs (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  outlet_id uuid references outlets(id),
  action text not null,                    -- 'create','update','delete','void','login','logout'
  module text not null,
  record_type text,
  record_id uuid,
  old_values jsonb,
  new_values jsonb,
  ip_address text,
  user_agent text,
  created_at timestamptz default now()
);

-- ============ SYSTEM SETTINGS ============
create table settings (
  id uuid primary key default gen_random_uuid(),
  outlet_id uuid references outlets(id),   -- null = global
  key text not null,
  value jsonb not null,
  unique(outlet_id, key)
);

-- ============ NOTIFICATIONS ============
create table notifications (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  title text,
  message text,
  type text,                               -- low_stock, appointment, payment, system
  priority text default 'normal',          -- low, normal, high, urgent
  reference_type text,
  reference_id uuid,
  is_read boolean default false,
  read_at timestamptz,
  created_at timestamptz default now()
);

-- ============ SEQUENCES (Auto-numbering) ============
create table sequences (
  prefix text primary key,                 -- 'INV','PO','SO','APPT','HOTEL', dll
  last_number int default 0,
  format text                              -- 'INV/{yyyy}{mm}/{nnnnn}'
);
```

---

## 3. 🔐 ROW LEVEL SECURITY (RLS) — DETAIL

```sql
-- Helper functions
create or replace function current_user_id() returns uuid as $$
  select auth.uid();
$$ language sql security definer;

create or replace function current_user_role() returns user_role as $$
  select role from profiles where id = auth.uid();
$$ language sql security definer;

create or replace function current_user_outlet() returns uuid as $$
  select outlet_id from profiles where id = auth.uid();
$$ language sql security definer;

-- ============ TRANSACTIONS ============
alter table transactions enable row level security;

-- Owner: lihat semua outlet
create policy "owner_all_transactions"
on transactions for all
using (current_user_role() = 'owner');

-- Admin/Kasir: hanya outlet sendiri
create policy "staff_own_outlet_transactions"
on transactions for all
using (
  current_user_role() in ('admin','kasir')
  and outlet_id = current_user_outlet()
);

-- Dokter: hanya lihat transaksi terkait appointment
create policy "dokter_medical_transactions"
on transactions for select
using (
  current_user_role() = 'dokter'
  and id in (select transaction_id from appointments where doctor_id = current_user_id())
);

-- Customer: hanya transaksi sendiri
create policy "customer_own_transactions"
on transactions for select
using (
  current_user_role() = 'customer'
  and customer_id in (
    select id from customers where profile_id = current_user_id()
  )
);

-- ============ MEDICAL RECORDS ============
alter table medical_records enable row level security;

create policy "owner_admin_all_medical"
on medical_records for all
using (current_user_role() in ('owner','admin'));

create policy "dokter_own_medical"
on medical_records for all
using (
  current_user_role() = 'dokter'
  and doctor_id = current_user_id()
);

create policy "customer_own_pet_medical"
on medical_records for select
using (
  current_user_role() = 'customer'
  and pet_id in (
    select p.id from pets p
    join customers c on c.id = p.customer_id
    where c.profile_id = current_user_id()
  )
);

-- ============ PRODUCTS ============
alter table products enable row level security;

create policy "owner_admin_manage_products"
on products for all
using (current_user_role() in ('owner','admin'));

create policy "staff_read_products"
on products for select
using (current_user_role() in ('owner','admin','kasir','dokter','customer'));

-- ============ STOCK MOVEMENTS ============
create policy "owner_admin_stock"
on stock_movements for all
using (current_user_role() in ('owner','admin'));

create policy "kasir_create_stock_out"
on stock_movements for insert
with check (
  current_user_role() = 'kasir'
  and type = 'out'
  and user_id = current_user_id()
);
```

---

## 4. 🔑 MEKANISME LOGIN PIN (AMAN)

```typescript
// supabase/functions/verify-pin/index.ts (Edge Function)
import { createClient } from '@supabase/supabase-js';
import bcrypt from 'bcryptjs';

export async function verifyPinLogin(username: string, pin: string) {
  const admin = createClient(SB_URL, SB_SERVICE_KEY);
  
  // 1. Ambil user
  const { data: user } = await admin
    .from('profiles')
    .select('*')
    .eq('username', username)
    .eq('is_active', true)
    .single();
  
  if (!user) return { error: 'Username tidak ditemukan' };
  
  // 2. Cek lock
  if (user.pin_locked_until && new Date(user.pin_locked_until) > new Date()) {
    return { error: 'Akun terkunci, coba lagi nanti' };
  }
  
  // 3. Verifikasi PIN
  const valid = await bcrypt.compare(pin, user.pin_hash);
  
  if (!valid) {
    // Increment attempts
    const attempts = user.pin_attempts + 1;
    const locked = attempts >= 5;
    
    await admin.from('profiles').update({
      pin_attempts: attempts,
      pin_locked_until: locked ? new Date(Date.now() + 15*60*1000) : null
    }).eq('id', user.id);
    
    return { error: locked ? 'Akun terkunci 15 menit' : `PIN salah (${attempts}/5)` };
  }
  
  // 4. Reset attempts & generate session
  await admin.from('profiles').update({
    pin_attempts: 0,
    pin_locked_until: null,
    last_login: new Date()
  }).eq('id', user.id);
  
  // 5. Create custom JWT via Supabase Auth
  const { data } = await admin.auth.admin.generateLink({
    type: 'magiclink',
    email: `${username}@petshop.local`
  });
  
  return { 
    success: true, 
    user: { 
      id: user.id, 
      role: user.role, 
      outlet_id: user.outlet_id,
      name: user.full_name 
    }
  };
}
```

**UI Login PIN:**
- Numeric keypad 4-6 digit
- Haptic feedback
- Auto-focus, auto-submit
- Animasi shake saat salah
- Countdown lock timer
- Biometric (FaceID/TouchID) untuk repeat login

---

## 5. 📁 STRUKTUR PROJECT (Next.js 14)

```
petshop-enterprise/
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx                   # PIN login screen
│   │   └── layout.tsx
│   │
│   ├── (dashboard)/
│   │   ├── layout.tsx                     # Sidebar + header dinamis per role
│   │   │
│   │   ├── owner/
│   │   │   ├── page.tsx                   # Executive dashboard
│   │   │   ├── reports/
│   │   │   │   ├── sales/page.tsx
│   │   │   │   ├── profit-loss/page.tsx
│   │   │   │   ├── inventory-valuation/page.tsx
│   │   │   │   └── customer-analysis/page.tsx
│   │   │   ├── outlets/page.tsx
│   │   │   ├── staff/page.tsx
│   │   │   ├── payroll/page.tsx
│   │   │   ├── accounting/
│   │   │   │   ├── journal/page.tsx
│   │   │   │   ├── cash-flow/page.tsx
│   │   │   │   └── balance-sheet/page.tsx
│   │   │   ├── audit-logs/page.tsx
│   │   │   └── settings/
│   │   │
│   │   ├── admin/
│   │   │   ├── page.tsx                   # Admin dashboard
│   │   │   ├── pos/
│   │   │   │   ├── page.tsx               # Kasir utama
│   │   │   │   ├── held/page.tsx          # Held transactions
│   │   │   │   └── returns/page.tsx
│   │   │   ├── inventory/
│   │   │   │   ├── stock/page.tsx
│   │   │   │   ├── movement/page.tsx
│   │   │   │   ├── opname/page.tsx
│   │   │   │   ├── transfer/page.tsx
│   │   │   │   └── batches/page.tsx
│   │   │   ├── products/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── categories/page.tsx
│   │   │   │   └── brands/page.tsx
│   │   │   ├── purchase/
│   │   │   │   ├── orders/page.tsx
│   │   │   │   ├── suppliers/page.tsx
│   │   │   │   └── payables/page.tsx
│   │   │   ├── customers/
│   │   │   │   ├── page.tsx
│   │   │   │   ├── pets/page.tsx
│   │   │   │   └── loyalty/page.tsx
│   │   │   ├── services/
│   │   │   │   ├── grooming/page.tsx
│   │   │   │   ├── hotel/page.tsx
│   │   │   │   └── packages/page.tsx
│   │   │   ├── marketing/
│   │   │   │   ├── vouchers/page.tsx
│   │   │   │   ├── promotions/page.tsx
│   │   │   │   └── campaigns/page.tsx
│   │   │   └── expenses/page.tsx
│   │   │
│   │   ├── dokter/
│   │   │   ├── page.tsx                   # Jadwal hari ini
│   │   │   ├── appointments/page.tsx
│   │   │   ├── medical-records/
│   │   │   │   ├── page.tsx
│   │   │   │   └── [petId]/page.tsx
│   │   │   ├── vaccinations/page.tsx
│   │   │   ├── prescriptions/page.tsx
│   │   │   └── patients/page.tsx
│   │   │
│   │   └── customer/
│   │       ├── page.tsx                   # Home customer
│   │       ├── booking/
│   │       │   ├── grooming/page.tsx
│   │       │   └── hotel/page.tsx
│   │       ├── my-pets/page.tsx
│   │       ├── history/page.tsx
│   │       ├── loyalty/page.tsx
│   │       └── profile/page.tsx
│   │
│   └── api/
│       ├── auth/
│       │   ├── login/route.ts
│       │   └── logout/route.ts
│       ├── pos/
│       │   ├── checkout/route.ts
│       │   ├── void/route.ts
│       │   └── hold/route.ts
│       ├── inventory/
│       │   ├── stock/route.ts
│       │   └── opname/route.ts
│       ├── services/
│       │   ├── booking/route.ts
│       │   └── daily-log/route.ts
│       ├── notifications/
│       │   └── whatsapp/route.ts
│       └── webhooks/
│           └── payment/route.ts
│
├── components/
│   ├── ui/                                # shadcn components
│   ├── layout/
│   │   ├── sidebar.tsx
│   │   ├── header.tsx
│   │   └── pin-pad.tsx
│   ├── pos/
│   │   ├── product-grid.tsx
│   │   ├── cart.tsx
│   │   ├── payment-modal.tsx
│   │   ├── receipt-printer.tsx
│   │   └── barcode-scanner.tsx
│   ├── inventory/
│   │   ├── stock-table.tsx
│   │   ├── batch-manager.tsx
│   │   └── opname-form.tsx
│   ├── services/
│   │   ├── grooming-calendar.tsx
│   │   ├── hotel-room-map.tsx
│   │   ├── daily-log-form.tsx
│   │   └── medical-record-form.tsx
│   ├── dashboard/
│   │   ├── kpi-card.tsx
│   │   ├── sales-chart.tsx
│   │   └── top-products.tsx
│   └── shared/
│       ├── data-table.tsx
│       ├── file-upload.tsx
│       ├── date-range-picker.tsx
│       └── confirm-dialog.tsx
│
├── lib/
│   ├── supabase/
│   │   ├── client.ts
│   │   ├── server.ts
│   │   └── middleware.ts
│   ├── auth.ts
│   ├── permissions.ts
│   ├── utils.ts
│   ├── constants.ts
│   └── validators.ts
│
├── hooks/
│   ├── use-auth.ts
│   ├── use-realtime.ts
│   ├── use-permission.ts
│   └── use-notification.ts
│
├── types/
│   ├── database.ts                        # Auto-generated
│   ├── pos.ts
│   ├── inventory.ts
│   └── services.ts
│
├── stores/
│   ├── pos-store.ts                       # Zustand
│   ├── cart-store.ts
│   └── notification-store.ts
│
└── supabase/
    ├── migrations/
    ├── functions/
    └── seed.sql
```

---

## 6. 🎯 FITUR DETAIL PER MODUL

### 📦 MODUL 1: POS & KASIR (Admin/Kasir)

| Fitur | Deskripsi |
|-------|-----------|
| **Product Search** | Cari via nama, SKU, barcode (scan), kategori |
| **Quick Category** | Tab kategori cepat (Makanan, Snack, Obat, Mainan) |
| **Shopping Cart** | Tambah/kurang qty, edit harga manual (butuh approval) |
| **Customer Selection** | Pilih customer existing / tambah baru / guest |
| **Discount** | Per-item (%/Rp), per-transaksi, voucher code |
| **Split Payment** | Bayar sebagian cash, sebagian QRIS, dll |
| **Hold Transaction** | Simpan cart sementara, ambil lagi nanti |
| **Multi-Receipt** | Cetak struk thermal, PDF, kirim WA |
| **Loyalty Integration** | Auto earn/redeem points |
| **Void Transaction** | Batalkan transaksi (butuh PIN owner/admin) |
| **Return/Refund** | Retur barang dengan approval |
| **Cash Drawer** | Buka kas, setor kas, closing harian |
| **Offline Mode** | Queue transaksi saat offline, sync saat online |
| **Shortcut Keys** | F1-F12 untuk aksi cepat |
| **Price Check** | Mode cek harga tanpa transaksi |
| **Customer Display** | Tampilan 2nd screen untuk customer |

### 📊 MODUL 2: INVENTORY & WAREHOUSE (Admin)

| Fitur | Deskripsi |
|-------|-----------|
| **Real-time Stock** | Update instan via Supabase Realtime |
| **Multi-Warehouse** | Gudang utama, display, rusak |
| **Batch Tracking** | No. batch, expired date, HPP per batch |
| **FEFO System** | First Expired First Out otomatis |
| **Stock Movement** | Log semua pergerakan (in/out/adjust) |
| **Stock Opname** | Proses SO dengan selisih & approval |
| **Stock Transfer** | Transfer antar gudang/outlet |
| **Low Stock Alert** | Notifikasi otomatis saat ≤ min stock |
| **Expired Alert** | H-30, H-7, H-1 notifikasi expired |
| **Stock Valuation** | Average, FIFO, specific identification |
| **Barcode Generator** | Generate & print barcode produk |
| **Import/Export** | Excel bulk upload produk & stok |
| **Stock Card** | Kartu stok per produk (Kartu Persediaan) |
| **Stock Forecast** | Prediksi kebutuhan berdasarkan sales history |
| **ABC Analysis** | Klasifikasi produk A/B/C berdasarkan value |

### 🏥 MODUL 3: KLINIK & REKAM MEDIS (Dokter)

| Fitur | Deskripsi |
|-------|-----------|
| **Appointment Calendar** | Jadwal konsultasi, drag & drop |
| **Queue Management** | Antrian pasien real-time |
| **EMR (Electronic Medical Record)** | Rekam medis lengkap digital |
| **SOAP Notes** | Subjective, Objective, Assessment, Plan |
| **Body Chart** | Diagram tubuh untuk tandai area masalah |
| **Diagnosis Code** | ICD-10 veterinary codes |
| **Prescription Builder** | Resep dengan dosis, frekuensi, durasi |
| **Vaccination Record** | Riwayat vaksin + jadwal berikutnya |
| **Lab Result Entry** | Input hasil lab, grafik trend |
| **Photo Documentation** | Foto kondisi sebelum/sesudah |
| **Medical History Timeline** | Timeline riwayat medis per pet |
| **Drug Interaction Check** | Cek interaksi obat (basic) |
| **Weight Chart** | Grafik berat badan per visit |
| **Print Medical Certificate** | Surat keterangan sehat |
| **Referral Letter** | Surat rujukan ke specialist |
| **Follow-up Reminder** | Auto reminder kontrol berikutnya |

### ✂️ MODUL 4: GROOMING (Admin/Groomer)

| Fitur | Deskripsi |
|-------|-----------|
| **Booking Calendar** | Jadwal grooming per groomer |
| **Service Packages** | Paket grooming (Basic, Full, Premium) |
| **Add-on Services** | Extra: cut kuku, cleaning telinga, dll |
| **Groomer Assignment** | Assign groomer spesifik |
| **Status Tracking** | Pending → In Progress → Done → Picked Up |
| **Before/After Photo** | Upload foto hasil grooming |
| **Special Request** | Catatan: "jangan cukur bagian wajah" |
| **Skin/Fur Assessment** | Catatan kondisi kulit & bulu |
| **Duration Tracking** | Waktu mulai & selesai aktual |
| **Commission Tracking** | Komisi per groomer |
| **Customer Notification** | WA: "Grooming selesai, siap dijemput" |
| **Grooming Style Gallery** | Referensi model potongan |
| **Rebooking** | Quick rebook untuk customer regular |
| **Allergy/Warning Flag** | Alert alergi shampoo tertentu |

### 🏨 MODUL 5: PET HOTEL (Admin)

| Fitur | Deskripsi |
|-------|-----------|
| **Room Map** | Visual layout kamar (available/occupied) |
| **Room Types** | Small, Medium, Large, VIP, Suite |
| **Booking Calendar** | Timeline booking per kamar |
| **Check-in/Check-out** | Proses arrival & departure |
| **Daily Care Log** | Log harian: makan, BAB, aktivitas, mood |
| **Photo/Video Update** | Kirim update harian ke owner pet |
| **Feeding Schedule** | Jadwal & porsi makan |
| **Medication Schedule** | Jadwal obat jika ada |
| **Walk/Play Log** | Catatan jalan-jalan/bermain |
| **Weight Monitoring** | Timbang harian |
| **Incident Report** | Laporkan jika ada masalah |
| **Deposit Management** | Deposit & refund |
| **Extension Booking** | Perpanjang masa inap |
| **Auto Reminder** | H-1 reminder check-out ke customer |
| **Multi-pet Discount** | Diskon inap >1 pet |
| **CCTV Integration** | Link live CCTV (opsional) |

### 👥 MODUL 6: CRM & CUSTOMER (Admin)

| Fitur | Deskripsi |
|-------|-----------|
| **Customer Database** | Data lengkap + history |
| **Pet Profiles** | Data hewan peliharaan per customer |
| **Purchase History** | Riwayat transaksi produk & layanan |
| **Service History** | Riwayat grooming/hotel/klinik |
| **Loyalty Program** | Tier, points, rewards |
| **Customer Segmentation** | VIP, regular, dormant, new |
| **Birthday Reminder** | Reminder ulang tahun customer/pet |
| **Preference Notes** | Catatan preferensi |
| **Communication Log** | Riwayat chat/telepon |
| **Referral Program** | Kode referral + reward |
| **Blacklist** | Customer bermasalah |
| **Merge Duplicates** | Gabung data duplikat |
| **Import/Export** | Bulk customer data |
| **Customer 360 View** | Satu halaman semua info customer |

### 💰 MODUL 7: PURCHASE & SUPPLIER (Admin/Owner)

| Fitur | Deskripsi |
|-------|-----------|
| **Supplier Management** | Data supplier lengkap |
| **Purchase Order** | Buat PO ke supplier |
| **Goods Received Note** | Catat barang masuk |
| **Partial Receive** | Terima barang bertahap |
| **Supplier Price List** | Harga per supplier |
| **Outstanding Balance** | Hutang ke supplier |
| **Payment Scheduling** | Jadwal pembayaran berdasarkan tempo |
| **Supplier Performance** | Rating supplier (kualitas, ketepatan) |
| **Purchase Comparison** | Bandingkan harga antar supplier |
| **Auto Reorder** | Saran reorder berdasarkan min stock |
| **Return to Supplier** | Retur barang ke supplier |
| **Supplier Statement** | Rekening koran supplier |

### 💵 MODUL 8: ACCOUNTING & FINANCE (Owner)

| Fitur | Deskripsi |
|-------|-----------|
| **Chart of Accounts** | Struktur akun lengkap |
| **Journal Entries** | Jurnal manual & otomatis |
| **Auto Journal** | Jurnal otomatis dari transaksi |
| **General Ledger** | Buku besar per akun |
| **Trial Balance** | Neraca saldo |
| **Profit & Loss** | Laporan laba rugi |
| **Balance Sheet** | Neraca |
| **Cash Flow** | Arus kas |
| **Cash Register** | Kas per outlet |
| **Daily Closing** | Closing kas harian |
| **Expense Management** | Catat pengeluaran operasional |
| **Receivables** | Piutang customer |
| **Payables** | Hutang supplier |
| **Bank Reconciliation** | Rekonsiliasi bank |
| **Tax Report** | Laporan PPN/PPh |
| **Budgeting** | Anggaran vs realisasi |
| **Export Accounting** | Export ke format MYOB/Accurate/XLS |

### 👨‍💼 MODUL 9: HRM & PAYROLL (Owner)

| Fitur | Deskripsi |
|-------|-----------|
| **Employee Database** | Data karyawan lengkap |
| **Attendance** | Clock in/out, late tracking |
| **Leave Management** | Cuti, sakit, izin |
| **Payroll Calculation** | Gaji pokok + tunjangan + komisi - pajak |
| **Commission Tracking** | Komisi kasir, groomer, dokter |
| **Overtime** | Hitung lembur |
| **Payroll Report** | Slip gaji, report bulanan |
| **BPJS Calculation** | Hitung BPJS Kesehatan & Ketenagakerjaan |
| **Tax PPh 21** | Hitung pajak progresif |
| **Performance Review** | Penilaian kinerja |
| **Document Management** | Kontrak, SK, dokumen karyawan |

### 🎁 MODUL 10: MARKETING & PROMO (Admin/Owner)

| Fitur | Deskripsi |
|-------|-----------|
| **Voucher System** | Voucher %/Rp/free item/BOGO |
| **Promotion Engine** | Diskon bertingkat, bundle, buy X get Y |
| **Flash Sale** | Promo waktu terbatas |
| **Membership Tier** | Regular, Silver, Gold, Platinum |
| **Loyalty Points** | Earn & redeem points |
| **Referral Program** | Ajak teman dapat reward |
| **Birthday Promo** | Auto promo saat ulang tahun |
| **Campaign Builder** | Blast WA/email ke segment customer |
| **Promo Analytics** | Efektivitas promo (redemption rate) |
| **Coupon Generator** | Generate kode kupon massal |
| **Landing Page** | Halaman promo khusus |

### 📈 MODUL 11: MONITORING & BI (Owner)

| Fitur | Deskripsi |
|-------|-----------|
| **Executive Dashboard** | KPI utama: revenue, profit, customers |
| **Sales Analytics** | Grafik penjualan harian/mingguan/bulanan |
| **Product Performance** | Best/worst selling products |
| **Category Analysis** | Penjualan per kategori |
| **Customer Analysis** | Acquisition, retention, CLV |
| **Service Analytics** | Utilisasi grooming/hotel/klinik |
| **Staff Performance** | Produktivitas kasir, groomer, dokter |
| **Inventory Analytics** | Turnover ratio, dead stock |
| **Forecasting** | Prediksi penjualan (basic) |
| **Outlet Comparison** | Bandingkan performa antar outlet |
| **Custom Reports** | Report builder |
| **Export** | PDF, Excel, CSV |
| **Scheduled Reports** | Auto email report harian/mingguan |
| **Heatmap** | Jam sibuk, hari sibuk |
| **Cohort Analysis** | Retention customer per bulan |

### 🔔 MODUL 12: NOTIFICATION CENTER (All)

| Fitur | Deskripsi |
|-------|-----------|
| **WhatsApp Integration** | Kirim invoice, reminder, promo |
| **Email Notification** | Email otomatis |
| **In-App Notification** | Notifikasi di aplikasi |
| **SMS Gateway** | SMS untuk customer tanpa WA |
| **Appointment Reminder** | H-1 reminder appointment |
| **Vaccine Reminder** | Reminder vaksin berikutnya |
| **Hotel Daily Report** | Update harian ke owner pet |
| **Low Stock Alert** | Alert ke admin saat stok menipis |
| **Payment Reminder** | Reminder piutang jatuh tempo |
| **Birthday Greeting** | Auto greeting ulang tahun |
| **Template Builder** | Custom template pesan |
| **Delivery Log** | Log pesan terkirim |

### ⚙️ MODUL 13: SETTINGS & CONFIGURATION (Owner/Admin)

| Fitur | Deskripsi |
|-------|-----------|
| **Outlet Management** | Multi-cabang |
| **Business Profile** | Nama, alamat, logo, NPWP |
| **Tax Configuration** | Setting pajak |
| **Payment Methods** | Aktifkan/nonaktifkan metode bayar |
| **Printer Setup** | Setup printer thermal, A4 |
| **Receipt Template** | Custom template struk |
| **Barcode Settings** | Format barcode |
| **Numbering Format** | Format nomor invoice, PO, dll |
| **User Management** | Tambah/edit user, reset PIN |
| **Role & Permission** | Setting hak akses granular |
| **Backup & Restore** | Manual backup database |
| **Integration Settings** | API keys (WA, payment gateway) |
| **Localization** | Bahasa, mata uang, timezone |
| **Theme Settings** | Dark/light mode |

### 🔒 MODUL 14: AUDIT & SECURITY (Owner)

| Fitur | Deskripsi |
|-------|-----------|
| **Audit Log** | Semua aksi user tercatat |
| **Login History** | Riwayat login (IP, device) |
| **Session Management** | Lihat & revoke session aktif |
| **PIN Policy** | Min length, complexity, expiry |
| **Failed Login Monitor** | Deteksi brute force |
| **Sensitive Action Log** | Void, refund, edit harga tercatat |
| **Data Change History** | Before/after values |
| **IP Whitelist** | Batasi akses per IP |
| **2FA (Optional)** | Two-factor authentication |
| **Data Export Log** | Siapa export data apa |

### 📱 MODUL 15: CUSTOMER PORTAL (Customer)

| Fitur | Deskripsi |
|-------|-----------|
| **Self-Registration** | Daftar sendiri via WA/web |
| **Pet Management** | Tambah/edit data pet |
| **Online Booking** | Booking grooming/hotel/klinik |
| **Booking History** | Riwayat booking |
| **Transaction History** | Riwayat belanja |
| **Medical Records View** | Lihat rekam medis pet |
| **Vaccination Certificate** | Download sertifikat vaksin |
| **Loyalty Dashboard** | Points, tier, rewards |
| **Voucher Wallet** | Voucher yang dimiliki |
| **Payment History** | Riwayat pembayaran |
| **Profile Management** | Edit profil, alamat |
| **Notification Preferences** | Setting notifikasi |
| **Feedback/Rating** | Beri rating layanan |
| **Invoice Download** | Download invoice PDF |

---

## 7. 🎭 MATRIX HAK AKSES PER ROLE

| Modul | Owner | Admin | Dokter | Kasir | Groomer | Customer |
|-------|:-----:|:-----:|:------:|:-----:|:-------:|:--------:|
| Executive Dashboard | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| POS / Kasir | ✅ | ✅ | ❌ | ✅ | ❌ | ❌ |
| Inventory | ✅ | ✅ | 👁️ | 👁️ | ❌ | ❌ |
| Products | ✅ | ✅ | 👁️ | 👁️ | 👁️ | 👁️ |
| Purchase | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Supplier | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Customer CRM | ✅ | ✅ | 👁️ | 👁️ | 👁️ | 👁️(own) |
| Pet Management | ✅ | ✅ | ✅ | 👁️ | 👁️ | ✅(own) |
| Grooming | ✅ | ✅ | 👁️ | ✅ | ✅(own) | ✅(own) |
| Pet Hotel | ✅ | ✅ | 👁️ | ✅ | ❌ | ✅(own) |
| Clinic/Medical | ✅ | ✅ | ✅ | ❌ | ❌ | 👁️(own) |
| Vaccination | ✅ | ✅ | ✅ | ❌ | ❌ | 👁️(own) |
| Prescription | ✅ | ✅ | ✅ | ✅(fulfill) | ❌ | 👁️(own) |
| Accounting | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Payroll | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| HRM | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Marketing | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Reports/BI | ✅ | ✅(limited) | 👁️(own) | 👁️(own) | 👁️(own) | ✅(own) |
| Vouchers | ✅ | ✅ | ❌ | ✅(apply) | ❌ | ✅(own) |
| Settings | ✅ | ✅(limited) | ❌ | ❌ | ❌ | ❌ |
| Audit Log | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| Void/Refund | ✅ | ✅ | ❌ | ❌(need approval) | ❌ | ❌ |

**Legend:** ✅ = Full Access | 👁️ = Read Only | ✅(own) = Hanya data sendiri | ❌ = No Access

---

## 8. 🔌 API ENDPOINTS UTAMA

```typescript
// AUTH
POST   /api/auth/login              // Login PIN
POST   /api/auth/logout             // Logout
POST   /api/auth/change-pin         // Ganti PIN
POST   /api/auth/forgot-pin         // Request reset PIN

// POS
POST   /api/pos/checkout            // Proses transaksi
POST   /api/pos/hold                // Hold transaction
GET    /api/pos/held                // List held transactions
POST   /api/pos/void                // Void transaction
POST   /api/pos/return              // Retur barang
GET    /api/pos/receipt/:id         // Get receipt data
POST   /api/pos/print               // Trigger print

// INVENTORY
GET    /api/inventory/stock         // List stok
POST   /api/inventory/adjustment    // Adjustment stok
POST   /api/inventory/opname        // Stock opname
POST   /api/inventory/transfer      // Transfer stok
GET    /api/inventory/movements     // Stock movements
GET    /api/inventory/expiring      // Produk mendekati expired
GET    /api/inventory/low-stock     // Produk stok rendah

// PRODUCTS
GET    /api/products                // List produk
POST   /api/products                // Create produk
PUT    /api/products/:id            // Update produk
DELETE /api/products/:id            // Delete produk
POST   /api/products/import         // Bulk import
GET    /api/products/export         // Export produk

// CUSTOMERS
GET    /api/customers               // List customer
POST   /api/customers               // Create customer
GET    /api/customers/:id/pets      // Pets of customer
GET    /api/customers/:id/history   // History transaksi

// PETS
GET    /api/pets                    // List pets
POST   /api/pets                    // Create pet
GET    /api/pets/:id/medical        // Medical records

// SERVICES - GROOMING
GET    /api/grooming/bookings       // List bookings
POST   /api/grooming/bookings       // Create booking
PUT    /api/grooming/bookings/:id   // Update booking
POST   /api/grooming/:id/complete   // Complete grooming

// SERVICES - HOTEL
GET    /api/hotel/rooms             // List rooms
GET    /api/hotel/bookings          // List bookings
POST   /api/hotel/bookings          // Create booking
POST   /api/hotel/:id/checkin       // Check in
POST   /api/hotel/:id/checkout      // Check out
POST   /api/hotel/:id/daily-log     // Daily care log

// SERVICES - CLINIC
GET    /api/clinic/appointments     // List appointments
POST   /api/clinic/appointments     // Create appointment
POST   /api/clinic/:id/medical      // Create medical record
POST   /api/clinic/:id/vaccination  // Record vaccination
POST   /api/clinic/:id/prescription // Create prescription

// PURCHASE
GET    /api/purchase/orders         // List PO
POST   /api/purchase/orders         // Create PO
POST   /api/purchase/orders/:id/receive // Receive goods
GET    /api/purchase/suppliers      // List suppliers

// ACCOUNTING
GET    /api/accounting/journal      // Journal entries
GET    /api/accounting/cash-flow    // Cash flow
GET    /api/accounting/profit-loss  // P&L report
GET    /api/accounting/balance-sheet// Balance sheet
POST   /api/accounting/closing      // Cash closing

// HRM
GET    /api/hrm/attendance          // Attendance
POST   /api/hrm/attendance/clock    // Clock in/out
GET    /api/hrm/payroll             // Payroll list
POST   /api/hrm/payroll/generate    // Generate payroll

// MARKETING
GET    /api/marketing/vouchers      // List vouchers
POST   /api/marketing/vouchers      // Create voucher
POST   /api/marketing/vouchers/validate // Validate voucher
GET    /api/marketing/promotions    // List promotions

// REPORTS
GET    /api/reports/sales           // Sales report
GET    /api/reports/products        // Product report
GET    /api/reports/customers       // Customer report
GET    /api/reports/inventory       // Inventory report
GET    /api/reports/services        // Services report

// NOTIFICATIONS
POST   /api/notifications/whatsapp  // Send WA
POST   /api/notifications/email     // Send email
GET    /api/notifications/list      // List notifications
POST   /api/notifications/read      // Mark as read

// SETTINGS
GET    /api/settings                // Get settings
PUT    /api/settings                // Update settings
GET    /api/audit-logs              // Audit logs
```

---

## 9. 🚀 DEPLOYMENT & INFRASTRUKTUR

### 9.1 Environment Variables (Vercel)

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...

# WhatsApp Gateway (Fonnte)
FONNTE_TOKEN=xxxxx
FONNTE_DEVICE=xxxxx

# Payment Gateway (Midtrans)
MIDTRANS_SERVER_KEY=xxxxx
MIDTRANS_CLIENT_KEY=xxxxx
MIDTRANS_IS_PRODUCTION=false

# Email (Resend)
RESEND_API_KEY=re_xxxxx

# App
NEXT_PUBLIC_APP_URL=https://petshop.com
NEXT_PUBLIC_APP_NAME="PetShop Enterprise"
NEXT_PUBLIC_DEFAULT_OUTLET_ID=uuid-here
```

### 9.2 Vercel Config

```json
{
  "framework": "nextjs",
  "regions": ["sin1"],
  "functions": {
    "app/api/**/*": {
      "maxDuration": 30
    }
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "no-store" }
      ]
    }
  ]
}
```

### 9.3 Supabase Setup

```bash
# Install Supabase CLI
npm i -g supabase

# Init project
supabase init

# Link ke project
supabase link --project-ref xxxxx

# Push migrations
supabase db push

# Generate types
supabase gen types typescript --local > types/database.ts
```

---

## 10. 💡 FITUR ADVANCED (BONUS)

### 10.1 AI-Powered Features
- **Demand Forecasting**: Prediksi kebutuhan stok 30 hari ke depan
- **Smart Reorder**: Auto-suggest PO berdasarkan sales velocity
- **Customer Churn Prediction**: Identifikasi customer yang akan hilang
- **Price Optimization**: Saran harga optimal berdasarkan elastisitas
- **Voice Order**: Kasir via suara ("Tambah Royal Canin 2 kg")

### 10.2 IoT Integration
- **Smart Scale**: Timbangan terhubung untuk timbang makanan
- **Temperature Sensor**: Monitor suhu pet hotel & vaccine fridge
- **CCTV Integration**: Live view di dashboard hotel
- **Smart Feeder**: Auto feed di hotel sesuai jadwal

### 10.3 Mobile Apps
- **Kasir Mobile**: Tablet POS untuk kasir
- **Dokter Mobile**: Akses rekam medis via mobile
- **Customer App**: iOS/Android app untuk customer
- **Groomer App**: Update status grooming via mobile

### 10.4 Advanced Analytics
- **Customer Lifetime Value (CLV)**
- **Cohort Analysis**
- **Market Basket Analysis** (produk yang sering dibeli bersama)
- **RFM Analysis** (Recency, Frequency, Monetary)
- **Break-even Analysis**

---

## 11. 📊 ESTIMASI BIAYA & TIMELINE

### 11.1 Biaya Bulanan

| Komponen | Plan | Biaya |
|----------|------|-------|
| Vercel Pro | 1 seat | $20 |
| Supabase Pro | Database + Realtime | $25 |
| Domain + SSL | .com | $1 |
| WhatsApp Gateway | Fonnte/Wablas | $10-30 |
| Email (Resend) | 50k emails | $20 |
| Backup Storage | 100GB | $5 |
| **TOTAL** | | **~$85-105/bulan** |

### 11.2 Timeline Implementasi (1 Developer)

| Fase | Durasi | Deliverable |
|------|--------|-------------|
| **Phase 1: Foundation** | 2 minggu | DB schema, auth PIN, layout, permissions |
| **Phase 2: Core POS** | 3 minggu | POS lengkap, inventory, products |
| **Phase 3: Services** | 3 minggu | Grooming, hotel, clinic |
| **Phase 4: CRM & Finance** | 2 minggu | Customer, loyalty, accounting |
| **Phase 5: HRM & Marketing** | 2 minggu | Payroll, vouchers, campaigns |
| **Phase 6: BI & Reports** | 2 minggu | Dashboard, reports, analytics |
| **Phase 7: Integration** | 2 minggu | WhatsApp, payment, printer |
| **Phase 8: Testing & Deploy** | 2 minggu | UAT, training, go-live |
| **TOTAL** | **~18 minggu** | |

---

## 12. 🎯 REKOMENDASI PRIORITAS MVP

Untuk launch cepat, fokus ke **MVP (Minimum Viable Product)** dulu:

### 🥇 Priority 1 (Minggu 1-6)
1. ✅ Auth PIN + Role Management
2. ✅ Master Produk + Inventory Dasar
3. ✅ POS Lengkap + Kasir
4. ✅ Customer + Pet Management
5. ✅ Laporan Penjualan Dasar

### 🥈 Priority 2 (Minggu 7-12)
6. ✅ Grooming Booking
7. ✅ Pet Hotel
8. ✅ Klinik + Rekam Medis
9. ✅ Purchase + Supplier
10. ✅ Loyalty Points

### 🥉 Priority 3 (Minggu 13-18)
11. ✅ Accounting Dasar
12. ✅ Payroll
13. ✅ Marketing & Voucher
14. ✅ BI Dashboard
15. ✅ WhatsApp Integration

---

## 🎁 BONUS: Quick Start Commands

```bash
# 1. Create project
npx create-next-app@latest petshop-enterprise --typescript --tailwind --app --src-dir

# 2. Install dependencies
cd petshop-enterprise
npm i @supabase/supabase-js @supabase/ssr
npm i zustand recharts date-fns lucide-react
npm i zod react-hook-form @hookform/resolvers
npm i bcryptjs clsx tailwind-merge
npm i -D @types/bcryptjs

# 3. Init shadcn
npx shadcn-ui@latest init
npx shadcn-ui@latest add button input card dialog table form toast

# 4. Init Supabase
npx supabase init
npx supabase start

# 5. Run dev
npm run dev
```

---

## 📝 CHECKLIST IMPLEMENTASI

- [ ] Setup Supabase project
- [ ] Run semua migrations
- [ ] Seed data awal (roles, outlets, accounts)
- [ ] Setup Vercel project + env vars
- [ ] Implement auth PIN
- [ ] Setup RLS policies
- [ ] Build dashboard layout per role
- [ ] Modul POS
- [ ] Modul Inventory
- [ ] Modul Services (grooming, hotel, clinic)
- [ ] Modul CRM
- [ ] Modul Accounting
- [ ] Modul Reports
- [ ] WhatsApp integration
- [ ] Printer thermal setup
- [ ] UAT dengan user asli
- [ ] Training staff
- [ ] Go-live 🚀

---

