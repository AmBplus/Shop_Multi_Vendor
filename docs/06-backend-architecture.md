# ماژول ۶ – معماری بک‌اند (.NET 10)

> **هدف این سند:** تعریف کامل معماری بک‌اند، ساختار لایه‌ها، فناوری‌های مورد استفاده، حالت‌های اجرا و راهنمای تنظیم پروژه.
>
> **وضعیت:** سند اولیه (v1.0) – آماده برای طراحی فنی

---

## فهرست مطالب

1. [فناوری‌های استفاده‌شده](#1-فناوری‌های-استفاده‌شده)
2. [ساختار معماری پروژه](#2-ساختار-معماری-پروژه)
3. [حالت‌های اجرا (Run Modes)](#3-حالت‌های-اجرا-run-modes)
4. [راه‌اندازی دیتابیس (Auto Migration & Seed)](#4-راه‌اندازی-دیتابیس-auto-migration--seed)
5. [قراردادهای کدنویسی](#5-قراردادهای-کدنویسی)
6. [پیکربندی و محیط‌های اجرا](#6-پیکربندی-و-محیط‌های-اجرا)
7. [زیرساخت لاگ‌گذاری](#7-زیرساخت-لاگ‌گذاری)
8. [داده‌های پیش‌فرض (Seed Data)](#8-داده‌های-پیش‌فرض-seed-data)

---

## 1. فناوری‌های استفاده‌شده

| تکنولوژی | نسخه | کاربرد |
|----------|------|--------|
| ASP.NET Core | **10.0** | فریمورک وب API |
| Entity Framework Core | آخرین | ORM و Migration خودکار |
| Dapper | آخرین | کوئری‌های کارایی بالا (Read-heavy) |
| Wolverine | آخرین | CQRS و Message Bus (با قرارداد Convention-based) |
| Mapster | آخرین | Object Mapping |
| Serilog | آخرین | لاگ‌گذاری ساختاریافته |
| StackExchange.Redis | آخرین | کلاینت Redis (کش، سشن، کپچا، OTP) |
| Scalar | آخرین | مستندسازی API |
| BCrypt.Net | آخرین | هش رمز عبور |
| PostgreSQL | 16 | دیتابیس اصلی |
| Redis | 7 | کش و نشست‌ها |
| MinIO | آخرین | ذخیره‌سازی فایل (Object Storage) |
| SixLabors.ImageSharp | آخرین | پردازش تصویر (کپچا، تصاویر) |

---

## 2. ساختار معماری پروژه

معماری پروژه از الگوی **Vertical Slice** با CQRS پیروی می‌کند. چهار لایه اصلی وجود دارد:

```
Shop_Multi_Vendor/
├── src/
│   ├── AppCore/          # هسته اصلی برنامه
│   ├── Module/           # سرویس‌های خارجی و زیرساختی
│   ├── Web/              # لایه API (Controllers, Host)
│   └── Framework/        # کلاس‌های پایه مشترک
├── tests/                # تست‌های واحد و یکپارچگی
├── docker-compose.yml        # حالت ۱: همه چیز با Docker
├── docker-compose.app.yml    # حالت ۲: فقط برنامه با Docker
└── appsettings.json          # حالت ۳: اجرای محلی
```

---

### 2.1 لایه AppCore

هسته اصلی برنامه. شامل همه Feature های کسب‌وکار با الگوی Vertical Slice.

```
AppCore/
├── Auth/
│   ├── Features/
│   │   ├── Register/
│   │   │   ├── RegisterCommand.cs        # Command DTO
│   │   │   ├── RegisterCommandHandler.cs # Wolverine Handler
│   │   │   └── RegisterCommandValidator.cs
│   │   ├── Login/
│   │   ├── RefreshToken/
│   │   ├── Captcha/
│   │   │   ├── GenerateCaptchaQuery.cs
│   │   │   └── VerifyCaptchaCommand.cs
│   │   └── TwoFactor/
│   ├── Data/
│   │   ├── DbContext/
│   │   │   └── AuthDbContext.cs
│   │   ├── Mapping/
│   │   │   └── UserMappingProfile.cs     # Mapster Config
│   │   └── Seed/
│   │       ├── DefaultRolesSeed.cs
│   │       └── DefaultAdminSeed.cs
│   └── Core/
│       ├── Entities/
│       │   ├── User.cs                   # Id: long
│       │   ├── Role.cs                   # Id: long
│       │   ├── Permission.cs             # Id: long
│       │   └── Session.cs                # Id: long
│       └── ValueObjects/
│
├── Products/
│   ├── Features/
│   │   ├── CreateProduct/
│   │   ├── UpdateProduct/
│   │   └── GetProductList/               # Dapper Query
│   ├── Data/
│   └── Core/
│       └── Entities/
│           ├── Product.cs                # Id: long
│           ├── ProductMedia.cs           # Id: long
│           ├── ProductVariation.cs       # Id: long
│           └── UnitConversion.cs         # Id: long
│
├── Vendors/
├── Orders/
├── Payments/
└── Shared/
    └── DbContext/
        └── AppDbContext.cs               # DbContext مشترک
```

**قواعد Wolverine (Convention-based):**
```csharp
// Wolverine به صورت Convention نام Handler ها را تشخیص می‌دهد
// نیازی به Register دستی نیست

public class RegisterCommand { /* ... */ }

// Handler به صورت خودکار توسط Wolverine یافت می‌شود:
public class RegisterCommandHandler
{
    public async Task<RegisterResult> Handle(RegisterCommand command)
    { /* ... */ }
}
```

---

### 2.2 لایه Module

سرویس‌های خارجی و زیرساختی که توسط AppCore استفاده می‌شوند:

```
Module/
├── Caching/
│   ├── Redis/
│   │   ├── RedisConfiguration.cs         # تنظیمات Redis
│   │   └── RedisCacheService.cs          # سرویس کش Redis
│   └── Captcha/
│       └── CaptchaService.cs             # کپچای اختصاصی (Redis + ImageSharp)
│
├── Storage/
│   ├── IStorageProvider.cs               # اینترفیس مشترک
│   ├── FileSystem/
│   │   └── FileSystemStorageProvider.cs
│   ├── MinIO/
│   │   └── MinIOStorageProvider.cs
│   └── External/
│       └── ExternalStorageProvider.cs    # قابل توسعه
│
├── Sms/
│   ├── ISmsProvider.cs
│   ├── Kavenegar/
│   │   └── KavenegarSmsProvider.cs
│   └── Melipayamak/
│       └── MelipayamakSmsProvider.cs
│
├── Email/
│   ├── IEmailProvider.cs
│   └── Smtp/
│       └── SmtpEmailProvider.cs
│
└── Payment/
    ├── IPaymentGateway.cs
    ├── ZarinPal/
    └── Mellat/
```

---

### 2.3 لایه Web

لایه API و Host:

```
Web/
├── Controllers/
│   ├── AuthController.cs
│   ├── CaptchaController.cs
│   ├── ProductController.cs
│   ├── VendorController.cs
│   ├── OrderController.cs
│   ├── FileController.cs                 # سرو فایل‌ها (لینک داخلی)
│   └── AdminController.cs
│
├── Helpers/
│   ├── JwtHelper.cs
│   ├── PaginationHelper.cs
│   └── ResponseHelper.cs
│
├── Middleware/
│   ├── ExceptionHandlingMiddleware.cs
│   ├── RateLimitingMiddleware.cs
│   └── AuditMiddleware.cs
│
├── Extensions/
│   ├── ServiceCollectionExtensions.cs    # DI Registration
│   └── AppBuilderExtensions.cs
│
└── Program.cs                            # Entry Point
```

**یافتن Controller ها (Find Assembly):**
```csharp
// در Program.cs – همه Controller ها از طریق Assembly Scanning یافت می‌شوند
builder.Services.AddControllers()
    .AddApplicationPart(typeof(AuthController).Assembly);

// یا با convention scanning اگر در چند Assembly باشند:
var assemblies = AppDomain.CurrentDomain.GetAssemblies()
    .Where(a => a.GetName().Name!.StartsWith("Shop."));
builder.Services.AddControllers()
    .ConfigureApplicationPartManager(manager =>
    {
        foreach (var assembly in assemblies)
            manager.ApplicationParts.Add(new AssemblyPart(assembly));
    });
```

---

### 2.4 لایه Framework

کلاس‌های پایه مشترک که در همه لایه‌ها استفاده می‌شوند:

```
Framework/
├── Base/
│   ├── AuditableEntity.cs        # کلاس پایه با Audit Fields
│   ├── BaseEntity.cs             # Id: long
│   └── SoftDeletableEntity.cs    # IsDeleted, DeletedAt, DeletedBy
│
├── Results/
│   ├── ResultOperation.cs        # نتیجه عملیات (Success/Failure)
│   ├── ResultOperation<T>.cs     # نتیجه با داده
│   └── ErrorCode.cs              # کدهای خطای استاندارد
│
├── Utilities/
│   ├── TimeUtility.cs            # تبدیل تاریخ شمسی/میلادی
│   ├── TextUtility.cs            # پردازش رشته (فارسی/انگلیسی)
│   ├── SlugUtility.cs            # تبدیل به Slug
│   └── SecurityUtility.cs        # توابع امنیتی
│
└── Pagination/
    ├── PagedList<T>.cs
    └── PaginationParams.cs
```

**کلاس پایه AuditableEntity:**
```csharp
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

public abstract class BaseEntity
{
    public long Id { get; set; }  // همه Id ها از نوع long هستند
}
```

**کلاس ResultOperation:**
```csharp
public class ResultOperation
{
    public bool IsSuccess { get; }
    public string? Message { get; }
    public ErrorCode? Error { get; }

    public static ResultOperation Success(string? message = null) => ...;
    public static ResultOperation Failure(ErrorCode error, string? message = null) => ...;
}

public class ResultOperation<T> : ResultOperation
{
    public T? Data { get; }
    public static ResultOperation<T> Success(T data, string? message = null) => ...;
}
```

---

## 3. حالت‌های اجرا (Run Modes)

سه حالت اجرا پشتیبانی می‌شود:

### حالت ۱ – همه چیز با Docker (پیش‌فرض) ✅

همه سرویس‌ها شامل دیتابیس، Redis، MinIO و برنامه با Docker Compose اجرا می‌شوند.

```bash
# اجرای کامل
docker compose up -d

# مشاهده لاگ‌ها
docker compose logs -f backend
```

**`docker-compose.yml`:**
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
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
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

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

---

### حالت ۲ – برنامه با Docker، دیتابیس خارجی

برنامه با Docker اجرا می‌شود اما دیتابیس و Redis از طریق Environment Variable تنظیم می‌شوند.

```bash
# اجرا با env های خارجی
docker compose -f docker-compose.app.yml up -d
```

**`docker-compose.app.yml`:**
```yaml
services:
  backend:
    build:
      context: ./src
      dockerfile: Web/Dockerfile
    ports:
      - "5000:8080"
    environment:
      - ASPNETCORE_ENVIRONMENT=Production
      - ConnectionStrings__Default=${DATABASE_URL}
      - Redis__ConnectionString=${REDIS_URL}
      - Storage__Provider=${STORAGE_PROVIDER:-MinIO}
      - Storage__MinIO__Endpoint=${MINIO_ENDPOINT}
      - Storage__MinIO__AccessKey=${MINIO_ACCESS_KEY}
      - Storage__MinIO__SecretKey=${MINIO_SECRET_KEY}
      - Storage__MinIO__Bucket=${MINIO_BUCKET:-shop-files}
    restart: unless-stopped
```

**فایل `.env` (نمونه):**
```env
DATABASE_URL=Host=my-db-server;Database=shop_db;Username=shop_user;Password=mypassword
REDIS_URL=my-redis-server:6379,password=myredispassword
MINIO_ENDPOINT=my-minio-server:9000
MINIO_ACCESS_KEY=myaccesskey
MINIO_SECRET_KEY=mysecretkey
```

---

### حالت ۳ – اجرای محلی (بدون Docker)

همه چیز به صورت دستی اجرا می‌شود و تنظیمات در `appsettings.json` انجام می‌گیرد.

```bash
# اجرای محلی
cd src/Web
dotnet run
```

**`appsettings.json`:**
```json
{
  "ConnectionStrings": {
    "Default": "Host=localhost;Port=5432;Database=shop_db;Username=shop_user;Password=yourpassword"
  },
  "Redis": {
    "ConnectionString": "localhost:6379"
  },
  "Jwt": {
    "Secret": "your_jwt_secret_key_min_32_chars_here",
    "RefreshSecret": "your_refresh_secret_key_32_chars",
    "AccessTokenExpiry": "00:15:00",
    "RefreshTokenExpiry": "30.00:00:00"
  },
  "Storage": {
    "Provider": "MinIO",
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
    "TtlSeconds": 180,
    "ImageWidth": 200,
    "ImageHeight": 60
  },
  "Serilog": {
    "MinimumLevel": "Information",
    "WriteTo": [
      { "Name": "Console" },
      { "Name": "File", "Args": { "path": "logs/app-.log", "rollingInterval": "Day" } }
    ]
  }
}
```

---

## 4. راه‌اندازی دیتابیس (Auto Migration & Seed)

دیتابیس به صورت **خودکار** هنگام راه‌اندازی برنامه آماده می‌شود:

```csharp
// در Program.cs
app.UseAutoMigration(); // اجرای Migration های جدید
app.UseDataSeeding();   // اجرای Seed Data های اولیه
```

**پیاده‌سازی:**
```csharp
public static class AppBuilderExtensions
{
    public static IApplicationBuilder UseAutoMigration(this IApplicationBuilder app)
    {
        using var scope = app.ApplicationServices.CreateScope();
        var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
        
        if (db.Database.GetPendingMigrations().Any())
        {
            db.Database.Migrate();
        }
        return app;
    }

    public static IApplicationBuilder UseDataSeeding(this IApplicationBuilder app)
    {
        using var scope = app.ApplicationServices.CreateScope();
        var seeders = scope.ServiceProvider.GetServices<IDataSeeder>()
            .OrderBy(s => s.Order);
        
        foreach (var seeder in seeders)
        {
            seeder.SeedAsync().GetAwaiter().GetResult();
        }
        return app;
    }
}

public interface IDataSeeder
{
    int Order { get; }
    Task SeedAsync();
}
```

**Seed Data های پیش‌فرض (اجرا به ترتیب Order):**

| Order | Seeder | توضیح |
|-------|--------|--------|
| 1 | `DefaultRolesSeed` | نقش‌های پیش‌فرض (SuperAdmin, Admin, Vendor, Reseller, Customer, ...) |
| 2 | `DefaultPermissionsSeed` | مجوزهای پیش‌فرض |
| 3 | `DefaultAdminSeed` | کاربر SuperAdmin اولیه |
| 4 | `DefaultSettingsSeed` | تنظیمات سیستم (captcha TTL، حداکثر سایز فایل، ...) |
| 5 | `DefaultResellerTiersSeed` | سطوح اعتباری همکاران |
| 6 | `DemoDataSeed` | داده‌های نمونه برای تست (فقط در محیط Development) |

**حساب‌های پیش‌فرض پس از Seed:**
```
SuperAdmin:
  Email: superadmin@shop.local
  Phone: 09000000001
  Password: SuperAdmin@1234

Admin:
  Email: admin@shop.local
  Phone: 09000000002
  Password: Admin@1234

Vendor:
  Email: vendor@shop.local
  Phone: 09000000003
  Password: Vendor@1234

Customer:
  Email: customer@shop.local
  Phone: 09000000004
  Password: Customer@1234
```

---

## 5. قراردادهای کدنویسی

### 5.1 شناسه‌ها (ID)

```csharp
// همه شناسه‌ها از نوع long هستند
public class Product : AuditableEntity
{
    // Id: long  →  از کلاس پایه BaseEntity
    public string Name { get; set; } = string.Empty;
    public long CategoryId { get; set; }  // FK هم long
    // ...
}
```

### 5.2 الگوی CQRS با Wolverine

```csharp
// Command (تغییر وضعیت)
public record CreateProductCommand(
    string Name,
    string Slug,
    decimal Price,
    long CategoryId
);

// Handler (Convention: نام Handler باید با نام Command مطابقت داشته باشد)
public class CreateProductCommandHandler
{
    private readonly AppDbContext _db;
    public CreateProductCommandHandler(AppDbContext db) => _db = db;

    public async Task<ResultOperation<long>> Handle(CreateProductCommand cmd)
    {
        // ...
    }
}

// Query (خواندن داده)
public record GetProductsQuery(int Page, int PageSize, string? Search);

public class GetProductsQueryHandler
{
    private readonly IDbConnection _db; // Dapper برای کوئری‌های پیچیده
    
    public async Task<PagedList<ProductDto>> Handle(GetProductsQuery query)
    {
        // ...
    }
}
```

### 5.3 ساختار Response API

```json
{
  "isSuccess": true,
  "message": "عملیات با موفقیت انجام شد",
  "data": { ... },
  "errors": null
}

// در صورت خطا:
{
  "isSuccess": false,
  "message": "اطلاعات نامعتبر است",
  "data": null,
  "errors": [
    { "field": "email", "message": "ایمیل نامعتبر است" }
  ]
}
```

---

## 6. پیکربندی و محیط‌های اجرا

```
appsettings.json           → تنظیمات پایه (همیشه لود می‌شود)
appsettings.Development.json → override در محیط Development
appsettings.Production.json  → override در محیط Production
Environment Variables       → بالاترین اولویت (override همه)
```

**متغیرهای محیطی مهم:**

| متغیر | توضیح | مثال |
|-------|--------|-------|
| `ASPNETCORE_ENVIRONMENT` | محیط اجرا | `Development` / `Production` |
| `ConnectionStrings__Default` | رشته اتصال PostgreSQL | `Host=...;Database=...` |
| `Redis__ConnectionString` | آدرس Redis | `localhost:6379` |
| `Jwt__Secret` | کلید JWT | رشته حداقل ۳۲ کاراکتر |
| `Storage__Provider` | نوع ذخیره‌سازی | `MinIO` / `FileSystem` |

---

## 7. زیرساخت لاگ‌گذاری

**Serilog** با Sink های مختلف:

```csharp
// در Program.cs
builder.Host.UseSerilog((ctx, config) =>
{
    config
        .ReadFrom.Configuration(ctx.Configuration)
        .Enrich.FromLogContext()
        .Enrich.WithProperty("Application", "ShopMultiVendor")
        .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}")
        .WriteTo.File("logs/app-.log", rollingInterval: RollingInterval.Day)
        .WriteTo.Seq(ctx.Configuration["Seq:Url"] ?? "http://localhost:5341") // اختیاری
        ;
});
```

**Sink های پشتیبانی‌شده:**
- Console (همیشه فعال)
- File (همیشه فعال)
- Seq (اختیاری – برای مشاهده آنلاین لاگ‌ها)
- Graylog (اختیاری – برای محیط Production)

---

## 8. داده‌های پیش‌فرض (Seed Data)

برای اطمینان از قابلیت تست کامل سیستم از همان ابتدا، داده‌های زیر به صورت خودکار ایجاد می‌شوند:

### محصولات نمونه (فقط در Development)

```
دسته‌بندی: الکترونیک
  └── محصول: گوشی موبایل Samsung A54
        قیمت: 12,000,000 تومان
        تصاویر: 3 عکس نمونه
        موجودی: 100 عدد

دسته‌بندی: پوشاک
  └── محصول: کفش ورزشی Nike Air Max
        قیمت: 2,500,000 تومان
        تنوع: رنگ (قرمز/آبی) × سایز (40/41/42)
```

### فروشنده نمونه

```
نام تجاری: فروشگاه نمونه
وضعیت: approved
محصولات: 5 محصول پیش‌فرض
```

### تنظیمات پیش‌فرض سیستم

```json
{
  "captcha_ttl_seconds": 180,
  "max_image_size_mb": 5,
  "max_video_size_mb": 100,
  "allowed_image_types": ["jpg", "jpeg", "png", "webp"],
  "registration_methods": {
    "email_password": true,
    "mobile_otp": true
  },
  "features": {
    "social_login": false,
    "kyc": false,
    "2fa": true,
    "reseller_program": true
  }
}
```

---

> **یادداشت برای توسعه‌دهنده:**
> - همه `Id` ها باید از نوع `long` باشند
> - همه موجودیت‌ها باید از `AuditableEntity` ارث‌بری کنند (مگر موارد استثناء)
> - Seed Data ها باید Idempotent باشند (قابل اجرای مجدد بدون خطا)
> - از Wolverine Convention برای Handler ها استفاده کنید (نه Manual Registration)
> - API Response همیشه از `ResultOperation<T>` استفاده کند
