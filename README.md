# Shop Multi-Vendor – مستندات طراحی سیستم (RDP)

سامانه فروشگاهی چند فروشنده با قابلیت مقیاس‌پذیری از ساده‌ترین تا پیچیده‌ترین حالت.

---

## 📂 مستندات طراحی (RDP)

تمام نیازمندی‌ها و تحلیل سیستم در ۷ سند مجزا در پوشه [`docs/`](./docs) نگهداری می‌شوند:

| # | سند | محتوا |
|---|-----|--------|
| 1 | [احراز هویت و کاربران](./docs/01-auth-and-users.md) | ثبت‌نام، ورود، 2FA، کپچای اختصاصی، نقش‌ها (شامل همکار)، پروفایل |
| 2 | [مدیریت فروشندگان](./docs/02-vendor-management.md) | پنل فروشنده، Storefront، مالی، ارتباطات، گزارشات |
| 3 | [مدیریت محصولات](./docs/03-product-management.md) | محصول، تبدیل واحد، رسانه، قیمت‌گذاری (همکاران/گروه‌ها)، انبار، SEO |
| 4 | [سفارشات و پرداخت](./docs/04-orders-and-payments.md) | سبد خرید، Checkout، سفارشات، ارسال، مرجوعی، وفاداری |
| 5 | [CMS و تنظیمات](./docs/05-cms-and-settings.md) | تم‌بندی، فونت‌های فارسی، UI Components، منو، ادمین |
| 6 | [معماری بک‌اند (.NET 10)](./docs/06-backend-architecture.md) | ساختار لایه‌ها، حالت‌های اجرا، Migration، Seed Data |
| 7 | [راهنمای فرانت‌اند (Next.js)](./docs/07-frontend-nextjs.md) | امنیت، لایوت‌ها، Responsive، مدیریت حافظه، کامپوننت‌ها |

---

## 🚀 راه‌اندازی سریع

| روش | راهنما |
|-----|--------|
| 🐳 با Docker | [SETUP-DOCKER.md](./SETUP-DOCKER.md) |
| 💻 بدون Docker | [SETUP-LOCAL.md](./SETUP-LOCAL.md) |

---

## ✨ ویژگی‌های کلیدی

- **چند فروشنده:** یک محصول توسط چند فروشنده با قیمت‌های مختلف عرضه می‌شود
- **همکاران فروش:** نقش همکار با سطوح اعتباری و قیمت‌های ویژه
- **چند انباری:** مدیریت موجودی در چندین انبار مجزا
- **تبدیل واحد:** ثبت موجودی به صورت بسته‌بندی (جین، کارتن، کامیون، ...)
- **تم‌پذیر:** ۱۰ تم رنگی پیش‌فرض + ویرایشگر تم با پیش‌نمایش زنده
- **فونت فارسی:** وزیر، شبنم، صمیم، تنها – تم مجزا برای ادمین و کاربر
- **مقیاس‌پذیر:** از فروشگاه ساده تک‌فروشنده تا مارکت‌پلیس کامل
- **Feature Flags:** فعال/غیرفعال‌سازی هر ماژول
- **RTL نیتیو:** پشتیبانی کامل از راست به چپ
- **موبایل‌فرست:** Swiper برای نمایش محصولات در موبایل

---

## 🗂 ساختار پروژه (پیشنهادی)

```
Shop_Multi_Vendor/
├── src/
│   ├── AppCore/           # هسته: Features, DbContext, Entities, Seed
│   ├── Module/            # سرویس‌های خارجی: SMS, Storage, Payment, Redis
│   ├── Web/               # API: Controllers, Middleware, Program.cs
│   └── Framework/         # پایه: ResultOperation, AuditableEntity, Utils
├── apps/
│   └── frontend/          # Next.js 16 (TypeScript)
├── tests/                 # تست‌های واحد و یکپارچگی
├── docs/                  # مستندات RDP
├── docker-compose.yml         # حالت ۱: همه با Docker
├── docker-compose.app.yml     # حالت ۲: فقط برنامه با Docker
├── SETUP-DOCKER.md
├── SETUP-LOCAL.md
└── README.md
```

---

## 🛠 فناوری‌های اصلی

**Backend (.NET 10)**

| تکنولوژی | کاربرد |
|----------|--------|
| ASP.NET Core 10 | Web API |
| EF Core + Dapper | ORM و کوئری |
| Wolverine | CQRS و Message Bus |
| PostgreSQL 16 | دیتابیس اصلی |
| Redis 7 | کش، سشن، کپچا |
| MinIO | ذخیره‌سازی فایل |
| Serilog | لاگ‌گذاری |
| Scalar | مستندات API |

**Frontend (Next.js 16)**

| تکنولوژی | کاربرد |
|----------|--------|
| Next.js 16.2 + React 19 | فریمورک |
| TypeScript 5.7 | زبان |
| Tailwind CSS 4 | استایل |
| Zod + React Hook Form | فرم‌ها |

---

> این مخزن در حال توسعه است. مستندات RDP پایه طراحی فنی سامانه هستند.
