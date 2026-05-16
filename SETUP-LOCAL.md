# راهنمای اجرای پروژه بدون Docker (محلی / Local)

> این راهنما نحوه راه‌اندازی مستقیم پروژه Shop Multi-Vendor روی سیستم محلی بدون Docker را توضیح می‌دهد.
> بک‌اند با **ASP.NET Core 10** پیاده‌سازی شده است.

---

## پیش‌نیازها

| ابزار | نسخه | لینک |
|-------|------|------|
| .NET SDK | **10.0** | https://dotnet.microsoft.com/download/dotnet/10.0 |
| Node.js | 20 LTS | https://nodejs.org |
| PostgreSQL | 15 یا 16 | https://www.postgresql.org/download |
| Redis | 7+ | https://redis.io/docs/install |
| Git | هر نسخه | – |

### بررسی نصب:
```bash
dotnet --version        # باید 10.x.x باشد
node --version          # باید v20.x.x باشد
psql --version          # باید 15 یا 16 باشد
redis-cli ping          # باید PONG برگرداند
```

---

## ۱. آماده‌سازی اولیه

### کلون کردن مخزن

```bash
git clone https://github.com/AmBplus/Shop_Multi_Vendor.git
cd Shop_Multi_Vendor
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
cd src/Web
cp appsettings.json appsettings.Development.json
```

**ویرایش `src/Web/appsettings.Development.json`:**
```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5432;Database=shop_db;Username=shop_user;Password=your_password"
  },
  "Redis": {
    "ConnectionString": "localhost:6379"
  },
  "Jwt": {
    "Secret": "your_super_secret_jwt_key_minimum_32_characters",
    "RefreshSecret": "another_secret_refresh_key_32_characters",
    "AccessTokenExpiry": "00:15:00",
    "RefreshTokenExpiry": "30.00:00:00"
  },
  "Storage": {
    "Provider": "FileSystem",
    "FileSystem": {
      "BasePath": "./uploads"
    },
    "MinIO": {
      "Endpoint": "localhost:9000",
      "AccessKey": "minioadmin",
      "SecretKey": "minioadmin",
      "Bucket": "shop-files",
      "UseSSL": false
    }
  },
  "Captcha": {
    "CodeLength": 5,
    "TtlSeconds": 180
  },
  "Sms": {
    "Provider": "Kavenegar",
    "Kavenegar": {
      "ApiKey": "your_kavenegar_api_key",
      "Sender": "10008xxx"
    }
  },
  "Email": {
    "Host": "smtp.example.com",
    "Port": 587,
    "User": "noreply@example.com",
    "Password": "your_email_password",
    "From": "Shop <noreply@example.com>"
  }
}
```

### Frontend

```bash
cd apps/frontend
cp .env.example .env.local
```

**ویرایش `apps/frontend/.env.local`:**
```env
NEXT_PUBLIC_API_URL=http://localhost:5000
NEXT_PUBLIC_SITE_NAME=فروشگاه چند فروشنده
NEXT_PUBLIC_DEFAULT_THEME=ocean-blue
```

---

## ۳. Migration و Seed خودکار

دیتابیس به صورت **خودکار** هنگام اجرای برنامه Migration و Seed می‌شود.
نیازی به اجرای دستی دستوری نیست.

پس از اجرا، حساب‌های زیر در دسترس خواهند بود:

```
SuperAdmin:
  Email: superadmin@shop.local
  Mobile: 09000000001
  Password: SuperAdmin@1234

Admin:
  Email: admin@shop.local
  Mobile: 09000000002
  Password: Admin@1234

Vendor:
  Email: vendor@shop.local
  Mobile: 09000000003
  Password: Vendor@1234

Customer:
  Email: customer@shop.local
  Mobile: 09000000004
  Password: Customer@1234
```

---

## ۴. اجرای پروژه

### حالت ۱: Frontend + Backend (کامل)

**ترمینال ۱ – Backend (ASP.NET Core):**
```bash
cd src/Web
dotnet run
# → اجرا روی http://localhost:5000
# → Scalar (API Docs): http://localhost:5000/scalar
```

**ترمینال ۲ – Frontend (Next.js):**
```bash
cd apps/frontend
npm install
npm run dev
# → اجرا روی http://localhost:3000
```

---

### حالت ۲: فقط Backend

```bash
cd src/Web
dotnet run
```

تست API:
```bash
# بررسی سلامت
curl http://localhost:5000/health

# لیست محصولات
curl http://localhost:5000/api/products

# مستندات کامل:
open http://localhost:5000/scalar
```

---

### حالت ۳: فقط Frontend

```bash
cd apps/frontend
npm install
npm run dev
# → اجرا روی http://localhost:3000
```

---

## ۵. ساختار پوشه‌ها پس از اجرا

```
Shop_Multi_Vendor/
├── src/
│   ├── AppCore/          ← هسته برنامه
│   ├── Module/           ← سرویس‌های خارجی
│   ├── Web/
│   │   ├── uploads/      ← فایل‌های آپلود (اگر FileSystem Storage)
│   │   └── logs/         ← لاگ‌ها (gitignore)
│   └── Framework/        ← کلاس‌های پایه
│
├── apps/
│   └── frontend/
│       └── .next/        ← ساخته می‌شود خودکار (gitignore)
│
└── docs/                 ← مستندات پروژه
```

---

## ۶. دستورات توسعه

```bash
# Backend – Build
cd src/Web
dotnet build

# Backend – تست‌ها
cd tests
dotnet test

# Frontend – نصب وابستگی‌ها
cd apps/frontend && npm install

# Frontend – Build برای production
cd apps/frontend && npm run build

# Frontend – بررسی نوع TypeScript
cd apps/frontend && npm run type-check

# Frontend – Lint
cd apps/frontend && npm run lint
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

# نصب .NET 10:
brew install --cask dotnet

# اجرا به صورت سرویس:
brew services start postgresql@16
brew services start redis
```

---

## ۹. رفع مشکلات رایج

### خطای اتصال به دیتابیس
```
Npgsql.NpgsqlException: Connection refused
```
```bash
# بررسی وضعیت PostgreSQL:
pg_isready -h localhost -p 5432
```

### خطای اتصال به Redis
```
StackExchange.Redis.RedisConnectionException
```
```bash
redis-cli ping  # باید PONG برگرداند
redis-server    # اجرای مستقیم
```

### خطای .NET version
```bash
dotnet --list-sdks  # باید 10.x.x نمایش دهد
```

---

## ۱۰. متغیرهای پیش‌فرض برای تست سریع

برای شروع سریع با حداقل تنظیمات:

```json
// appsettings.Development.json – حداقل تنظیمات
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5432;Database=shop_db;Username=postgres;Password=postgres"
  },
  "Redis": {
    "ConnectionString": "localhost:6379"
  },
  "Jwt": {
    "Secret": "dev_secret_key_for_testing_32chars!!",
    "RefreshSecret": "dev_refresh_secret_32chars_here!!"
  },
  "Storage": {
    "Provider": "FileSystem",
    "FileSystem": { "BasePath": "./uploads" }
  }
}
```

```bash
# ساخت سریع دیتابیس:
psql -U postgres -c "CREATE DATABASE shop_db;"

# اجرای برنامه (Migration و Seed خودکار اجرا می‌شوند):
cd src/Web && dotnet run
```

---

> **تبریک!** اگر همه مراحل موفق بود، فروشگاه شما روی `http://localhost:3000` در دسترس است.
>
> برای اجرا با Docker: [SETUP-DOCKER.md](./SETUP-DOCKER.md)

