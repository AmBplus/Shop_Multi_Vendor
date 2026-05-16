# ماژول ۵ – CMS، طراحی رابط کاربری و تنظیمات سامانه

> **هدف این سند:** تعریف کامل سیستم مدیریت محتوا (CMS)، تم‌بندی و پالت رنگ، کامپوننت‌های UI استاندارد، منو چند سطحی، Swiper حرفه‌ای، تنظیمات سراسری، پنل ادمین و راهنمای اجرا.
>
> **وضعیت:** سند اولیه (v1.0) – آماده برای طراحی فنی

---

## فهرست مطالب

1. [سیستم مدیریت محتوا (CMS)](#1-سیستم-مدیریت-محتوا-cms)
2. [سیستم تم و پالت رنگ](#2-سیستم-تم-و-پالت-رنگ)
3. [کامپوننت‌های UI استاندارد](#3-کامپوننت‌های-ui-استاندارد)
4. [منو چند سطحی](#4-منو-چند-سطحی)
5. [Swiper و چیدمان محصولات](#5-swiper-و-چیدمان-محصولات)
6. [ناوبری و Header/Footer](#6-ناوبری-و-headerfooter)
7. [تنظیمات سراسری سامانه](#7-تنظیمات-سراسری-سامانه)
8. [پنل ادمین](#8-پنل-ادمین)
9. [بهینه‌سازی عملکرد و حافظه](#9-بهینه‌سازی-عملکرد-و-حافظه)
10. [معماری پیشنهادی](#10-معماری-پیشنهادی)

---

## 1. سیستم مدیریت محتوا (CMS)

### 1.1 مدیریت صفحه‌ها

| # | قابلیت | توضیح |
|---|--------|--------|
| 1 | ایجاد صفحه استاتیک | درباره ما، تماس با ما |
| 2 | ویرایشگر WYSIWYG | TipTap / Quill |
| 3 | ویرایش HTML مستقیم | برای متخصصان |
| 4 | صفحات پیش‌فرض | Home, About, Contact, FAQ |
| 5 | URL سفارشی (Slug) | `/about` `/terms` |
| 6 | Meta SEO هر صفحه | عنوان، توضیحات |
| 7 | وضعیت (Draft/Published) | – |
| 8 | زمان‌بندی انتشار | scheduled publish |
| 9 | نسخه‌بندی صفحات | rollback |
| 10 | کپی صفحه | – |

### 1.2 بلاگ

| # | قابلیت |
|---|--------|
| 11 | مقالات (Posts) |
| 12 | دسته‌بندی مقالات |
| 13 | تگ مقالات |
| 14 | نویسنده مقاله |
| 15 | تصویر شاخص مقاله |
| 16 | نظرات مقاله |
| 17 | جستجوی مقالات |
| 18 | RSS Feed |
| 19 | اشتراک خبرنامه |
| 20 | مقالات مرتبط |

### 1.3 مدیریت بنرها و بخش‌های تبلیغاتی

| # | موقعیت بنر | توضیح |
|---|------------|--------|
| 21 | هدر اصلی (Hero Slider) | Swiper – حداکثر ۱۰ اسلاید |
| 22 | بنر زیر هدر | یک یا دو بنر |
| 23 | بنر میان‌صفحه | بین ردیف‌های محصول |
| 24 | بنر سایدبار | – |
| 25 | بنر فوتر | – |
| 26 | پاپ‌آپ تبلیغاتی | با تأخیر قابل تنظیم |
| 27 | نوار اطلاع‌رسانی بالای صفحه | topbar |

**تنظیمات هر بنر:**
```yaml
banner:
  title: string
  subtitle: string
  image: url
  mobile_image: url (اختیاری)
  link: url
  link_type: [internal, external]
  bg_color: hex
  text_color: hex
  start_date: datetime
  end_date: datetime
  is_active: boolean
  sort_order: int
  target: [_self, _blank]
```

### 1.4 Page Builder (سازنده صفحه)

> برای صفحه اصلی و صفحات Landing، امکان ساخت بصری با بلوک‌ها:

| # | بلوک |
|---|------|
| 28 | Hero Section |
| 29 | محصولات ویژه |
| 30 | دسته‌بندی‌ها |
| 31 | برندها |
| 32 | بنر تبلیغاتی |
| 33 | فروشندگان برتر |
| 34 | محصولات جدید |
| 35 | Flash Sale با تایمر |
| 36 | نظرات مشتریان |
| 37 | آمار سایت |
| 38 | HTML Block سفارشی |
| 39 | Video Section |
| 40 | Newsletter Signup |

**قابلیت‌های Page Builder:**
```
- نمایش/عدم نمایش هر بلوک
- ترتیب بلوک‌ها (drag & drop)
- تنظیمات رنگ هر بلوک
- تعداد ستون‌های نمایش محصول
- انتخاب محصولات دستی یا خودکار (پرفروش/جدید/...)
```

### 1.5 مدیریت منوها

| # | قابلیت |
|---|--------|
| 41 | ایجاد منوی جدید (Header/Footer/...) |
| 42 | افزودن آیتم (صفحه/دسته/URL) |
| 43 | منو چند سطحی (نامحدود) |
| 44 | آیکون برای هر آیتم |
| 45 | Badge برای آیتم (مثلاً "جدید") |
| 46 | نمایش موبایل / دسکتاپ جداگانه |
| 47 | منوی مگا (Mega Menu) |

### 1.6 پوپ‌آپ و نوتیفیکیشن‌های CMS

| # | نوع |
|---|-----|
| 48 | Announcement Bar (نوار اطلاع‌رسانی) |
| 49 | Exit Intent Popup |
| 50 | Welcome Popup |
| 51 | Cookie Consent Banner |
| 52 | Age Verification |

---

## 2. سیستم تم و پالت رنگ

### 2.1 معماری CSS Variables

```css
/* فایل: assets/css/themes/base.css */
:root {
  /* رنگ‌های اصلی */
  --color-primary:        #3B82F6;
  --color-primary-hover:  #2563EB;
  --color-primary-light:  #EFF6FF;

  /* ثانویه */
  --color-secondary:      #6B7280;
  --color-secondary-hover:#4B5563;

  /* تأکیدی */
  --color-accent:         #F59E0B;

  /* موفقیت / هشدار / خطا / اطلاعات */
  --color-success:        #10B981;
  --color-warning:        #F59E0B;
  --color-danger:         #EF4444;
  --color-info:           #3B82F6;

  /* پس‌زمینه */
  --color-bg-primary:     #FFFFFF;
  --color-bg-secondary:   #F9FAFB;
  --color-bg-tertiary:    #F3F4F6;

  /* متن */
  --color-text-primary:   #111827;
  --color-text-secondary: #6B7280;
  --color-text-muted:     #9CA3AF;

  /* حاشیه */
  --color-border:         #E5E7EB;
  --color-border-focus:   var(--color-primary);

  /* شعاع گوشه */
  --radius-sm:  4px;
  --radius-md:  8px;
  --radius-lg:  12px;
  --radius-xl:  16px;
  --radius-full:9999px;

  /* سایه */
  --shadow-sm:  0 1px 2px rgba(0,0,0,0.05);
  --shadow-md:  0 4px 6px rgba(0,0,0,0.07);
  --shadow-lg:  0 10px 15px rgba(0,0,0,0.1);

  /* فونت */
  --font-family: 'Vazir', 'Segoe UI', sans-serif;
  --font-size-xs:  0.75rem;
  --font-size-sm:  0.875rem;
  --font-size-base:1rem;
  --font-size-lg:  1.125rem;
  --font-size-xl:  1.25rem;
  --font-size-2xl: 1.5rem;
  --font-size-3xl: 1.875rem;

  /* فاصله */
  --spacing-1: 0.25rem;
  --spacing-2: 0.5rem;
  --spacing-3: 0.75rem;
  --spacing-4: 1rem;
  --spacing-6: 1.5rem;
  --spacing-8: 2rem;
}
```

### 2.2 ۱۰ تم رنگی پیش‌فرض

| # | نام تم | Primary | Accent | توضیح |
|---|--------|---------|--------|--------|
| 1 | Ocean Blue (پیش‌فرض) | `#3B82F6` | `#F59E0B` | آبی اقیانوس |
| 2 | Emerald Green | `#10B981` | `#F97316` | سبز زمردی |
| 3 | Purple Haze | `#8B5CF6` | `#EC4899` | بنفش |
| 4 | Coral Red | `#EF4444` | `#F97316` | قرمز مرجانی |
| 5 | Slate Dark | `#1E293B` | `#38BDF8` | تیره (Dark Mode) |
| 6 | Rose Gold | `#F43F5E` | `#FB923C` | رز طلایی |
| 7 | Teal Mint | `#14B8A6` | `#A3E635` | فیروزه‌ای |
| 8 | Amber Warm | `#F59E0B` | `#10B981` | کهربایی گرم |
| 9 | Indigo Night | `#4F46E5` | `#EC4899` | نیلی شب |
| 10 | Stone Natural | `#78716C` | `#16A34A` | سنگی طبیعی |

### 2.3 تم سفارشی (Custom Theme Builder)

```typescript
interface CustomTheme {
  name: string;
  colors: {
    primary: string;       // hex
    primaryHover: string;  // auto-generated (darker 10%)
    primaryLight: string;  // auto-generated (lighter 90%)
    secondary: string;
    accent: string;
    success: string;
    warning: string;
    danger: string;
    info: string;
    bgPrimary: string;
    bgSecondary: string;
    textPrimary: string;
    textSecondary: string;
    border: string;
  };
  typography: {
    fontFamily: string;    // از لیست موجود
    borderRadius: 'sharp' | 'soft' | 'rounded'; // sm/md/lg
  };
  isDark: boolean;
}
```

**Theme Builder UI:**
```
- Color Picker برای هر متغیر رنگی
- پیش‌نمایش زنده
- Import/Export JSON
- اشتراک‌گذاری تم
- ذخیره در localStorage + ارسال به سرور (برای کاربران لاگین)
```

### 2.4 رفع مشکل Flash of Wrong Theme (FOWT)

```html
<!-- در <head> قبل از هر چیز دیگری -->
<script>
  (function() {
    var theme = localStorage.getItem('theme') || 'ocean-blue';
    var themeVars = JSON.parse(localStorage.getItem('theme-vars') || 'null');
    
    // اعمال فوری data-theme
    document.documentElement.setAttribute('data-theme', theme);
    
    // اگر تم سفارشی داشتیم
    if (themeVars) {
      var style = document.createElement('style');
      style.id = 'theme-override';
      var css = ':root{';
      Object.keys(themeVars).forEach(function(k) {
        css += k + ':' + themeVars[k] + ';';
      });
      css += '}';
      style.textContent = css;
      document.head.appendChild(style);
    }
  })();
</script>
```

```css
/* themes/ocean-blue.css */
[data-theme="ocean-blue"] {
  --color-primary: #3B82F6;
  /* ... */
}

[data-theme="emerald-green"] {
  --color-primary: #10B981;
  /* ... */
}
/* ... بقیه تم‌ها */
```

---

## 3. کامپوننت‌های UI استاندارد

> تمام کامپوننت‌ها از CSS Variables استفاده می‌کنند و با تغییر تم، تمام سامانه به‌روز می‌شود.

### 3.1 Button

```html
<!-- کلاس‌های اصلی: btn -->
<button class="btn btn-primary">ثبت سفارش</button>
<button class="btn btn-secondary">لغو</button>
<button class="btn btn-outline-primary">مشاهده</button>
<button class="btn btn-ghost">بیشتر</button>
<button class="btn btn-danger">حذف</button>
<button class="btn btn-success">تأیید</button>
<button class="btn btn-sm btn-primary">کوچک</button>
<button class="btn btn-lg btn-primary">بزرگ</button>
<button class="btn btn-primary btn-loading" disabled>
  <span class="btn-spinner"></span> در حال پردازش
</button>
<a href="#" class="btn btn-primary">لینک دکمه</a>
```

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--spacing-2);
  padding: var(--spacing-2) var(--spacing-4);
  font-family: var(--font-family);
  font-size: var(--font-size-sm);
  font-weight: 500;
  border-radius: var(--radius-md);
  cursor: pointer;
  transition: all 0.2s ease;
  border: 2px solid transparent;
  text-decoration: none;
  white-space: nowrap;
}
.btn-primary {
  background: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}
.btn-primary:hover {
  background: var(--color-primary-hover);
  border-color: var(--color-primary-hover);
}
/* ... ادامه */
```

### 3.2 Alert

```html
<div class="alert alert-success">
  <span class="alert-icon">✓</span>
  <span class="alert-message">سفارش با موفقیت ثبت شد.</span>
  <button class="alert-close">×</button>
</div>
<div class="alert alert-warning">...</div>
<div class="alert alert-danger">...</div>
<div class="alert alert-info">...</div>
```

```css
.alert {
  display: flex;
  align-items: flex-start;
  gap: var(--spacing-3);
  padding: var(--spacing-3) var(--spacing-4);
  border-radius: var(--radius-md);
  border: 1px solid transparent;
  font-size: var(--font-size-sm);
}
.alert-success {
  background: color-mix(in srgb, var(--color-success) 10%, white);
  border-color: color-mix(in srgb, var(--color-success) 30%, white);
  color: color-mix(in srgb, var(--color-success) 80%, black);
}
```

### 3.3 Badge

```html
<span class="badge badge-primary">جدید</span>
<span class="badge badge-success">موجود</span>
<span class="badge badge-danger">ناموجود</span>
<span class="badge badge-warning">محدود</span>
<span class="badge badge-lg badge-accent">۲۰٪ تخفیف</span>
<span class="badge badge-dot badge-success"></span> <!-- فقط نقطه -->
```

### 3.4 Card

```html
<div class="card">
  <div class="card-header">عنوان</div>
  <div class="card-body">محتوا</div>
  <div class="card-footer">پاورقی</div>
</div>

<div class="card card-hover card-clickable">...</div> <!-- با افکت hover -->
<div class="card card-flat">...</div> <!-- بدون سایه -->
<div class="card card-bordered">...</div> <!-- با حاشیه -->
```

### 3.5 Switch / Toggle

```html
<label class="switch">
  <input type="checkbox" class="switch-input">
  <span class="switch-track"></span>
  <span class="switch-label">فعال</span>
</label>

<!-- سایزها -->
<label class="switch switch-sm">...</label>
<label class="switch switch-lg">...</label>
```

### 3.6 Tooltip

```html
<!-- tooltip با CSS خالص -->
<button 
  class="btn-icon" 
  data-tooltip="افزودن به علاقه‌مندی‌ها" 
  data-tooltip-position="top">
  ♡
</button>
```

```css
[data-tooltip] {
  position: relative;
}
[data-tooltip]::before {
  content: attr(data-tooltip);
  position: absolute;
  bottom: calc(100% + 8px);
  left: 50%;
  transform: translateX(-50%);
  background: var(--color-text-primary);
  color: white;
  padding: 4px 8px;
  border-radius: var(--radius-sm);
  font-size: var(--font-size-xs);
  white-space: nowrap;
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.2s;
}
[data-tooltip]:hover::before { opacity: 1; }
```

### 3.7 Notification Popup (اعلان‌ها با Hover)

```html
<div class="nav-item nav-notification" tabindex="0">
  <button class="btn-icon" aria-label="اعلان‌ها">
    🔔
    <span class="badge badge-danger badge-sm">۳</span>
  </button>
  
  <!-- پاپ‌آپ اعلان - نمایش با hover یا focus -->
  <div class="notification-popup" role="dialog">
    <div class="notification-header">
      <h3>اعلان‌ها</h3>
      <button class="btn-ghost btn-sm">خواندن همه</button>
    </div>
    <div class="notification-list">
      <div class="notification-item notification-unread">
        <div class="notification-icon notification-icon--order">📦</div>
        <div class="notification-content">
          <p class="notification-title">سفارش شما ارسال شد</p>
          <p class="notification-time">۵ دقیقه پیش</p>
        </div>
      </div>
      <!-- ... -->
    </div>
    <div class="notification-footer">
      <a href="/notifications" class="btn btn-ghost btn-sm">مشاهده همه</a>
    </div>
  </div>
</div>
```

```css
.nav-notification { position: relative; }
.notification-popup {
  position: absolute;
  top: calc(100% + 8px);
  left: 0; /* در RTL: right: 0 */
  width: 360px;
  background: var(--color-bg-primary);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-lg);
  opacity: 0;
  visibility: hidden;
  transform: translateY(-8px);
  transition: all 0.2s ease;
  z-index: 1000;
}
.nav-notification:hover .notification-popup,
.nav-notification:focus-within .notification-popup {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}
```

### 3.8 Cart Popup (سبد خرید با Hover/Click)

```html
<div class="nav-item nav-cart" tabindex="0">
  <button class="btn-icon" aria-label="سبد خرید">
    🛒
    <span class="badge badge-primary badge-sm cart-count">۲</span>
  </button>
  
  <div class="cart-popup" role="dialog" aria-label="سبد خرید">
    <div class="cart-popup-header">
      <h3>سبد خرید</h3>
      <span class="cart-item-count">۲ محصول</span>
    </div>
    
    <div class="cart-popup-items">
      <div class="cart-popup-item">
        <img src="..." alt="..." class="cart-item-img">
        <div class="cart-item-details">
          <p class="cart-item-name">نام محصول</p>
          <div class="cart-item-qty">
            <button class="qty-btn">-</button>
            <span>۲</span>
            <button class="qty-btn">+</button>
          </div>
          <p class="cart-item-price">۲۵۰,۰۰۰ تومان</p>
        </div>
        <button class="cart-item-remove" aria-label="حذف">×</button>
      </div>
    </div>
    
    <div class="cart-popup-footer">
      <div class="cart-total">
        <span>جمع کل:</span>
        <strong>۵۰۰,۰۰۰ تومان</strong>
      </div>
      <div class="cart-popup-actions">
        <a href="/cart" class="btn btn-outline-primary btn-sm">مشاهده سبد</a>
        <a href="/checkout" class="btn btn-primary btn-sm">تکمیل خرید</a>
      </div>
    </div>
  </div>
</div>
```

---

## 4. منو چند سطحی

### 4.1 ساختار منو (Client-side)

```
ناوبار اصلی (RTL – راست‌چین)
├── خانه
├── محصولات ▾ (Mega Menu)
│     ├── الکترونیک ▸
│     │     ├── گوشی موبایل
│     │     ├── لپ‌تاپ
│     │     └── تبلت
│     ├── پوشاک ▸
│     │     ├── مردانه
│     │     └── زنانه
│     └── خانه و آشپزخانه
├── برندها
├── تخفیف‌ها 🔴
└── تماس با ما
```

```css
/* منو RTL – راست‌چین */
.navbar {
  direction: rtl;
  text-align: right;
}
.nav-menu {
  display: flex;
  flex-direction: row;
  justify-content: flex-end; /* راست‌چین */
  gap: var(--spacing-1);
  list-style: none;
  margin: 0;
  padding: 0;
}
```

### 4.2 منوی موبایل (Drawer)

```
☰ → باز شدن Drawer از راست

┌─────────────────────────┐
│  [لوگو]      [× بستن]   │
├─────────────────────────┤
│  [جستجو]                │
├─────────────────────────┤
│  خانه                   │
│  محصولات        ˂       │  ← کلیک باز کردن submenu
│    ↓ (submenu)          │
│      الکترونیک  ˂       │
│      پوشاک      ˂       │
│  برندها                 │
│  تخفیف‌ها               │
│  تماس با ما             │
├─────────────────────────┤
│  [ورود]  [ثبت نام]      │
└─────────────────────────┘
```

### 4.3 منوی ادمین (با اسکرول)

```
پنل مدیریت
┌─────────────────────────┐
│  [لوگو]                 │
├─────────────────────────┤
│  داشبورد               │
├─────────────────────────┤
│  📦 محصولات      ▾      │  ← بخش‌های تاشو
│    لیست محصولات         │
│    دسته‌بندی‌ها          │
│    برندها               │
│    اتریبیوت‌ها           │
├─────────────────────────┤
│  🛒 سفارشات      ▾      │
│    همه سفارشات          │
│    مرجوعی‌ها             │
│ ...                     │
│ (overflow-y: auto)      │  ← اسکرول
│ ...                     │
│  📊 گزارشات      ▾      │
│  ⚙️ تنظیمات      ▾      │
└─────────────────────────┘
```

```css
.admin-sidebar {
  height: 100vh;
  overflow-y: auto;
  overflow-x: hidden;
  position: sticky;
  top: 0;
  /* Custom scrollbar */
  scrollbar-width: thin;
  scrollbar-color: var(--color-border) transparent;
}
```

---

## 5. Swiper و چیدمان محصولات

### 5.1 Swiper در موبایل

```javascript
// در موبایل: محصولات به صورت swiper افقی
// در دسکتاپ: grid عادی

import Swiper from 'swiper';
import { Navigation, Pagination, Autoplay, Lazy } from 'swiper/modules';

const productSwiper = new Swiper('.product-swiper', {
  modules: [Navigation, Pagination, Autoplay, Lazy],
  
  // موبایل: ۱.۵ محصول نمایش
  slidesPerView: 1.5,
  spaceBetween: 16,
  centeredSlides: false,
  
  breakpoints: {
    480: { slidesPerView: 2.2, spaceBetween: 16 },
    768: { slidesPerView: 3,   spaceBetween: 20 },
    1024: { slidesPerView: 4,  spaceBetween: 24 },
    1280: { slidesPerView: 5,  spaceBetween: 24 },
  },
  
  pagination: {
    el: '.swiper-pagination',
    clickable: true,
    dynamicBullets: true,
  },
  
  navigation: {
    nextEl: '.swiper-button-next',
    prevEl: '.swiper-button-prev',
  },
  
  lazy: {
    loadPrevNext: true,
    loadPrevNextAmount: 2,
  },
  
  rtl: true, // پشتیبانی از راست به چپ
});
```

```css
/* در موبایل: swiper */
@media (max-width: 767px) {
  .product-grid {
    display: none;
  }
  .product-swiper {
    display: block;
  }
}
/* در دسکتاپ: grid */
@media (min-width: 768px) {
  .product-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
    gap: var(--spacing-6);
  }
  .product-swiper {
    display: none;
  }
}
```

### 5.2 Hero Slider (صفحه اصلی)

```javascript
const heroSwiper = new Swiper('.hero-slider', {
  modules: [Autoplay, Pagination, Navigation, EffectFade],
  effect: 'fade',
  autoplay: { delay: 5000, disableOnInteraction: false },
  loop: true,
  rtl: true,
  pagination: { el: '.hero-pagination', clickable: true },
  navigation: {
    nextEl: '.hero-next',
    prevEl: '.hero-prev',
  },
});
```

### 5.3 Swiper تم و استایل سفارشی

```css
/* دکمه‌های ناوبری Swiper با تم سامانه */
.swiper-button-next,
.swiper-button-prev {
  color: var(--color-primary);
  background: var(--color-bg-primary);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-full);
  width: 40px;
  height: 40px;
  box-shadow: var(--shadow-md);
}
.swiper-button-next:hover,
.swiper-button-prev:hover {
  background: var(--color-primary);
  color: white;
}
.swiper-pagination-bullet-active {
  background: var(--color-primary);
}
```

---

## 6. ناوبری و Header/Footer

### 6.1 ساختار Header

```
┌────────────────────────────────────────────────────────┐
│  [Topbar: اطلاع‌رسانی / تماس / ارز]                   │
├────────────────────────────────────────────────────────┤
│  [لوگو]  [🔍 جستجو────────────────]  [♡][🛒][👤][☰]  │
├────────────────────────────────────────────────────────┤
│  [منوی اصلی – راست‌چین: خانه | محصولات ▾ | برندها...] │
└────────────────────────────────────────────────────────┘
```

**نکات:**
- آیکون علاقه‌مندی‌ها: tooltip با hover (`data-tooltip="علاقه‌مندی‌ها"`)
- آیکون سبد خرید: cart popup با hover
- آیکون اعلان‌ها: notification popup با hover
- منوی کاربر: dropdown با hover
- **تغییر تم از صفحه فرود حذف می‌شود** (فقط از تنظیمات پروفایل)

### 6.2 ساختار Footer

```
┌────────────────────────────────────────────────────────┐
│  [لوگو + توضیح]  [لینک‌ها]  [دسترسی سریع]  [تماس]    │
├────────────────────────────────────────────────────────┤
│  [خبرنامه: ایمیل خود را وارد کنید...]                  │
├────────────────────────────────────────────────────────┤
│  [شبکه‌های اجتماعی]   [کپی‌رایت]   [نمادهای اعتماد]  │
└────────────────────────────────────────────────────────┘
```

---

## 7. تنظیمات سراسری سامانه

### 7.1 تنظیمات عمومی

| # | تنظیم | نوع |
|---|-------|-----|
| 1 | نام سایت | text |
| 2 | لوگو | image |
| 3 | Favicon | image |
| 4 | زبان پیش‌فرض | select |
| 5 | ارز پیش‌فرض | select |
| 6 | منطقه زمانی | select |
| 7 | ایمیل مدیر | email |
| 8 | شماره تماس | phone |
| 9 | آدرس | textarea |
| 10 | توضیح کوتاه سایت | text |

### 7.2 تنظیمات ظاهری

| # | تنظیم |
|---|-------|
| 11 | تم رنگی پیش‌فرض سایت |
| 12 | اجازه تغییر تم توسط کاربر |
| 13 | فونت اصلی |
| 14 | حالت تاریک/روشن پیش‌فرض |
| 15 | تعداد محصول در هر ردیف (دسکتاپ) |
| 16 | تعداد محصول در هر ردیف (تبلت) |

### 7.3 تنظیمات فروشگاه

| # | تنظیم | پیش‌فرض |
|---|-------|--------|
| 17 | حالت چند فروشنده | فعال |
| 18 | تأیید خودکار فروشندگان | غیرفعال |
| 19 | تأیید خودکار محصولات | غیرفعال |
| 20 | حداقل قیمت محصول | ۰ |
| 21 | کمیسیون پیش‌فرض | ۸٪ |
| 22 | ارز رابط پرداخت | تومان |
| 23 | مالیات ارزش افزوده | ۱۰٪ |
| 24 | ارسال رایگان بالای | ۵۰۰,۰۰۰ تومان |
| 25 | مهلت مرجوعی | ۷ روز |

### 7.4 Feature Flags

```yaml
features:
  # ماژول‌های اصلی
  multi_vendor: true
  multi_warehouse: true
  digital_products: true
  subscription_products: false
  
  # پرداخت
  cod_payment: true
  wallet_payment: true
  installment_payment: false
  crypto_payment: false
  
  # احراز هویت
  social_login: false  # نیاز به config
  two_factor_auth: true
  kyc: false           # نیاز به سرویس خارجی
  
  # پیشرفته
  ai_recommendations: false
  price_comparison: false
  loyalty_program: true
  affiliate_program: false
  
  # CMS
  blog: true
  page_builder: true
  mega_menu: true
  
  # فروشنده
  vendor_chat: true
  vendor_api_access: false
```

---

## 8. پنل ادمین

### 8.1 داشبورد ادمین

```
┌──────────────────────────────────────────────────────────┐
│  فروش امروز  │  سفارشات جدید  │  کاربران جدید  │  درآمد  │
├──────────────────────────────────────────────────────────┤
│  نمودار فروش ۳۰ روز (line chart)                         │
├──────────────────────────────────────────────────────────┤
│  سفارشات اخیر             │  فروشندگان در انتظار تأیید   │
├──────────────────────────────────────────────────────────┤
│  محصولات بدون موجودی      │  گزارش‌های مالی خلاصه        │
└──────────────────────────────────────────────────────────┘
```

### 8.2 بخش‌های پنل ادمین

| # | بخش |
|---|-----|
| 1 | داشبورد |
| 2 | مدیریت کاربران |
| 3 | مدیریت فروشندگان |
| 4 | مدیریت محصولات |
| 5 | مدیریت سفارشات |
| 6 | مدیریت مرجوعی |
| 7 | مدیریت مالی |
| 8 | مدیریت تخفیف‌ها و کمپین |
| 9 | مدیریت انبارها |
| 10 | مدیریت محتوا (CMS) |
| 11 | مدیریت نظرات |
| 12 | تیکت‌های پشتیبانی |
| 13 | گزارشات پیشرفته |
| 14 | تنظیمات سیستم |
| 15 | لاگ‌ها و Audit Trail |

---

## 9. بهینه‌سازی عملکرد و حافظه

### 9.1 اشتراک‌گذاری Layout (نه بازنویسی)

**اشتباه (تکرار Layout):**
```jsx
// ❌ هر صفحه Layout را دوباره رندر می‌کند
function ProductPage() {
  return (
    <>
      <Header />
      <Sidebar />
      <main>...</main>
      <Footer />
    </>
  );
}
```

**درست (Layout ثابت):**
```jsx
// ✅ Layout یک بار رندر می‌شود - فقط محتوا عوض می‌شود

// layouts/MainLayout.tsx
export default function MainLayout({ children }) {
  return (
    <>
      <Header />
      <main id="page-content">{children}</main>
      <Footer />
    </>
  );
}

// app/layout.tsx (Next.js)
export default function RootLayout({ children }) {
  return (
    <html lang="fa" dir="rtl">
      <head>...</head>
      <body>
        <MainLayout>{children}</MainLayout>
      </body>
    </html>
  );
}
```

### 9.2 تکنیک‌های بهینه‌سازی

| # | تکنیک | توضیح |
|---|-------|--------|
| 1 | Code Splitting | هر route جداگانه بارگذاری شود |
| 2 | React.memo | جلوگیری از رندر مجدد کامپوننت‌های ثابت |
| 3 | useMemo/useCallback | کش‌کردن محاسبات سنگین |
| 4 | Virtual Scrolling | برای لیست‌های طولانی |
| 5 | Image Lazy Loading | intersection observer |
| 6 | SWR / React Query | کش API requests |
| 7 | Bundle Analysis | آنالیز حجم باندل |
| 8 | Tree Shaking | حذف کد استفاده نشده |
| 9 | CSS Purge | حذف CSS استفاده نشده |
| 10 | Service Worker | کش مرورگر |

### 9.3 مدیریت State

```
Global State (Redux/Zustand):
  - اطلاعات کاربر (auth)
  - سبد خرید (cart)
  - تم (theme)
  - تنظیمات سامانه (settings)

Local State (useState):
  - حالت‌های UI کامپوننت
  - فرم‌ها

Server State (React Query):
  - محصولات
  - سفارشات
  - هر داده‌ای از API
```

---

## 10. معماری پیشنهادی

### 10.1 Stack فناوری

```
Frontend:
  - Framework: Next.js 14+ (App Router)
  - Language: TypeScript
  - Styling: CSS Modules + CSS Variables (بدون Tailwind برای آزادی تم)
  - Icons: React Icons / Lucide
  - Slider: Swiper.js
  - State: Zustand + React Query
  - Forms: React Hook Form + Zod

Backend:
  - Framework: NestJS یا Express
  - Language: TypeScript
  - ORM: Prisma / TypeORM
  - Database: PostgreSQL
  - Cache: Redis
  - Queue: Bull (Redis-based)
  - Storage: MinIO / S3
  - Search: Meilisearch

Infrastructure:
  - Container: Docker
  - Web Server: Nginx
  - SSL: Let's Encrypt
```

### 10.2 ساختار پوشه‌بندی پروژه

```
shop-multi-vendor/
├── apps/
│   ├── frontend/          # Next.js
│   │   ├── app/
│   │   │   ├── (shop)/    # صفحات فروشگاه
│   │   │   ├── (auth)/    # ورود و ثبت‌نام
│   │   │   ├── (vendor)/  # پنل فروشنده
│   │   │   └── (admin)/   # پنل ادمین
│   │   ├── components/
│   │   │   ├── ui/        # کامپوننت‌های پایه (Button, Alert, ...)
│   │   │   ├── layout/    # Header, Footer, Sidebar
│   │   │   ├── product/   # کارت محصول، ...
│   │   │   └── ...
│   │   ├── styles/
│   │   │   ├── base.css
│   │   │   ├── components/
│   │   │   └── themes/    # تم‌های رنگی
│   │   └── public/
│   │       └── fonts/     # Vazir
│   │
│   └── backend/           # NestJS
│       ├── src/
│       │   ├── auth/
│       │   ├── users/
│       │   ├── vendors/
│       │   ├── products/
│       │   ├── orders/
│       │   ├── payments/
│       │   └── cms/
│       └── ...
│
├── packages/
│   └── shared/            # types, utils, constants
│
├── docs/                  # مستندات (همین فایل‌ها)
├── docker-compose.yml
├── docker-compose.dev.yml
├── SETUP-DOCKER.md
└── SETUP-LOCAL.md
```

### 10.3 فونت وزیر

```css
/* styles/fonts.css */
@font-face {
  font-family: 'Vazir';
  src: url('/fonts/Vazir.woff2') format('woff2'),
       url('/fonts/Vazir.woff')  format('woff');
  font-weight: normal;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: 'Vazir';
  src: url('/fonts/Vazir-Bold.woff2') format('woff2'),
       url('/fonts/Vazir-Bold.woff')  format('woff');
  font-weight: bold;
  font-style: normal;
  font-display: swap;
}

:root {
  --font-family: 'Vazir', 'Tahoma', sans-serif;
}

body {
  font-family: var(--font-family);
  direction: rtl;
}
```

---

> **یادداشت:** این ماژول شامل تمام جنبه‌های بصری و تجربه کاربری سامانه است. پیش از شروع توسعه، Design System باید به صورت Storybook یا Figma مستند شود تا تمام توسعه‌دهندگان از کامپوننت‌های یکسان استفاده کنند.
