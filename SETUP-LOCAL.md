# راهنمای اجرای پروژه بدون Docker (محلی / Local)

> این راهنما نحوه راه‌اندازی مستقیم پروژه Shop Multi-Vendor روی سیستم محلی بدون Docker را توضیح می‌دهد.

---

## پیش‌نیازها

| ابزار | نسخه | لینک |
|-------|------|------|
| Node.js | 20 LTS | https://nodejs.org |
| npm | 10+ (همراه Node) | – |
| PostgreSQL | 15 یا 16 | https://www.postgresql.org/download |
| Redis | 7+ | https://redis.io/docs/install |
| Git | هر نسخه | – |

### بررسی نصب:
```bash
node --version      # باید v20.x.x باشد
npm --version       # باید 10.x.x باشد
psql --version      # باید 15 یا 16 باشد
redis-cli ping      # باید PONG برگرداند
```

---

## ۱. آماده‌سازی اولیه

### کلون کردن مخزن

```bash
git clone https://github.com/AmBplus/Shop_Multi_Vendor.git
cd Shop_Multi_Vendor
```

### نصب وابستگی‌ها

```bash
# اگر از npm workspaces استفاده می‌شود:
npm install

# یا نصب جداگانه:
cd apps/frontend && npm install && cd ../..
cd apps/backend  && npm install && cd ../..
```

### ساخت دیتابیس PostgreSQL

```sql
-- اجرا در psql یا pgAdmin
CREATE USER shop_user WITH PASSWORD 'your_password';
CREATE DATABASE shop_db OWNER shop_user;
GRANT ALL PRIVILEGES ON DATABASE shop_db TO shop_user;
```

```bash
# از ترمینال:
psql -U postgres -c "CREATE USER shop_user WITH PASSWORD 'your_password';"
psql -U postgres -c "CREATE DATABASE shop_db OWNER shop_user;"
```

---

## ۲. تنظیم متغیرهای محیطی

### Backend

```bash
cd apps/backend
cp .env.example .env
```

**ویرایش `apps/backend/.env`:**
```env
NODE_ENV=development

# Database
DATABASE_URL=postgresql://shop_user:your_password@localhost:5432/shop_db

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your_super_secret_jwt_key_minimum_32_characters
JWT_REFRESH_SECRET=another_secret_refresh_key_32_characters
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=30d

# URLs
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:4000

# Email
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USER=noreply@example.com
MAIL_PASS=your_email_password
MAIL_FROM="Shop <noreply@example.com>"

# Storage (محلی / MinIO / S3)
STORAGE_TYPE=local           # local | minio | s3
UPLOAD_DIR=./uploads         # برای storage محلی

# یا MinIO:
# STORAGE_TYPE=minio
# MINIO_ENDPOINT=localhost
# MINIO_PORT=9000
# MINIO_ACCESS_KEY=minioadmin
# MINIO_SECRET_KEY=minioadmin
# MINIO_BUCKET=shop-uploads

# SMS (اختیاری)
SMS_PROVIDER=kavenegar       # kavenegar | melipayamak
SMS_API_KEY=your_sms_api_key
SMS_FROM=10008xxx

# Social Login (اختیاری)
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
FACEBOOK_APP_ID=
FACEBOOK_APP_SECRET=
```

### Frontend

```bash
cd apps/frontend
cp .env.example .env.local
```

**ویرایش `apps/frontend/.env.local`:**
```env
NEXT_PUBLIC_API_URL=http://localhost:4000
NEXT_PUBLIC_SITE_NAME=فروشگاه چند فروشنده
NEXT_PUBLIC_DEFAULT_THEME=ocean-blue

# برای فقط Frontend (بدون Backend):
# NEXT_PUBLIC_USE_MOCK=true
```

---

## ۳. اجرای Migration و Seed

```bash
cd apps/backend

# اجرای migration (ساخت جداول)
npm run migration:run

# پر کردن داده‌های اولیه (نقش‌ها، تنظیمات، کاربر ادمین)
npm run seed

# (اختیاری) داده‌های نمونه بیشتر برای تست
npm run seed:demo
```

**بعد از seed، حساب ادمین:**
```
Email: admin@shop.local
Password: Admin@1234
```

---

## ۴. اجرای پروژه

### حالت ۱: Frontend + Backend (کامل)

**ترمینال ۱ – Backend:**
```bash
cd apps/backend
npm run start:dev
# → اجرا روی http://localhost:4000
# → Swagger: http://localhost:4000/api-docs
```

**ترمینال ۲ – Frontend:**
```bash
cd apps/frontend
npm run dev
# → اجرا روی http://localhost:3000
```

---

### حالت ۲: فقط Frontend (بدون Backend)

برای کار روی UI بدون نیاز به backend:

