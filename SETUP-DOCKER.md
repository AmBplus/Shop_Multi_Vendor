# راهنمای اجرای پروژه با Docker

> این راهنما نحوه راه‌اندازی سریع پروژه Shop Multi-Vendor را با استفاده از Docker توضیح می‌دهد.
> بک‌اند با **ASP.NET Core 10** پیاده‌سازی شده است.

---

## پیش‌نیازها

| ابزار | نسخه حداقل | لینک |
|-------|-----------|------|
| Docker | 24+ | https://docs.docker.com/get-docker/ |
| Docker Compose | 2.20+ | همراه Docker Desktop نصب می‌شود |
| Git | هر نسخه | – |

بررسی نصب:
```bash
docker --version
docker compose version
```

---

## حالت‌های اجرا

سه حالت اجرا وجود دارد:

| حالت | توضیح | فایل |
|------|--------|------|
| **حالت ۱** (پیش‌فرض) | همه چیز با Docker – برنامه + دیتابیس + Redis + MinIO | `docker-compose.yml` |
| **حالت ۲** | فقط برنامه با Docker – دیتابیس خارجی از Environment | `docker-compose.app.yml` |
| **حالت ۳** | اجرای محلی کامل – تنظیم در `appsettings.json` | – |

---

## ۱. حالت ۱ – همه چیز با Docker (پیش‌فرض) ✅

### کلون کردن مخزن

```bash
git clone https://github.com/AmBplus/Shop_Multi_Vendor.git
cd Shop_Multi_Vendor
```

### کپی فایل‌های محیطی

```bash
cp .env.example .env
```

**تنظیمات `.env` (ویرایش کنید):**
```env
# Database
DB_PASSWORD=your_strong_password

# JWT
JWT_SECRET=your_super_secret_jwt_key_min_32_chars
JWT_REFRESH_SECRET=another_secret_for_refresh_tokens

# Storage (MinIO)
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# App
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:5000
```

### اجرای همه سرویس‌ها

```bash
# اجرای کامل (frontend + backend + postgres + redis + minio)
docker compose up -d

# مشاهده لاگ‌ها
docker compose logs -f

# مشاهده لاگ سرویس خاص
docker compose logs -f backend
```

**سرویس‌ها پس از اجرا:**

| سرویس | آدرس | توضیح |
|-------|------|--------|
| Frontend (Next.js) | http://localhost:3000 | پنل مشتری |
| Backend (ASP.NET Core) | http://localhost:5000 | API |
| API Docs (Scalar) | http://localhost:5000/scalar | مستندات API |
| PostgreSQL | localhost:5432 | پایگاه داده |
| Redis | localhost:6379 | کش |
| MinIO Console | http://localhost:9001 | مدیریت فایل |
| Adminer | http://localhost:8080 | مدیریت DB |

> **نکته:** دیتابیس به صورت **خودکار** Migration و Seed می‌شود. نیازی به اجرای دستی دستوری نیست.

---

## ۲. حالت ۲ – برنامه با Docker، دیتابیس خارجی

برنامه با Docker اجرا می‌شود اما PostgreSQL، Redis و MinIO از Environment Variable تنظیم می‌شوند:

```bash
# تنظیم متغیرهای محیطی (یا در فایل .env)
export DATABASE_URL="Host=my-server;Database=shop_db;Username=shop_user;Password=mypass"
export REDIS_URL="my-redis:6379"
export MINIO_ENDPOINT="my-minio:9000"
export MINIO_ACCESS_KEY="mykey"
export MINIO_SECRET_KEY="mysecret"

# اجرا
docker compose -f docker-compose.app.yml up -d
```

---

## ۳. حالت ۳ – اجرای محلی کامل

برای اجرای کامل محلی، به [SETUP-LOCAL.md](./SETUP-LOCAL.md) مراجعه کنید.

---

## ۴. دستورات پرکاربرد

```bash
# مشاهده وضعیت سرویس‌ها
docker compose ps

# متوقف کردن
docker compose down

# متوقف کردن + حذف volume‌ها (داده‌ها پاک می‌شوند!)
docker compose down -v

# Build مجدد یک سرویس
docker compose build backend
docker compose up -d backend

# مشاهده لاگ‌ها
docker compose logs -f --tail=100 backend

# ورود به shell کانتینر
docker compose exec backend sh
docker compose exec postgres psql -U shop_user -d shop_db

# پشتیبان‌گیری از دیتابیس
docker compose exec postgres pg_dump -U shop_user shop_db > backup.sql

# بازیابی دیتابیس
cat backup.sql | docker compose exec -T postgres psql -U shop_user shop_db
```

---

## ۵. ساختار `docker-compose.yml`

```yaml
services:
  backend:
    build:
      context: ./src
      dockerfile: Web/Dockerfile
    ports:
      - "5000:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__Default=Host=postgres;Database=shop_db;Username=shop_user;Password=${DB_PASSWORD}
      - Redis__ConnectionString=redis:6379
      - Storage__Provider=MinIO
      - Storage__MinIO__Endpoint=minio:9000
      - Storage__MinIO__AccessKey=${MINIO_ACCESS_KEY}
      - Storage__MinIO__SecretKey=${MINIO_SECRET_KEY}
      - Jwt__Secret=${JWT_SECRET}
      - Jwt__RefreshSecret=${JWT_REFRESH_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  frontend:
    build:
      context: ./apps/frontend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NEXT_PUBLIC_API_URL=http://localhost:5000
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: shop_db
      POSTGRES_USER: shop_user
      POSTGRES_PASSWORD: ${DB_PASSWORD:-shoppassword}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U shop_user"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ACCESS_KEY:-minioadmin}
      MINIO_ROOT_PASSWORD: ${MINIO_SECRET_KEY:-minioadmin}
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data
    restart: unless-stopped

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

---

## ۶. رفع مشکلات رایج

### پورت در حال استفاده است
```bash
# پیدا کردن پروسه
lsof -i :5000
# تغییر پورت در docker-compose.yml:
ports:
  - "5001:8080"
```

### کانتینر اجرا نمی‌شود
```bash
docker compose logs backend
docker compose ps
```

### حافظه کم
```bash
docker system prune -a
```

---

> **نکته:** در محیط Windows، مسیرهای volume را با `/` بنویسید نه `\`.

