# راهنمای اجرای پروژه با Docker

> این راهنما نحوه راه‌اندازی سریع پروژه Shop Multi-Vendor را با استفاده از Docker توضیح می‌دهد.

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

## ۱. حالت توسعه (Development) – با Backend

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
DB_HOST=postgres
DB_PORT=5432
DB_NAME=shop_db
DB_USER=shop_user
DB_PASSWORD=your_strong_password

# Redis
REDIS_HOST=redis
REDIS_PORT=6379

# JWT
JWT_SECRET=your_super_secret_jwt_key_min_32_chars
JWT_REFRESH_SECRET=another_secret_for_refresh_tokens

# App
NODE_ENV=development
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:4000

# Storage (MinIO)
MINIO_ENDPOINT=minio
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# Email (Mailhog در development)
MAIL_HOST=mailhog
MAIL_PORT=1025
```

### اجرای همه سرویس‌ها

```bash
# اجرای کامل (frontend + backend + database + redis + minio)
docker compose -f docker-compose.dev.yml up -d

# مشاهده لاگ‌ها
docker compose -f docker-compose.dev.yml logs -f

# مشاهده لاگ یک سرویس خاص
docker compose -f docker-compose.dev.yml logs -f backend
```

**سرویس‌ها پس از اجرا:**

| سرویس | آدرس | توضیح |
|-------|------|--------|
| Frontend (Next.js) | http://localhost:3000 | پنل مشتری |
| Backend (NestJS) | http://localhost:4000 | API |
| API Docs (Swagger) | http://localhost:4000/api-docs | مستندات API |
| PostgreSQL | localhost:5432 | پایگاه داده |
| Redis | localhost:6379 | کش |
| MinIO Console | http://localhost:9001 | مدیریت فایل |
| Mailhog | http://localhost:8025 | ایمیل تست |
| Adminer | http://localhost:8080 | مدیریت DB |

---

## ۲. حالت توسعه – فقط Frontend (بدون Backend)

اگر فقط می‌خواهید روی رابط کاربری کار کنید:

```bash
# اجرای فقط frontend با داده‌های Mock
docker compose -f docker-compose.dev.yml up -d frontend

# یا با Mock API Server
docker compose -f docker-compose.dev.yml up -d frontend mock-api
```

**Mock API** داده‌های نمونه برمی‌گرداند و نیازی به دیتابیس ندارد.

| سرویس | آدرس |
|-------|------|
| Frontend | http://localhost:3000 |
| Mock API | http://localhost:3001 |

---

## ۳. حالت Production

```bash
# Build و اجرا برای Production
docker compose up -d --build

# بررسی وضعیت
docker compose ps

# متوقف کردن
docker compose down

# متوقف کردن + حذف volume‌ها (داده‌ها پاک می‌شوند!)
docker compose down -v
```

### تنظیمات `docker-compose.yml` (Production):
```yaml
version: '3.9'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - frontend
      - backend
    restart: unless-stopped

  frontend:
    build:
      context: ./apps/frontend
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=${BACKEND_URL}
    restart: unless-stopped

  backend:
    build:
      context: ./apps/backend
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
    env_file: .env
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
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
      MINIO_ROOT_USER: ${MINIO_ACCESS_KEY}
      MINIO_ROOT_PASSWORD: ${MINIO_SECRET_KEY}
    volumes:
      - minio_data:/data
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

### تنظیمات `docker-compose.dev.yml` (Development):
```yaml
version: '3.9'

services:
  frontend:
    build:
      context: ./apps/frontend
      dockerfile: Dockerfile.dev
    volumes:
      - ./apps/frontend:/app
      - /app/node_modules
      - /app/.next
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - NEXT_PUBLIC_API_URL=http://localhost:4000
    command: npm run dev

  mock-api:
    build:
      context: ./apps/mock-api
    ports:
      - "3001:3001"
    command: npm run start

  backend:
    build:
      context: ./apps/backend
      dockerfile: Dockerfile.dev
    volumes:
      - ./apps/backend:/app
      - /app/node_modules
    ports:
      - "4000:4000"
    env_file: .env
    command: npm run start:dev
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: shop_db
      POSTGRES_USER: shop_user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_dev_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: minioadmin
      MINIO_ROOT_PASSWORD: minioadmin
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_dev_data:/data

  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"
      - "8025:8025"

  adminer:
    image: adminer
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres_dev_data:
  minio_dev_data:
```

---

## ۴. دستورات پرکاربرد

```bash
# اجرای migration دیتابیس
docker compose exec backend npm run migration:run

# Seed دیتابیس (داده‌های نمونه)
docker compose exec backend npm run seed

# اجرای تست‌ها
docker compose exec backend npm run test
docker compose exec frontend npm run test

# Build مجدد یک سرویس خاص
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

## ۵. رفع مشکلات رایج

### پورت در حال استفاده است
```bash
# پیدا کردن پروسه
lsof -i :3000
# یا
netstat -tulpn | grep 3000

# تغییر پورت در docker-compose.dev.yml:
ports:
  - "3001:3000"  # پورت میزبان:پورت کانتینر
```

### کانتینر اجرا نمی‌شود
```bash
docker compose logs backend
docker compose ps
docker inspect shop_backend
```

### حافظه کم
```bash
# پاک کردن کانتینرهای متوقف و Image های بلااستفاده
docker system prune -a
```

---

> **نکته:** در محیط Windows، مسیرهای volume را با `/` بنویسید نه `\`.