```bash
cd apps/frontend

# تنظیم متغیر Mock در .env.local
echo "NEXT_PUBLIC_USE_MOCK=true" >> .env.local

# اجرا
npm run dev
# → اجرا روی http://localhost:3000
# داده‌های نمونه از فایل‌های JSON در /mock-data/ استفاده می‌شود
```

---

### حالت ۳: فقط Backend (API)

برای توسعه API:

```bash
cd apps/backend
npm run start:dev
```

تست API:
```bash
# بررسی سلامت
curl http://localhost:4000/health

# لیست محصولات
curl http://localhost:4000/api/products

# مستندات کامل:
open http://localhost:4000/api-docs
```

---

## ۵. ساختار پوشه‌ها پس از اجرا

```
Shop_Multi_Vendor/
├── apps/
│   ├── frontend/
│   │   ├── .next/         ← ساخته می‌شود خودکار (gitignore)
│   │   └── public/
│   │       └── fonts/     ← فونت وزیر
│   │
│   └── backend/
│       ├── dist/          ← خروجی build (gitignore)
│       └── uploads/       ← فایل‌های آپلود شده (gitignore)
│
├── docs/                  ← مستندات پروژه
├── SETUP-DOCKER.md
└── SETUP-LOCAL.md
```

---

## ۶. دستورات توسعه

```bash
# اجرای تست‌ها
cd apps/backend  && npm run test          # unit tests
cd apps/backend  && npm run test:e2e      # end-to-end tests
cd apps/frontend && npm run test          # component tests

# Lint
npm run lint
npm run lint:fix

# Build برای production
cd apps/frontend && npm run build
cd apps/backend  && npm run build

# بررسی نوع TypeScript
npm run type-check

# ایجاد migration جدید
cd apps/backend
npm run migration:create -- --name=AddVendorTable
```

---

## ۷. نکات مهم برای Windows

```powershell
# اجرای PostgreSQL service
Start-Service postgresql

# یا با pg_ctl:
pg_ctl start -D "C:\Program Files\PostgreSQL\16\data"

# اجرای Redis (نیاز به WSL یا Redis for Windows)
# توصیه: استفاده از WSL2 برای Redis
wsl redis-server
```

---

## ۸. نکات مهم برای macOS

```bash
# نصب با Homebrew:
brew install postgresql@16 redis

# اجرا به صورت سرویس:
brew services start postgresql@16
brew services start redis

# یا اجرای موقت:
pg_ctl start -D /usr/local/var/postgresql@16
redis-server
```

---

## ۹. رفع مشکلات رایج

### خطای اتصال به دیتابیس
```
Error: connect ECONNREFUSED 127.0.0.1:5432
```
```bash
# بررسی وضعیت PostgreSQL:
pg_isready
pg_ctl status -D /path/to/data

# بررسی تنظیمات pg_hba.conf:
# اطمینان از وجود خط:
# local   all   all   md5
```

### خطای اتصال به Redis
```
Error: connect ECONNREFUSED 127.0.0.1:6379
```
```bash
redis-cli ping  # باید PONG برگرداند
redis-server    # اجرای مستقیم
```

### خطای پورت در حال استفاده
```bash
# پیدا کردن پروسه:
lsof -i :3000   # macOS/Linux
netstat -ano | findstr :3000  # Windows

# Kill کردن پروسه:
kill -9 <PID>   # macOS/Linux
taskkill /PID <PID> /F  # Windows
```

### خطای npm install
```bash
# پاک کردن cache:
npm cache clean --force
rm -rf node_modules package-lock.json
npm install
```

---

## ۱۰. متغیرهای محیطی پیش‌فرض برای تست سریع

اگر می‌خواهید سریع شروع کنید، این مقادیر حداقلی را در `.env` استفاده کنید:

```env
# Backend - حداقل برای شروع
NODE_ENV=development
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/shop_db
REDIS_URL=redis://localhost:6379
JWT_SECRET=dev_secret_key_32_characters_min_here
JWT_REFRESH_SECRET=dev_refresh_secret_32_characters_here
FRONTEND_URL=http://localhost:3000

# Frontend - حداقل برای شروع
NEXT_PUBLIC_API_URL=http://localhost:4000
```

```bash
# ساخت سریع دیتابیس با تنظیمات پیش‌فرض:
psql -U postgres -c "CREATE DATABASE shop_db;"

# اجرای migration + seed:
cd apps/backend && npm run setup:dev  # این یک script ترکیبی است
```

---

> **تبریک!** اگر همه مراحل موفق بود، فروشگاه شما روی `http://localhost:3000` در دسترس است.
>
> برای اجرا با Docker: [SETUP-DOCKER.md](./SETUP-DOCKER.md)
