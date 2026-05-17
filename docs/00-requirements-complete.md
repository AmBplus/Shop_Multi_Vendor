# سند یکپارچه نیازمندی‌ها و نحوه ساخت سامانه فروشگاهی چند فروشنده

> **نسخه:** 1.0  
> **وضعیت:** سند اصلی – تجمیع همه ماژول‌ها  
> **هدف:** این سند تمام نیازمندی‌های سامانه را به صورت شماره‌گذاری‌شده و با جزئیات فنی کامل در یک فایل واحد ارائه می‌دهد.

---

## فهرست مطالب

1. [فناوری‌های مورد استفاده](#فصل-۱--فناوری‌های-مورد-استفاده)
2. [معماری بک‌اند (.NET 10)](#فصل-۲--معماری-بک‌اند-net-10)
3. [حالت‌های اجرا](#فصل-۳--حالت‌های-اجرا)
4. [احراز هویت و کاربران](#فصل-۴--احراز-هویت-و-کاربران)
5. [نقش‌ها و مجوزها](#فصل-۵--نقش‌ها-و-مجوزها)
6. [پروفایل کاربری](#فصل-۶--پروفایل-کاربری)
7. [مدیریت فروشندگان](#فصل-۷--مدیریت-فروشندگان)
8. [مدیریت محصولات](#فصل-۸--مدیریت-محصولات)
9. [سفارشات، پرداخت و ارسال](#فصل-۹--سفارشات-پرداخت-و-ارسال)
10. [CMS، تم و تنظیمات سامانه](#فصل-۱۰--cms-تم-و-تنظیمات-سامانه)
11. [راهنمای فرانت‌اند (Next.js)](#فصل-۱۱--راهنمای-فرانت‌اند-nextjs)
12. [داده‌های پیش‌فرض و Seed](#فصل-۱۲--داده‌های-پیش‌فرض-و-seed)

---

---

## فصل ۱ – فناوری‌های مورد استفاده

### بک‌اند

| # | تکنولوژی | نسخه | کاربرد |
|---|----------|------|--------|
| F.1 | ASP.NET Core | **10.0** | فریمورک وب API |
| F.2 | Entity Framework Core | آخرین | ORM و Migration خودکار |
| F.3 | Dapper | آخرین | کوئری‌های سنگین (Read-heavy) |
| F.4 | Wolverine | آخرین | CQRS، Message Bus (Convention-based) |
| F.5 | Mapster | آخرین | Object Mapping |
| F.6 | Serilog | آخرین | لاگ‌گذاری ساختاریافته |
| F.7 | StackExchange.Redis | آخرین | کش، سشن، کپچا، OTP |
| F.8 | Scalar | آخرین | مستندسازی API (جایگزین Swagger) |
| F.9 | BCrypt.Net | آخرین | هش رمز عبور |
| F.10 | SixLabors.ImageSharp | آخرین | پردازش تصویر (کپچا، thumbnail) |
| F.11 | PostgreSQL | 16 | دیتابیس اصلی |
| F.12 | Redis | 7 | کش و نشست‌ها |
| F.13 | MinIO | آخرین | ذخیره‌سازی فایل (Object Storage) |

### فرانت‌اند

| # | تکنولوژی | نسخه | کاربرد |
|---|----------|------|--------|
| F.14 | Next.js | **16.2.0** | فریمورک React (App Router) |
| F.15 | React | 19.0.0 | کتابخانه UI |
| F.16 | TypeScript | 5.7 | زبان تایپ‌دار |
| F.17 | Tailwind CSS | 4.0 | استایل‌دهی (با CSS Variables) |
| F.18 | Axios | 1.7.9 | کلاینت HTTP |
| F.19 | React Hook Form | 7.54 | مدیریت فرم‌ها |
| F.20 | Zod | 3.24 | اعتبارسنجی Schema |
| F.21 | Recharts | 2.15 | نمودارها |
| F.22 | next-themes | 0.4 | مدیریت تم |
| F.23 | jalaali-js | 1.2.7 | تقویم شمسی |
| F.24 | Lucide React | 0.468 | آیکون‌ها |
| F.25 | Swiper.js | آخرین | اسلایدر |
| F.26 | date-fns | 4.1 | عملیات تاریخ |
| F.27 | KaTeX | 0.16 | رندر فرمول ریاضی |

### زیرساخت

| # | تکنولوژی | کاربرد |
|---|----------|--------|
| F.28 | Docker | کانتینرسازی |
| F.29 | Docker Compose | ارکستریشن سرویس‌ها |
| F.30 | Seq | مشاهده آنلاین لاگ‌ها (اختیاری) |
| F.31 | Graylog | مدیریت لاگ در Production (اختیاری) |

---

---

## فصل ۲ – معماری بک‌اند (.NET 10)

### R.1 ساختار کلی پروژه

پروژه از الگوی **Vertical Slice** با CQRS پیروی می‌کند و به چهار لایه اصلی تقسیم می‌شود:

```
Shop_Multi_Vendor/
├── src/
│   ├── AppCore/        ← هسته: Features, DbContext, Entities, Seed
│   ├── Module/         ← سرویس‌های خارجی: SMS, Storage, Payment, Redis
│   ├── Web/            ← API: Controllers (Find Assembly), Middleware, Program.cs
│   └── Framework/      ← پایه: ResultOperation, AuditableEntity, Utils
├── apps/frontend/      ← Next.js 16
├── tests/              ← تست‌های واحد و یکپارچگی
├── docker-compose.yml      ← حالت ۱: همه چیز Docker
├── docker-compose.app.yml  ← حالت ۲: فقط برنامه Docker
└── appsettings.json        ← حالت ۳: محلی
```

### R.2 لایه AppCore

هسته اصلی برنامه. هر Feature کسب‌وکار به صورت Vertical Slice مستقل است:

```
AppCore/
├── Auth/
│   ├── Features/ (Register, Login, RefreshToken, TwoFactor, Captcha)
│   ├── Data/ (DbContext, Mapster Profile, Seed)
│   └── Core/ (User, Role, Permission, Session – همه Id: long)
├── Products/
│   ├── Features/ (CreateProduct, UpdateProduct, GetProductList)
│   ├── Data/
│   └── Core/ (Product, ProductMedia, ProductVariation, UnitConversion)
├── Vendors/
├── Orders/
├── Payments/
└── Shared/AppDbContext.cs
```

**قوانین Wolverine (Convention-based):**
```csharp
// نام Handler باید با نام Command/Query مطابقت داشته باشد
// نیازی به Register دستی نیست
public class CreateProductCommand { /* ... */ }
public class CreateProductCommandHandler
{
    public async Task<ResultOperation<long>> Handle(CreateProductCommand cmd) { }
}
```

### R.3 لایه Module

سرویس‌های خارجی و زیرساختی:

```
Module/
├── Caching/Redis/     ← RedisCacheService
├── Captcha/           ← CaptchaService (Redis + ImageSharp)
├── Storage/
│   ├── IStorageProvider.cs   ← اینترفیس مشترک
│   ├── FileSystem/
│   ├── MinIO/
│   └── External/             ← قابل توسعه
├── Sms/               ← ISmsProvider (Kavenegar, Melipayamak)
├── Email/             ← IEmailProvider
└── Payment/           ← IPaymentGateway (ZarinPal, Mellat)
```

### R.4 لایه Web

```
Web/
├── Controllers/        ← کشف خودکار از طریق Assembly Scanning
├── Helpers/
├── Middleware/         ← ExceptionHandling, RateLimiting, Audit
├── Extensions/
└── Program.cs
```

```csharp
// کشف خودکار Controller ها:
builder.Services.AddControllers()
    .AddApplicationPart(typeof(AuthController).Assembly);
```

### R.5 لایه Framework

کلاس‌های پایه مشترک:

```csharp
// BaseEntity – همه Id ها از نوع long هستند
public abstract class BaseEntity { public long Id { get; set; } }

// AuditableEntity – همه موجودیت‌ها از این ارث می‌برند
public abstract class AuditableEntity : BaseEntity
{
    public DateTime CreatedAt { get; set; }
    public long CreatedBy { get; set; }
    public DateTime? UpdatedAt { get; set; }
    public long? UpdatedBy { get; set; }
    public DateTime? DeletedAt { get; set; }
    public long? DeletedBy { get; set; }
    public bool IsDeleted { get; set; }
}

// ResultOperation – فرمت استاندارد Response API
public class ResultOperation<T>
{
    public bool IsSuccess { get; }
    public string? Message { get; }
    public T? Data { get; }
    public ErrorCode? Error { get; }
}
```

### R.6 فرمت استاندارد API Response

```json
{
  "isSuccess": true,
  "message": "عملیات با موفقیت انجام شد",
  "data": { ... },
  "errors": null
}
```

---

---

## فصل ۳ – حالت‌های اجرا

### R.7 حالت ۱ – همه چیز با Docker (پیش‌فرض) ✅

```bash
docker compose up -d
```

تمام سرویس‌ها (Backend، Frontend، PostgreSQL، Redis، MinIO) با Docker اجرا می‌شوند.

**سرویس‌ها:**

| سرویس | پورت | توضیح |
|-------|------|--------|
| Frontend | 3000 | Next.js |
| Backend | 5000 | ASP.NET Core |
| Scalar (API Docs) | 5000/scalar | مستندات API |
| PostgreSQL | 5432 | دیتابیس |
| Redis | 6379 | کش |
| MinIO Console | 9001 | مدیریت فایل |
| Adminer | 8080 | مدیریت DB |

### R.8 حالت ۲ – برنامه با Docker، دیتابیس خارجی

```bash
docker compose -f docker-compose.app.yml up -d
```

دیتابیس، Redis و MinIO از Environment Variable خوانده می‌شوند:

```env
DATABASE_URL=Host=my-server;Database=shop_db;Username=shop_user;Password=...
REDIS_URL=my-redis:6379
MINIO_ENDPOINT=my-minio:9000
```

### R.9 حالت ۳ – اجرای محلی (بدون Docker)

```bash
# پیش‌نیاز: .NET 10 SDK، PostgreSQL 16، Redis 7
cd src/Web && dotnet run
cd apps/frontend && npm run dev
```

تنظیمات در `appsettings.json`:

```json
{
  "ConnectionStrings": { "Default": "Host=localhost;..." },
  "Redis": { "ConnectionString": "localhost:6379" },
  "Storage": { "Provider": "FileSystem", "FileSystem": { "BasePath": "./uploads" } }
}
```

### R.10 Auto Migration و Seed

دیتابیس هنگام راه‌اندازی **به صورت خودکار** Migration و Seed می‌شود:

```csharp
// در Program.cs
app.UseAutoMigration(); // EF Core Migrate
app.UseDataSeeding();   // اجرای IDataSeeder ها به ترتیب Order
```

| Order | Seeder | توضیح |
|-------|--------|--------|
| 1 | DefaultRolesSeed | نقش‌های پیش‌فرض |
| 2 | DefaultPermissionsSeed | مجوزهای پیش‌فرض |
| 3 | DefaultAdminSeed | کاربر SuperAdmin اولیه |
| 4 | DefaultSettingsSeed | تنظیمات سیستم |
| 5 | DefaultResellerTiersSeed | سطوح همکاران |
| 6 | DemoDataSeed | داده نمونه (فقط Development) |

---

---

## فصل ۴ – احراز هویت و کاربران

### R.11 ثبت‌نام با ایمیل و رمز عبور

ثبت‌نام با ایمیل و رمز عبور قابل **غیرفعال‌سازی** است. می‌توان سیستم را طوری پیکربندی کرد که فقط ثبت‌نام با شماره موبایل ممکن باشد. این تنظیم از جدول `settings` کنترل می‌شود:

```json
"registration_methods": {
  "email_password": true,   // قابل غیرفعال
  "mobile_otp": true        // حداقل یکی فعال باشد
}
```

### R.12 ثبت‌نام با شماره موبایل

تأیید با OTP از طریق SMS.

### R.13 تأیید ایمیل با لینک فعال‌سازی

### R.14 تأیید شماره موبایل با کد OTP

### R.15 ورود با ایمیل/موبایل + رمز عبور (روش پایه)

### R.16 Social Login – Google (OAuth 2.0)

### R.17 Social Login – Facebook (OAuth 2.0)

### R.18 Social Login – Apple (Sign in with Apple)

### R.19 ورود بدون رمز (Magic Link) – لینک یک‌بار مصرف

> ورود بیومتریک و SSO در این مرحله مورد نیاز نیستند.

### R.20 احراز هویت دو مرحله‌ای (2FA) – Authenticator App (TOTP)

### R.21 احراز هویت دو مرحله‌ای (2FA) – کد SMS

### R.22 احراز هویت دو مرحله‌ای (2FA) – کد ایمیل

### R.23 مدیریت نشست‌های فعال

نمایش همه دستگاه‌های لاگین‌شده با اطلاعات IP، مرورگر، زمان.

### R.24 مدیریت دستگاه‌های معتمد

### R.25 تاریخچه ورود به سیستم

### R.26 هشدار ورود مشکوک (IP ناشناس، موقعیت غیرعادی)

### R.27 Session Timeout قابل تنظیم (پیش‌فرض: ۳۰ دقیقه)

### R.28 Force Logout از همه دستگاه‌ها

### R.29 قفل خودکار پس از تلاش‌های ناموفق (پیش‌فرض: ۵ بار)

### R.30 سیاست رمز عبور (طول، پیچیدگی، انقضا) – قابل تنظیم توسط ادمین

### R.31 بازیابی رمز عبور با ایمیل (لینک یک‌بار مصرف)

### R.32 بازیابی رمز عبور با SMS (کد OTP)

### R.33 تغییر رمز عبور اجباری دوره‌ای – قابل تنظیم توسط ادمین

### R.34 کپچای اختصاصی (Custom CAPTCHA)

کد تصادفی در Redis با TTL مشخص ذخیره می‌شود. تصویر به همراه Session ID به کاربر برگشت داده می‌شود.

```
POST /api/captcha/generate
← { sessionId: "uuid", imageBase64: "data:image/png;base64,..." }
   [Redis Key: captcha:{sessionId} → "ABCD1" | TTL: 3 دقیقه]

POST /api/captcha/verify
Body: { sessionId: "uuid", answer: "ABCD1" }
← 200 OK { token: "captcha_verified_token" }
   [کلید Redis پس از تأیید حذف می‌شود – یک‌بار مصرف]

GET /api/captcha/image/{sessionId}
← تصویر PNG
```

**ویژگی‌های فنی:**
- تولید تصویر سمت سرور با SixLabors.ImageSharp
- کد تصادفی ۵–۶ کاراکتر
- TTL و سایز تصویر از تنظیمات سامانه
- Session ID یکتا per-request
- یک‌بار مصرف

### R.35 Rate Limiting API (پیش‌فرض: ۱۰۰ req/min)

### R.36 IP Whitelisting/Blacklisting (قابل تنظیم در ادمین)

### R.37 Audit Log کامل فعالیت‌های کاربران

### R.38 Token Strategy

```
Access Token:  JWT، مدت ۱۵ دقیقه، در حافظه (نه localStorage)
Refresh Token: opaque، مدت ۳۰ روز، در HttpOnly Cookie
Remember Me:   Refresh Token طولانی‌تر (۹۰ روز)
```

---

---

## فصل ۵ – نقش‌ها و مجوزها

### R.39 نقش‌های پیش‌فرض سیستم

```
SuperAdmin          ← فراتر از همه؛ می‌تواند ویژگی‌های سیستمی را محدود کند
  └── Admin         ← مدیریت کامل به جز تنظیمات بحرانی
        ├── Content Manager
        ├── Support Agent
        ├── Accountant
        ├── Warehouse Manager
        └── Delivery Agent
Vendor (فروشنده)
Reseller (همکار)    ← قیمت ویژه بر اساس سطح اعتباری
Customer (مشتری)
Guest (مهمان)
```

| # | نقش | دسترسی پیش‌فرض |
|---|-----|---------------|
| R.40 | SuperAdmin | همه چیز + کنترل Feature Flag |
| R.41 | Admin | مدیریت کامل |
| R.42 | Vendor | پنل فروشنده، محصولات، سفارشات خود |
| R.43 | Reseller (همکار) | مشاهده قیمت‌های ویژه، خرید با تخفیف سطح اعتباری |
| R.44 | Customer | خرید، پروفایل، سفارشات |
| R.45 | Guest | مشاهده و جستجو |

> **نکته SuperAdmin:** نقش SuperAdmin در سطحی بالاتر از Admin است و می‌تواند برخی ویژگی‌ها را حتی از دید Admin پنهان کند. این برای سناریوی فروش سورس فروشگاه کاربرد دارد.

### R.46 نقش‌های سفارشی

ایجاد نقش‌های کاملاً سفارشی با مجوزهای دلخواه، سلسله‌مراتب نقش، وراثت مجوزها از نقش والد.

### R.47 مجوزهای موقت با تاریخ انقضا

### R.48 مجوزهای شرطی (Context-based) بر اساس IP و زمان

### R.49 تاریخچه تغییرات نقش‌ها

### R.50 قالب‌های آماده نقش و کپی نقش

---

---

## فصل ۶ – پروفایل کاربری

### R.51 اطلاعات پایه کاربر

| # | فیلد | نوع | اجباری |
|---|------|-----|--------|
| R.51.1 | نام | text | ✅ |
| R.51.2 | نام خانوادگی | text | ✅ |
| R.51.3 | نام کاربری | text unique | ✅ |
| R.51.4 | ایمیل | email unique | ✅ |
| R.51.5 | شماره موبایل | phone | ✅ |
| R.51.6 | کد ملی | text | ❌ |
| R.51.7 | تاریخ تولد | date | ❌ |
| R.51.8 | جنسیت | enum | ❌ |
| R.51.9 | تصویر پروفایل | image | ❌ |
| R.51.10 | بیوگرافی | textarea | ❌ |

**فیلدهای Audit (برای همه موجودیت‌های سیستم):**

همه موجودیت‌ها از `AuditableEntity` ارث می‌برند:

```
CreatedAt   ← زمان ایجاد
CreatedBy   ← شناسه کاربر ایجادکننده (long FK)
UpdatedAt   ← زمان آخرین ویرایش
UpdatedBy   ← شناسه کاربر ویرایش‌کننده (long? FK)
DeletedAt   ← زمان Soft Delete
DeletedBy   ← شناسه کاربر حذف‌کننده (long? FK)
IsDeleted   ← علامت حذف نرم
```

### R.52 آدرس‌های کاربر به صورت JSON

آدرس‌ها در ستون `Addresses` از نوع `jsonb` در جدول `users` ذخیره می‌شوند:

```json
[{
  "id": 1,
  "label": "خانه",
  "fullName": "علی محمدی",
  "phone": "09121234567",
  "province": "تهران",
  "city": "تهران",
  "address": "خیابان ولیعصر، پلاک ۱۰",
  "postalCode": "1234567890",
  "lat": 35.7219,
  "lng": 51.3347,
  "isDefault": true
}]
```

آدرس فعال (پیش‌فرض) با فیلد `ActiveAddressId (long?)` ردیابی می‌شود.

### R.53 مدیریت آدرس (حداکثر ۱۰ آدرس، آدرس پیش‌فرض، نقشه تعاملی)

### R.54 لیست علاقه‌مندی‌ها (Wishlist)

### R.55 محصولات مشاهده‌شده اخیر

### R.56 پیگیری قیمت محصولات

### R.57 آپلود تصویر پروفایل

```
POST /api/profile/avatar
- حداکثر حجم: 5MB (از تنظیمات)
- فرمت: jpg, png, webp
- Crop در مرورگر
- تولید سایزهای: 32, 64, 128, 256 px
- URL بازگشتی: لینک داخلی (/files/...) – نه مستقیم MinIO
```

### R.58 Storage Provider چندگانه

```csharp
public interface IStorageProvider
{
    Task<StorageResult> UploadAsync(UploadRequest request);
    Task<Stream> DownloadAsync(string path);
    Task DeleteAsync(string path);
    string GetInternalUrl(string path); // همیشه URL داخلی
}
```

| روش | وضعیت |
|-----|--------|
| FileSystem (ذخیره محلی در مسیر قابل تنظیم) | ✅ |
| MinIO (پیش‌فرض – از طریق Docker) | ✅ |
| External (سرویس خارجی سفارشی) | قابل توسعه |

> هرگز لینک مستقیم MinIO به کاربر داده نمی‌شود. همیشه از `GET /files/{path}` استفاده می‌شود.

---

---

## فصل ۷ – مدیریت فروشندگان

### R.59 فرایند ثبت‌نام فروشنده

```
کاربر درخواست می‌دهد → فرم (حقیقی/حقوقی) → بارگذاری مدارک
→ بررسی ادمین → تأیید/رد → فعال‌سازی پنل
```

### R.60 ثبت دستی فروشنده توسط ادمین

ادمین می‌تواند بدون نیاز به درخواست کاربر، مستقیماً فروشنده ثبت کند:

```
POST /api/admin/vendors/manual-register
```

### R.61 اطلاعات پایه فروشنده

نام تجاری، لوگو، بنر، توضیحات، آدرس، شناسه ملی/کد اقتصادی، مجوز کسب‌وکار، شماره تماس.

### R.62 وضعیت‌های فروشنده

```
pending → approved → suspended → blocked → rejected
```

### R.63 صفحه اختصاصی فروشنده (Storefront)

آدرس اختصاصی `/store/{vendor-slug}`، انتخاب قالب، سفارشی‌سازی رنگ، بنر اسلایدر.

### R.64 محصول مشترک چند فروشنده

```
Product (محصول پایه)
├── VendorOffer (فروشنده ۱) → قیمت مستقل
├── VendorOffer (فروشنده ۲) → قیمت مستقل
└── VendorOffer (فروشگاه اصلی)
```

اگر فقط یک فروشنده باشد، بخش "انتخاب فروشنده" نمایش داده نمی‌شود.

### R.65 کیف پول فروشنده

```sql
vendor_wallets: total_balance, available_balance, pending_balance, frozen_balance
```

### R.66 ساختار کمیسیون

کمیسیون می‌تواند ثابت، درصدی، پلکانی، بر اساس دسته‌بندی یا امتیاز فروشنده باشد.

### R.67 تسویه‌حساب (خودکار/دستی)

درخواست برداشت، انتقال به حساب بانکی (شبا)، صورت‌حساب ماهانه PDF.

### R.68 امتیاز فروشنده

بر اساس رضایت مشتری (۳۰٪)، سرعت ارسال (۲۵٪)، نرخ پاسخگویی (۲۰٪)، نرخ لغو (۱۵٪)، کیفیت (۱۰٪).

### R.69 داشبورد فروشنده

فروش امروز، سفارشات فعال، موجودی، نمودار روند ۳۰ روز، محصولات پرفروش.

### R.70 آمار و KPI فروشنده

فروش دوره‌ای، AOV، Conversion Rate، نرخ مرجوعی، NPS.

### R.71 سیستم تیکت پشتیبانی فروشنده

### R.72 چرخه تأیید محصول فروشنده

```
draft → pending_review → approved / rejected (با دلیل)
```

---

---

## فصل ۸ – مدیریت محصولات

### R.73 فیلدهای اصلی محصول

نام (فارسی/انگلیسی)، Slug، SKU، Barcode، دسته‌بندی اصلی و فرعی، برند، تگ‌ها، توضیحات کوتاه، توضیحات کامل (WYSIWYG)، ویژگی‌های کلیدی، مشخصات فنی (JSON key:value).

### R.74 اطلاعات فیزیکی محصول

طول/عرض/ارتفاع (cm)، وزن (g)، وزن حجمی، واحد اندازه‌گیری.

### R.75 تبدیل واحد (Unit Decomposition)

فروشنده می‌تواند موجودی را به صورت بسته‌بندی بزرگ وارد کند و سیستم خودکار تبدیل کند:

```
1 جین   = 12 عدد
1 کارتن = 24 عدد
1 کامیون سنگ نوع A = 20,000 کیلوگرم
```

```sql
unit_conversions
  id         bigint PK
  name       text       -- نام واحد (مثلاً "جین")
  base_unit  text       -- واحد پایه (مثلاً "عدد")
  factor     decimal    -- ضریب تبدیل (مثلاً 12)
  category   text       -- دسته (مثلاً "پارچه" / "سنگ")
  is_active  boolean
  + AuditFields
```

### R.76 وضعیت‌های محصول

```
draft → pending_review → published / rejected
scheduled (زمان‌بندی انتشار) / archived
```

### R.77 تصاویر محصول

- حداکثر ۲۰ تصویر با ترتیب‌دهی drag & drop
- هر تصویر می‌تواند **فعال/غیرفعال** باشد
- یک تصویر به عنوان **Primary** (نمایش در جزئیات محصول)
- یک تصویر به عنوان **Thumbnail** (نمایش در کارت/لیست/جستجو)
- حداکثر سایز مجاز از **تنظیمات سامانه** کنترل می‌شود
- فشرده‌سازی خودکار به WebP + JPEG fallback
- تولید سایزهای: 100، 300، 600، 1200 px

### R.78 ویدیوهای محصول

- امکان افزودن **چندین ویدیو** با ترتیب‌دهی
- یک ویدیو به عنوان **ویدیوی اصلی** تعیین می‌شود
- حداکثر سایز از **تنظیمات سامانه**
- آپلود مستقیم یا لینک YouTube/Vimeo

### R.79 سایر فایل‌های رسانه‌ای

انواع فایل مجاز (PDF، ZIP، DWG/CAD و غیره) توسط **SuperAdmin** از پنل ادمین فعال/غیرفعال می‌شوند.

### R.80 انواع قیمت

قیمت پایه، قیمت فروش (Sale)، قیمت قبل از تخفیف (strike-through)، قیمت عمده‌فروشی، قیمت ویژه اعضا.

### R.81 قیمت‌گذاری پیشرفته

قیمت بر اساس نقش، قیمت پلکانی (Tier Pricing)، قیمت بر اساس وزن/حجم/مساحت، قیمت زمان‌بندی‌شده، قیمت بر اساس موقعیت جغرافیایی، چند ارزه.

### R.82 قیمت‌گذاری همکاران (Reseller Pricing)

همکاران بر اساس سطح اعتباری تخفیف دریافت می‌کنند:

```
سطح اعتباری ۱ → ۵٪ تخفیف
سطح اعتباری ۲ → ۸٪ تخفیف
سطح اعتباری ۳ → ۱۲٪ تخفیف
سطح اعتباری ۴ → ۱۵٪ تخفیف + شرایط ویژه
```

```sql
reseller_tiers
  id, name, credit_level (int), discount_pct (decimal),
  description, is_active, + AuditFields
```

### R.83 گروه‌های قیمتی (Price Groups)

گروه‌های قیمتی سراسری در سطح سیستم تعریف می‌شوند. هر محصول می‌تواند به یک گروه اختصاص یابد. با یک دکمه، تنظیمات گروه برای یک محصول override می‌شود.

```sql
price_groups: id, name, discount_type (percentage|fixed), discount_value,
              applicable_to (jsonb), is_active, + AuditFields

product_price_group: product_id, price_group_id,
                     is_override (bool) -- غیرفعال کردن گروه برای این محصول
```

### R.84 مالیات

VAT ۱۰٪ پیش‌فرض، محاسبه خودکار، نرخ بر اساس منطقه، معافیت مالیاتی.

### R.85 تاریخچه قیمت

ثبت همه تغییرات، هشدار کاهش قیمت برای Wishlist، نمودار تاریخچه.

### R.86 مدیریت چند انبار

```
Warehouse A (تهران) ← 150 عدد
Warehouse B (اصفهان) ← 80 عدد
Total Available: 230 عدد
```

موجودی هر انبار مستقل، انتقال بین انبارها، اولویت انبار بر اساس موقعیت مشتری.

### R.87 هشدارهای موجودی

Safety Stock، نقطه Reorder، هشدار موجودی کم/صفر.

### R.88 پیش‌سفارش (Pre-order) و Back-order

### R.89 اتریبیوت‌ها و تنوع محصول

اتریبیوت‌های سراسری (رنگ، سایز)، دسته‌بندی‌ای یا سفارشی. هر ترکیب = یک Variation با قیمت، موجودی، SKU و تصویر مستقل.

### R.90 محصولات دیجیتال

انواع: physical / digital / hybrid / service / subscription. امکان آپلود فایل، لینک دانلود، محدودیت تعداد/زمان دانلود، لایسنس کی.

### R.91 SEO محصول

Meta Title/Description، Canonical URL، Open Graph، Twitter Card، JSON-LD Schema.

### R.92 جستجوی پیشرفته

Full-text search، Instant Search، تصحیح اشتباه تایپی، فیلترهای پویا بر اساس اتریبیوت‌ها، مرتب‌سازی چندگانه.

---

---

## فصل ۹ – سفارشات، پرداخت و ارسال

### R.93 انواع سبد خرید

Guest Cart (localStorage)، User Cart (sync با سرور)، Saved Cart، Shared Cart (لینک قابل اشتراک‌گذاری).

### R.94 عملیات سبد خرید

افزودن محصول، حذف، تغییر تعداد (بررسی موجودی)، اعمال کد تخفیف، اعمال اعتبار کیف پول، انتخاب آدرس و روش ارسال.

### R.95 Cart Popup

نمایش سبد خرید با hover روی آیکون (نه کلیک). امکان تغییر تعداد و حذف در پاپ‌آپ.

### R.96 سبدهای رها شده (Abandoned Cart Recovery)

ردیابی، ایمیل یادآوری (۱ ساعت، ۲۴ ساعت)، SMS یادآوری، پیشنهاد تخفیف.

### R.97 مراحل Checkout

```
بررسی سبد → انتخاب آدرس → انتخاب روش ارسال
→ کد تخفیف/کیف پول → پرداخت → تأیید سفارش
```

پشتیبانی از One-Page و Multi-Step (قابل تنظیم).

### R.98 روش‌های پرداخت

درگاه اینترنتی (ZarinPal, Mellat, Sadad)، پرداخت در محل (COD)، کیف پول داخلی، کارت هدیه، اعتبار پنل، اقساط بانکی، رمزارز (اختیاری).

### R.99 تأیید پرداخت

> ⚠️ **امنیت:** هرگز بر اساس پارامتر callback تأیید نشود. همیشه از API درگاه پرداخت verify کنید.

```
POST /checkout/payment
→ lock inventory (Redis)
→ create order (pending_payment)
→ redirect to gateway

GET /checkout/verify?ref=...
→ verify with gateway API
→ if success: update order → paid, notify vendor (queue)
→ if fail: release lock, delete pending order
```

### R.100 چرخه عمر سفارش

```
pending_payment → paid → processing → ready_to_ship
→ shipped → out_for_delivery → delivered → completed
cancelled / refunded / on_hold
```

### R.101 سفارش چند فروشنده

یک سفارش = چند Sub-order (یکی به ازای هر فروشنده). هر Sub-order مستقل مدیریت می‌شود.

### R.102 روش‌های ارسال

پست پیشتاز/سفارشی، پیک موتوری، تیپاکس، ماهکس، ارسال توسط فروشنده، تحویل حضوری.

### R.103 ردیابی مرسوله

شماره رهگیری منحصربه‌فرد، وضعیت زنده (API پست/پیک)، SMS هنگام تغییر وضعیت.

### R.104 مرجوعی و استرداد

مهلت ۷ روز (قابل تنظیم)، فرایند: ثبت درخواست → بررسی فروشنده → تأیید → ارسال → استرداد وجه.

استرداد به روش پرداخت اولیه، کیف پول (فوری) یا کارت بانکی (۳-۷ روز).

### R.105 فاکتور رسمی PDF

فاکتور چند فروشنده (برای یک سفارش)، ارسال به ایمیل، شماره پیگیری منحصربه‌فرد.

### R.106 کدهای تخفیف

انواع: درصدی، مبلغ ثابت، ارسال رایگان، Buy X Get Y، Bundle، Flash Sale، کد معرف.

تنظیمات: تاریخ شروع/پایان، حداکثر استفاده، حداقل سفارش، محدود به دسته/محصول/کاربر.

### R.107 کمپین‌های بازاریابی

Flash Sale، جشنواره فصلی، تولد کاربر، Black Friday، کمپین ایمیل/SMS/Push.

### R.108 نظرات و امتیازدهی محصول

امتیاز ۱-۵ ستاره، متن نظر، آپلود تصویر (حداکثر ۵)، تأیید خرید (Verified Purchase)، پاسخ فروشنده.

### R.109 برنامه وفاداری

| رویداد | امتیاز |
|--------|--------|
| خرید | ۱ به ازای هر ۱۰,۰۰۰ تومان |
| ثبت نظر | ۵۰ |
| اولین خرید | ۲۰۰ |
| معرفی دوست | ۵۰۰ |

سطوح VIP: Bronze / Silver / Gold / Platinum.

---

---

## فصل ۱۰ – CMS، تم و تنظیمات سامانه

### R.110 متغیرهای CSS پایه

```css
:root {
  --color-primary, --color-secondary, --color-accent,
  --color-success, --color-warning, --color-danger, --color-info,
  --color-bg-primary, --color-bg-secondary, --color-text-primary,
  --color-border, --radius-sm/md/lg/xl/full,
  --shadow-sm/md/lg, --font-family, --spacing-*
}
```

### R.111 ۱۰ تم رنگی پیش‌فرض

| # | نام تم | Primary |
|---|--------|---------|
| 1 | Ocean Blue (پیش‌فرض) | #3B82F6 |
| 2 | Emerald Green | #10B981 |
| 3 | Purple Haze | #8B5CF6 |
| 4 | Coral Red | #EF4444 |
| 5 | Slate Dark | #1E293B |
| 6 | Rose Gold | #F43F5E |
| 7 | Teal Mint | #14B8A6 |
| 8 | Amber Warm | #F59E0B |
| 9 | Indigo Night | #4F46E5 |
| 10 | Stone Natural | #78716C |

### R.112 ویرایش تم‌های پیش‌فرض

کاربر می‌تواند هر تم پیش‌فرض را **ویرایش** کند و به عنوان تم اختصاصی ذخیره کند.

### R.113 علاقه‌مندی‌های تم

امکان ستاره‌دار کردن تم. تم‌های علاقه‌مند در اول لیست نمایش داده می‌شوند.

### R.114 ویرایشگر تم با پیش‌نمایش زنده

```
┌────────────────────────────┬──────────────────────────┐
│  پنل ویرایش                │  پیش‌نمایش کلی (Live)     │
│                            │                          │
│  رنگ اصلی: [████] #3B82F6  │  [نمایش کامپوننت‌های     │
│  ╔═══════════════╗         │   اصلی در تم جاری]       │
│  ║ [ثبت سفارش]  ║         │                          │
│  ╚═══════════════╝         │                          │
│  ← پیش‌نمایش زیر هر متغیر  │                          │
│                            │                          │
│  رنگ ثانویه: [████]        │                          │
│  ╔═══════════════╗         │                          │
│  ║ [لغو]        ║         │                          │
│  ╚═══════════════╝         │                          │
└────────────────────────────┴──────────────────────────┘
```

**قوانین:**
- هر متغیر رنگی: Color Picker + پیش‌نمایش کامپوننت مرتبط زیرش
- پیش‌نمایش کلی سمت دیگر (Real-time)
- Import/Export JSON

### R.115 تم مجزا برای پنل ادمین و بخش کاربری

```
user_preferences.user_panel_theme   ← تم بخش کاربری
user_preferences.admin_panel_theme  ← تم پنل ادمین
```

### R.116 فونت‌های فارسی پیش‌فرض

| فونت | پیش‌فرض |
|------|---------|
| Vazir | ✅ |
| Shabnam | – |
| Samim | – |
| Tanha | – |

امکان تغییر فونت برای پنل ادمین و بخش کاربری به صورت **مجزا**.

### R.117 رفع FOWT (Flash of Wrong Theme)

اجرای inline script در `<head>` قبل از hydration برای اعمال فوری `data-theme`.

### R.118 کامپوننت‌های UI استاندارد

مانند Bootstrap اختصاصی – همه کامپوننت‌ها از CSS Variables استفاده می‌کنند:

```
Button (همه variant ها: primary/secondary/outline/ghost/danger)
Alert, Badge, Card, Switch/Toggle, Tooltip
Cart Popup (hover)، Notification Popup (hover)
```

### R.119 منوی چند سطحی

منوی اصلی RTL با پشتیبانی Mega Menu، منوی موبایل (Drawer از راست).

### R.120 Swiper در موبایل / Grid در دسکتاپ

در موبایل: Swiper با `rtl: true`. در دسکتاپ: CSS Grid.

### R.121 Page Builder

بلوک‌های قابل تنظیم برای صفحه اصلی: بنر اسلایدر، محصولات ویژه، دسته‌بندی‌ها، بلاگ، نمودار آمار، HTML سفارشی و غیره.

### R.122 تنظیمات عمومی سامانه

نام سایت، لوگو، Favicon، زبان پیش‌فرض، ارز، منطقه زمانی، مالیات (۱۰٪ پیش‌فرض)، مهلت مرجوعی (۷ روز)، کمیسیون پیش‌فرض.

### R.123 Feature Flags

```yaml
multi_vendor: true
two_factor_auth: true
social_login: false   # نیاز به config
kyc: false            # مرحله بعدی
loyalty_program: true
```

### R.124 پنل ادمین – بخش‌های اصلی

داشبورد، کاربران، فروشندگان، محصولات، سفارشات، مرجوعی، مالی، تخفیف‌ها، انبارها، CMS، نظرات، تیکت‌ها، گزارشات پیشرفته، تنظیمات، Audit Trail.

### R.125 لاگ‌گذاری (Serilog)

```
Console (همیشه) + File (همیشه) + Seq (اختیاری) + Graylog (اختیاری)
```

---

---

## فصل ۱۱ – راهنمای فرانت‌اند (Next.js)

### R.126 مدیریت Token (امنیت)

```
✅ Access Token فقط در حافظه React State (نه localStorage)
✅ Refresh Token در HttpOnly Cookie
✅ Auto-refresh هنگام 401 (Axios interceptor)
```

### R.127 جلوگیری از XSS

هرگز از `dangerouslySetInnerHTML` بدون DOMPurify sanitize استفاده نشود.

### R.128 CSP Headers

```typescript
// next.config.ts
Content-Security-Policy: "default-src 'self'; script-src 'self' 'unsafe-inline'; ..."
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
```

### R.129 اعتبارسنجی با Zod

همه فرم‌ها باید Schema Zod داشته باشند. استفاده از `zodResolver` در React Hook Form.

### R.130 Cleanup در useEffect

```typescript
// ✅ همیشه AbortController، clearInterval، removeEventListener در Cleanup
useEffect(() => {
  const controller = new AbortController();
  fetchData({ signal: controller.signal });
  return () => controller.abort();
}, []);
```

### R.131 جلوگیری از Memory Leak

```typescript
// ✅ بررسی mounted قبل از setState
useEffect(() => {
  let mounted = true;
  fetchProduct(id).then(data => { if (mounted) setData(data); });
  return () => { mounted = false; };
}, [id]);
```

### R.132 بهینه‌سازی Re-render

`useMemo`، `useCallback`، `React.memo`، `useTransition` برای عملیات سنگین.

### R.133 ساختار لایوت‌ها (Reuse)

```
app/
├── layout.tsx            ← Root (فونت، تم، Providers)
├── (shop)/layout.tsx     ← Header + Footer + Cart
├── (auth)/layout.tsx     ← بدون Header/Footer
├── (vendor)/layout.tsx   ← Sidebar فروشنده
└── (admin)/layout.tsx    ← Sidebar ادمین
```

### R.134 Responsive Design (Mobile-First)

```tsx
// فقط دسکتاپ:
<div className="hidden lg:block"><DesktopFilterSidebar /></div>

// فقط موبایل:
<div className="block lg:hidden"><MobileFilterDrawer /></div>

// محصولات:
<div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5">
```

### R.135 Swiper موبایل / Grid دسکتاپ

```tsx
const ProductSection = ({ products }) => (
  <>
    <div className="block lg:hidden"><ProductSwiper products={products} /></div>
    <div className="hidden lg:grid grid-cols-4 xl:grid-cols-5 gap-6">
      {products.map(p => <ProductCard key={p.id} product={p} />)}
    </div>
  </>
);
```

### R.136 سیستم کامپوننت (مانند Bootstrap اختصاصی)

```
src/components/ui/       ← Button, Input, Select, Badge, Alert, Card, Modal, ...
src/components/features/ ← ProductCard, CartPopup, CaptchaWidget, ThemeBuilder
```

### R.137 Server State با TanStack Query

```typescript
const useProducts = (filters) => useQuery({
  queryKey: ['products', filters],
  queryFn: () => fetchProducts(filters),
  staleTime: 5 * 60 * 1000,   // 5 دقیقه
  gcTime: 30 * 60 * 1000,     // 30 دقیقه در Cache
});
```

### R.138 Client State با Zustand

```typescript
// stores/authStore.ts
const useAuthStore = create((set) => ({
  user: null,
  accessToken: null,  // فقط در memory
  setAuth: (user, token) => set({ user, accessToken: token }),
  clearAuth: () => set({ user: null, accessToken: null }),
}));
```

### R.139 Server Components به عنوان پیش‌فرض

`'use client'` فقط هنگام نیاز به `useState`، `useEffect` یا event handlers.

### R.140 Code Splitting و Dynamic Import

```typescript
const ThemeBuilder = dynamic(() => import('@/components/ThemeBuilder'), {
  loading: () => <Skeleton />,
});
```

### R.141 بهینه‌سازی تصاویر

```tsx
// همیشه از next/image – هرگز از <img> مستقیم
<Image src={url} alt={alt} width={300} height={300}
  loading="lazy" sizes="(max-width:640px) 50vw, 25vw" />
```

### R.142 Font Loading بهینه

```typescript
const vazirFont = localFont({
  src: [{ path: '../public/fonts/Vazirmatn-Regular.woff2', weight: '400' }],
  variable: '--font-vazir',
  display: 'swap',
});
```

### R.143 Error Handling در صفحات

هر صفحه باید `error.tsx` و `loading.tsx` (Skeleton) داشته باشد.

### R.144 API Calls از طریق lib/api

```typescript
// ✅ هرگز مستقیم fetch در کامپوننت نزنید
export const productsApi = {
  getList: (params) => apiClient.get('/products', { params }),
  getBySlug: (slug) => apiClient.get(`/products/${slug}`),
};
```

---

---

## فصل ۱۲ – داده‌های پیش‌فرض و Seed

### R.145 حساب‌های پیش‌فرض

```
SuperAdmin: superadmin@shop.local | 09000000001 | SuperAdmin@1234
Admin:      admin@shop.local      | 09000000002 | Admin@1234
Vendor:     vendor@shop.local     | 09000000003 | Vendor@1234
Customer:   customer@shop.local   | 09000000004 | Customer@1234
```

### R.146 تنظیمات پیش‌فرض سیستم (Seed)

```json
{
  "captcha_ttl_seconds": 180,
  "max_image_size_mb": 5,
  "max_video_size_mb": 100,
  "allowed_image_types": ["jpg", "jpeg", "png", "webp"],
  "registration_methods": { "email_password": true, "mobile_otp": true },
  "features": { "social_login": false, "kyc": false, "2fa": true, "reseller_program": true }
}
```

### R.147 سطوح همکار پیش‌فرض

```
سطح ۱ – آزاد:    ۵٪ تخفیف
سطح ۲ – نقره‌ای: ۸٪ تخفیف
سطح ۳ – طلایی:   ۱۲٪ تخفیف
سطح ۴ – پلاتین:  ۱۵٪ تخفیف + شرایط ویژه
```

### R.148 داده‌های نمونه (فقط در Development)

```
دسته‌بندی: الکترونیک / پوشاک
محصولات: گوشی Samsung A54 (۱۲,۰۰۰,۰۰۰ ت)، کفش Nike Air Max (۲,۵۰۰,۰۰۰ ت)
فروشنده نمونه: "فروشگاه نمونه" – approved – 5 محصول
```

### R.149 Idempotency Seed

همه Seeder ها باید قبل از ایجاد، وجود داده را بررسی کنند تا در اجرای مجدد خطا ندهند.

---

---

## خلاصه آمار نیازمندی‌ها

| فصل | تعداد نیازمندی |
|-----|---------------|
| فناوری‌ها | F.1 – F.31 (31 آیتم) |
| معماری بک‌اند | R.1 – R.10 (10 آیتم) |
| احراز هویت | R.11 – R.38 (28 آیتم) |
| نقش‌ها | R.39 – R.50 (12 آیتم) |
| پروفایل | R.51 – R.58 (8 آیتم) |
| فروشندگان | R.59 – R.72 (14 آیتم) |
| محصولات | R.73 – R.92 (20 آیتم) |
| سفارشات و پرداخت | R.93 – R.109 (17 آیتم) |
| CMS و تنظیمات | R.110 – R.125 (16 آیتم) |
| فرانت‌اند | R.126 – R.144 (19 آیتم) |
| Seed Data | R.145 – R.149 (5 آیتم) |
| **جمع کل** | **~180 نیازمندی** |

---

## راهنمای شروع سریع

```bash
# ۱. Clone
git clone https://github.com/AmBplus/Shop_Multi_Vendor.git
cd Shop_Multi_Vendor

# ۲. تنظیم .env
cp .env.example .env
# ویرایش .env (رمز DB، JWT Secret، ...)

# ۳. اجرا (حالت ۱ – پیش‌فرض)
docker compose up -d

# ✅ سیستم راه‌اندازی شد
# Frontend:  http://localhost:3000
# Backend:   http://localhost:5000
# API Docs:  http://localhost:5000/scalar
# MinIO:     http://localhost:9001
```

---

> **مستندات تفصیلی هر ماژول:**
> - [`01-auth-and-users.md`](./01-auth-and-users.md) – احراز هویت و کاربران
> - [`02-vendor-management.md`](./02-vendor-management.md) – مدیریت فروشندگان
> - [`03-product-management.md`](./03-product-management.md) – مدیریت محصولات
> - [`04-orders-and-payments.md`](./04-orders-and-payments.md) – سفارشات و پرداخت
> - [`05-cms-and-settings.md`](./05-cms-and-settings.md) – CMS و تنظیمات
> - [`06-backend-architecture.md`](./06-backend-architecture.md) – معماری بک‌اند (.NET 10)
> - [`07-frontend-nextjs.md`](./07-frontend-nextjs.md) – راهنمای فرانت‌اند
