# ماژول ۷ – راهنمای فنی فرانت‌اند (Next.js 16)

> **هدف این سند:** تعریف استانداردها، ملاحظات فنی و امنیتی، مدیریت حافظه، چرخه کامپوننت، لایوت‌ها و طراحی Responsive برای پروژه فرانت‌اند.
>
> **وضعیت:** سند اولیه (v1.0) – آماده برای طراحی فنی

---

## فهرست مطالب

1. [فناوری‌های استفاده‌شده](#1-فناوری‌های-استفاده‌شده)
2. [ملاحظات امنیتی](#2-ملاحظات-امنیتی)
3. [مدیریت حافظه و کامپوننت](#3-مدیریت-حافظه-و-کامپوننت)
4. [ساختار لایوت‌ها (Layout Reuse)](#4-ساختار-لایوت‌ها-layout-reuse)
5. [طراحی Responsive](#5-طراحی-responsive)
6. [سیستم کامپوننت](#6-سیستم-کامپوننت)
7. [مدیریت State و Cache](#7-مدیریت-state-و-cache)
8. [Performance بهینه‌سازی](#8-performance-بهینه‌سازی)
9. [ساختار پروژه](#9-ساختار-پروژه)
10. [قراردادهای کدنویسی](#10-قراردادهای-کدنویسی)

---

## 1. فناوری‌های استفاده‌شده

| تکنولوژی | نسخه | کاربرد |
|----------|------|--------|
| Next.js | **16.2.0** | فریمورک React (App Router) |
| React | **19.0.0** | کتابخانه UI |
| TypeScript | **5.7** | زبان تایپ‌دار |
| Tailwind CSS | **4.0** | استایل‌دهی (با CSS Variables) |
| Axios | 1.7.9 | کلاینت HTTP |
| React Hook Form | 7.54 | مدیریت فرم‌ها |
| Zod | 3.24 | اعتبارسنجی Schema |
| Recharts | 2.15 | نمودارها |
| next-themes | 0.4 | مدیریت تم (Dark/Light) |
| KaTeX | 0.16 | رندر فرمول ریاضی |
| date-fns | 4.1 | عملیات تاریخ |
| jalaali-js | 1.2.7 | تقویم شمسی |
| Lucide React | 0.468 | آیکون‌ها |
| Swiper.js | آخرین | اسلایدر |

---

## 2. ملاحظات امنیتی

### 2.1 مدیریت Token

```typescript
// ✅ درست: Access Token در حافظه (نه localStorage)
// ✅ درست: Refresh Token در HttpOnly Cookie

// useAuth.ts
const useAuth = () => {
  const [accessToken, setAccessToken] = useState<string | null>(null);
  // accessToken فقط در حافظه React State نگهداری می‌شود
  // هرگز در localStorage یا sessionStorage ذخیره نمی‌شود
};

// axiosInstance.ts
axiosInstance.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// خودکار Refresh هنگام 401
axiosInstance.interceptors.response.use(
  (res) => res,
  async (error) => {
    if (error.response?.status === 401 && !error.config._retry) {
      error.config._retry = true;
      await refreshAccessToken(); // از Refresh Token در Cookie استفاده می‌کند
      return axiosInstance(error.config);
    }
    return Promise.reject(error);
  }
);
```

### 2.2 XSS Prevention

```typescript
// ✅ هرگز از dangerouslySetInnerHTML استفاده نکنید مگر با sanitize
import DOMPurify from 'dompurify';

// برای محتوای CMS که باید HTML رندر شود:
const SafeHtml = ({ content }: { content: string }) => (
  <div
    dangerouslySetInnerHTML={{
      __html: DOMPurify.sanitize(content, {
        ALLOWED_TAGS: ['p', 'br', 'b', 'i', 'strong', 'em', 'ul', 'ol', 'li', 'a', 'h1', 'h2', 'h3'],
        ALLOWED_ATTR: ['href', 'target'],
      }),
    }}
  />
);
```

### 2.3 CSRF Protection

```typescript
// Next.js Server Actions به صورت خودکار CSRF protection دارند
// برای API calls از Axios:
axiosInstance.defaults.withCredentials = true; // برای ارسال Cookie

// سرور باید CORS را فقط برای domain معتبر تنظیم کند
```

### 2.4 Content Security Policy (CSP)

```typescript
// next.config.ts
const nextConfig = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'Content-Security-Policy',
            value: [
              "default-src 'self'",
              "script-src 'self' 'unsafe-inline'", // برای Next.js inline scripts
              "style-src 'self' 'unsafe-inline'",  // برای Tailwind
              "img-src 'self' data: blob:",
              "font-src 'self'",
              "connect-src 'self' http://localhost:5000", // API
            ].join('; '),
          },
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
        ],
      },
    ];
  },
};
```

### 2.5 Input Validation (Zod)

```typescript
// هر فرم باید Schema Zod داشته باشد
const registerSchema = z.object({
  mobile: z.string().regex(/^09[0-9]{9}$/, 'شماره موبایل نامعتبر است'),
  password: z
    .string()
    .min(8, 'رمز عبور باید حداقل ۸ کاراکتر باشد')
    .regex(/[A-Z]/, 'باید حداقل یک حرف بزرگ داشته باشد')
    .regex(/[0-9]/, 'باید حداقل یک عدد داشته باشد'),
  captchaSessionId: z.string().uuid(),
  captchaAnswer: z.string().length(5),
});

// در کامپوننت فرم:
const form = useForm<z.infer<typeof registerSchema>>({
  resolver: zodResolver(registerSchema),
});
```

### 2.6 Environment Variables

```typescript
// متغیرهای NEXT_PUBLIC_ به client می‌رسند → هرگز اطلاعات حساس نگذارید
// NEXT_PUBLIC_API_URL=http://localhost:5000  ✅
// NEXT_PUBLIC_SITE_NAME=فروشگاه              ✅

// متغیرهای بدون NEXT_PUBLIC_ فقط در server هستند
// SECRET_KEY=...  ✅ (فقط server-side)

// .env.local  →  local development (gitignore)
// .env        →  default values (می‌تواند commit شود – بدون اطلاعات حساس)
```

---

## 3. مدیریت حافظه و کامپوننت

### 3.1 Cleanup در useEffect

```typescript
// ✅ همیشه Cleanup برای subscriptions، timers، event listeners
useEffect(() => {
  const controller = new AbortController();
  
  fetchProducts({ signal: controller.signal });
  
  return () => {
    controller.abort(); // لغو در Unmount
  };
}, []);

useEffect(() => {
  const timer = setInterval(() => {
    updateTime();
  }, 1000);
  
  return () => clearInterval(timer); // پاکسازی timer
}, []);

useEffect(() => {
  window.addEventListener('resize', handleResize);
  return () => window.removeEventListener('resize', handleResize);
}, []);
```

### 3.2 جلوگیری از Memory Leak

```typescript
// ✅ بررسی mounted قبل از setState
const useProductData = (id: number) => {
  const [data, setData] = useState<Product | null>(null);
  
  useEffect(() => {
    let mounted = true;
    
    fetchProduct(id).then((product) => {
      if (mounted) setData(product); // فقط اگر هنوز mounted است
    });
    
    return () => { mounted = false; };
  }, [id]);
  
  return data;
};
```

### 3.3 بهینه‌سازی Re-render

```typescript
// ✅ useMemo برای محاسبات سنگین
const filteredProducts = useMemo(() => {
  return products.filter(p => p.price >= minPrice && p.price <= maxPrice);
}, [products, minPrice, maxPrice]);

// ✅ useCallback برای توابعی که به عنوان prop پاس داده می‌شوند
const handleAddToCart = useCallback((productId: number) => {
  addToCart(productId);
}, [addToCart]);

// ✅ React.memo برای کامپوننت‌هایی که props تغییر نمی‌کنند
const ProductCard = React.memo(({ product }: { product: Product }) => {
  return <div>...</div>;
});

// ✅ useTransition برای عملیات‌های سنگین
const [isPending, startTransition] = useTransition();

const handleSearch = (query: string) => {
  startTransition(() => {
    setSearchResults(performSearch(query));
  });
};
```

### 3.4 Image Lazy Loading

```typescript
// ✅ همیشه از next/image استفاده کنید
import Image from 'next/image';

<Image
  src={product.thumbnailUrl}
  alt={product.name}
  width={300}
  height={300}
  loading="lazy"
  placeholder="blur"
  blurDataURL={product.blurHash}
/>
```

---

## 4. ساختار لایوت‌ها (Layout Reuse)

لایوت‌ها در Next.js App Router با فایل `layout.tsx` تعریف می‌شوند و به صورت Nested کار می‌کنند:

```
app/
├── layout.tsx                  # Root Layout (فونت، تم، Providers)
│
├── (shop)/                     # Route Group: فروشگاه عمومی
│   ├── layout.tsx              # ShopLayout (Header، Footer، Cart)
│   ├── page.tsx                # صفحه اصلی
│   ├── products/
│   │   ├── page.tsx            # لیست محصولات
│   │   └── [slug]/page.tsx     # جزئیات محصول
│   ├── cart/page.tsx
│   └── checkout/page.tsx
│
├── (auth)/                     # Route Group: احراز هویت
│   ├── layout.tsx              # AuthLayout (بدون Header/Footer)
│   ├── login/page.tsx
│   └── register/page.tsx
│
├── (vendor)/                   # Route Group: پنل فروشنده
│   ├── layout.tsx              # VendorLayout (Sidebar فروشنده)
│   ├── dashboard/page.tsx
│   ├── products/page.tsx
│   └── orders/page.tsx
│
└── (admin)/                    # Route Group: پنل ادمین
    ├── layout.tsx              # AdminLayout (Sidebar ادمین)
    ├── dashboard/page.tsx
    ├── users/page.tsx
    └── settings/page.tsx
```

### 4.1 Root Layout

```tsx
// app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="fa" dir="rtl" suppressHydrationWarning>
      <head>
        <ThemeInitScript /> {/* جلوگیری از FOWT */}
      </head>
      <body className={vazirFont.className}>
        <ThemeProvider>
          <AuthProvider>
            <QueryClientProvider>
              {children}
            </QueryClientProvider>
          </AuthProvider>
        </ThemeProvider>
      </body>
    </html>
  );
}
```

### 4.2 Shop Layout

```tsx
// app/(shop)/layout.tsx
export default function ShopLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen flex flex-col">
      <Header />           {/* Header با Cart Popup و Notification */}
      <AnnouncementBar />  {/* نوار اطلاع‌رسانی (اختیاری) */}
      <main className="flex-1 container mx-auto px-4 py-6">
        {children}
      </main>
      <Footer />
    </div>
  );
}
```

### 4.3 Admin Layout

```tsx
// app/(admin)/layout.tsx
export default function AdminLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen overflow-hidden">
      <AdminSidebar />    {/* Sidebar با overflow-y: auto */}
      <div className="flex-1 flex flex-col overflow-hidden">
        <AdminHeader />
        <main className="flex-1 overflow-y-auto p-6 bg-[--color-bg-secondary]">
          {children}
        </main>
      </div>
    </div>
  );
}
```

---

## 5. طراحی Responsive

### 5.1 Breakpoints (Tailwind)

```
sm  → 640px   (موبایل افقی)
md  → 768px   (تبلت)
lg  → 1024px  (لپ‌تاپ کوچک)
xl  → 1280px  (دسکتاپ)
2xl → 1536px  (مانیتور بزرگ)
```

### 5.2 کامپوننت‌های فقط دسکتاپ / فقط موبایل

```tsx
// کامپوننت فقط در lg به بالا نمایش داده می‌شود:
<div className="hidden lg:block">
  <DesktopFilterSidebar />
</div>

// کامپوننت فقط در موبایل:
<div className="block lg:hidden">
  <MobileFilterDrawer />
</div>

// جدول داده‌ها – در موبایل به کارت تبدیل می‌شود:
// Desktop (lg+):
<div className="hidden lg:block">
  <DataTable data={orders} />
</div>
// Mobile:
<div className="lg:hidden space-y-4">
  {orders.map(order => <OrderCard key={order.id} order={order} />)}
</div>
```

### 5.3 محصولات – تعداد ستون بر اساس صفحه‌نمایش

```tsx
<div className="grid grid-cols-2 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 xl:grid-cols-5 gap-4">
  {products.map(product => (
    <ProductCard key={product.id} product={product} />
  ))}
</div>
```

### 5.4 Swiper در موبایل، Grid در دسکتاپ

```tsx
const ProductSection = ({ products }: { products: Product[] }) => {
  return (
    <>
      {/* موبایل: Swiper */}
      <div className="block lg:hidden">
        <ProductSwiper products={products} />
      </div>
      
      {/* دسکتاپ: Grid */}
      <div className="hidden lg:grid grid-cols-4 xl:grid-cols-5 gap-6">
        {products.map(p => <ProductCard key={p.id} product={p} />)}
      </div>
    </>
  );
};
```

### 5.5 موارد Responsive مهم

| عنصر | موبایل | دسکتاپ |
|------|--------|--------|
| منو | Drawer از راست | منوی افقی |
| فیلتر محصولات | Bottom Sheet / Drawer | Sidebar چپ |
| جدول ادمین | کارت‌ها | جدول |
| نمودار آمار | ساده | پیشرفته |
| Cart Popup | صفحه کامل | Dropdown |
| Header | compact با burger | Full Header |

---

## 6. سیستم کامپوننت

### 6.1 اصول کلی

مانند Bootstrap اختصاصی – کامپوننت‌های اصلی باید:
- از **CSS Variables** استفاده کنند (تم‌پذیر)
- **Accessible** باشند (ARIA attributes)
- **RTL** را پشتیبانی کنند
- **TypeScript** کامل داشته باشند

```
src/components/ui/          # کامپوننت‌های پایه (مثل Bootstrap)
├── Button.tsx
├── Input.tsx
├── Select.tsx
├── Badge.tsx
├── Alert.tsx
├── Card.tsx
├── Modal.tsx
├── Drawer.tsx
├── Tooltip.tsx
├── Spinner.tsx
└── Pagination.tsx

src/components/            # کامپوننت‌های ترکیبی
├── ProductCard/
├── CartPopup/
├── Header/
├── Sidebar/
└── ThemeBuilder/
```

### 6.2 نمونه کامپوننت Button

```tsx
// components/ui/Button.tsx
import { ButtonHTMLAttributes, forwardRef } from 'react';
import { clsx } from 'clsx';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline' | 'ghost' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  isLoading?: boolean;
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', isLoading, className, children, disabled, ...props }, ref) => {
    return (
      <button
        ref={ref}
        disabled={disabled || isLoading}
        className={clsx(
          'btn',
          `btn-${variant}`,
          `btn-${size}`,
          isLoading && 'btn-loading',
          className
        )}
        {...props}
      >
        {isLoading && <span className="btn-spinner" aria-hidden="true" />}
        {children}
      </button>
    );
  }
);

Button.displayName = 'Button';
```

### 6.3 کامپوننت Captcha

```tsx
// components/Captcha/CaptchaWidget.tsx
export const CaptchaWidget = ({ onVerified }: { onVerified: (sessionId: string) => void }) => {
  const [captcha, setCaptcha] = useState<{ sessionId: string; imageUrl: string } | null>(null);
  const [answer, setAnswer] = useState('');

  const loadCaptcha = async () => {
    const res = await apiClient.post('/captcha/generate');
    setCaptcha(res.data);
    setAnswer('');
  };

  useEffect(() => {
    loadCaptcha();
  }, []);

  return (
    <div className="captcha-widget">
      {captcha && (
        <>
          <img
            src={`/api/captcha/image/${captcha.sessionId}`}
            alt="کد تصویری"
            className="captcha-image"
          />
          <button type="button" onClick={loadCaptcha} title="تغییر کد">
            <RefreshIcon />
          </button>
        </>
      )}
      <Input
        placeholder="کد تصویری را وارد کنید"
        value={answer}
        onChange={(e) => setAnswer(e.target.value)}
        maxLength={6}
      />
    </div>
  );
};
```

---

## 7. مدیریت State و Cache

### 7.1 Server State (TanStack Query / React Query)

```typescript
// برای داده‌های سرور از TanStack Query استفاده کنید
// این ابزار Cache، Loading، Error و Stale را مدیریت می‌کند

const useProducts = (filters: ProductFilters) => {
  return useQuery({
    queryKey: ['products', filters],
    queryFn: () => fetchProducts(filters),
    staleTime: 5 * 60 * 1000,  // 5 دقیقه تازه است
    gcTime: 30 * 60 * 1000,    // 30 دقیقه در Cache نگه می‌دارد
  });
};

// Mutation با optimistic update
const useAddToCart = () => {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: addItemToCart,
    onMutate: async (newItem) => {
      // Optimistic Update
      await queryClient.cancelQueries({ queryKey: ['cart'] });
      const previous = queryClient.getQueryData(['cart']);
      queryClient.setQueryData(['cart'], (old: CartData) => ({
        ...old,
        items: [...old.items, newItem],
      }));
      return { previous };
    },
    onError: (err, newItem, context) => {
      // Rollback در صورت خطا
      queryClient.setQueryData(['cart'], context?.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['cart'] });
    },
  });
};
```

### 7.2 Client State (Zustand)

```typescript
// برای state های client از Zustand استفاده کنید
// مثل: Auth Token، Theme، Cart Count، UI State

// stores/authStore.ts
interface AuthStore {
  user: User | null;
  accessToken: string | null;
  setAuth: (user: User, token: string) => void;
  clearAuth: () => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  accessToken: null,
  setAuth: (user, token) => set({ user, accessToken: token }),
  clearAuth: () => set({ user: null, accessToken: null }),
}));
```

### 7.3 کانال‌های مختلف State

| نوع State | ابزار | مثال |
|-----------|-------|-------|
| Server Data | TanStack Query | لیست محصولات، پروفایل |
| Auth / User | Zustand | Access Token، اطلاعات کاربر |
| UI State | useState / useReducer | Modal باز/بسته |
| Form State | React Hook Form | فرم ثبت‌نام |
| URL State | useSearchParams | فیلتر، صفحه‌بندی |

---

## 8. Performance بهینه‌سازی

### 8.1 Code Splitting خودکار

Next.js App Router به صورت خودکار هر صفحه را Code Split می‌کند.
برای کامپوننت‌های بزرگ از Dynamic Import استفاده کنید:

```typescript
// برای کامپوننت‌های سنگین که در اولین بار نیاز نیستند
const RichTextEditor = dynamic(() => import('@/components/RichTextEditor'), {
  loading: () => <div className="h-48 bg-gray-100 animate-pulse rounded" />,
  ssr: false, // فقط client-side
});

const ThemeBuilder = dynamic(() => import('@/components/ThemeBuilder'), {
  loading: () => <div>در حال بارگذاری ویرایشگر تم...</div>,
});
```

### 8.2 استراتژی Cache

```typescript
// app/products/page.tsx – Server Component
export const revalidate = 300; // هر 5 دقیقه revalidate

// یا per-request:
const products = await fetch('/api/products', {
  next: { revalidate: 60 } // 1 دقیقه Cache
});

// ISR برای صفحات محصول:
export async function generateStaticParams() {
  const topProducts = await getTopProducts(100);
  return topProducts.map(p => ({ slug: p.slug }));
}
```

### 8.3 تصاویر بهینه

```tsx
// ✅ همیشه از next/image استفاده کنید
<Image
  src={product.thumbnailUrl}
  alt={product.name}
  width={300}
  height={300}
  sizes="(max-width: 640px) 50vw, (max-width: 1024px) 33vw, 25vw"
  priority={index < 4}  // اولین 4 تصویر با priority لود شوند
/>
```

### 8.4 Font Loading بهینه

```typescript
// app/layout.tsx
import localFont from 'next/font/local';

const vazirFont = localFont({
  src: [
    { path: '../public/fonts/Vazirmatn-Regular.woff2', weight: '400' },
    { path: '../public/fonts/Vazirmatn-Bold.woff2', weight: '700' },
  ],
  variable: '--font-vazir',
  display: 'swap',
  preload: true,
});
```

---

## 9. ساختار پروژه

```
apps/frontend/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # Root Layout
│   ├── (shop)/             # فروشگاه عمومی
│   ├── (auth)/             # صفحات احراز هویت
│   ├── (vendor)/           # پنل فروشنده
│   └── (admin)/            # پنل ادمین
│
├── components/
│   ├── ui/                 # کامپوننت‌های پایه (Button, Input, ...)
│   ├── layout/             # Header, Footer, Sidebar
│   ├── features/           # کامپوننت‌های Feature-specific
│   │   ├── products/       # ProductCard, ProductSwiper
│   │   ├── cart/           # CartPopup, CartItem
│   │   ├── auth/           # LoginForm, RegisterForm, CaptchaWidget
│   │   └── theme/          # ThemeBuilder, ThemePicker
│   └── shared/             # کامپوننت‌های مشترک
│
├── lib/
│   ├── api/                # API Client (Axios instances)
│   │   ├── axiosInstance.ts
│   │   └── endpoints/      # هر module endpoint جداگانه
│   ├── hooks/              # Custom Hooks
│   │   ├── useAuth.ts
│   │   ├── useTheme.ts
│   │   └── useCart.ts
│   ├── stores/             # Zustand Stores
│   │   ├── authStore.ts
│   │   └── cartStore.ts
│   └── utils/              # توابع کمکی
│       ├── date.ts         # تبدیل تاریخ شمسی
│       ├── currency.ts     # نمایش مبلغ فارسی
│       └── validators.ts   # Zod schemas
│
├── types/                  # TypeScript Types & Interfaces
│   ├── api.types.ts
│   ├── product.types.ts
│   └── user.types.ts
│
├── public/
│   ├── fonts/              # فونت‌های فارسی (Vazir, Shabnam, Samim, Tanha)
│   └── images/
│
├── styles/
│   ├── globals.css         # CSS Variables و تم‌ها
│   └── themes/             # فایل‌های تم
│
└── next.config.ts          # تنظیمات Next.js
```

---

## 10. قراردادهای کدنویسی

### 10.1 نام‌گذاری

```typescript
// کامپوننت‌ها: PascalCase
export const ProductCard = () => {};

// توابع و متغیرها: camelCase
const handleAddToCart = () => {};

// Constants: SCREAMING_SNAKE_CASE
const MAX_CART_ITEMS = 50;

// Type و Interface: PascalCase با پسوند
interface ProductCardProps { ... }
type CartItem = { ... };

// فایل‌ها: kebab-case برای صفحات، PascalCase برای کامپوننت‌ها
// components/ProductCard.tsx ✅
// app/products/page.tsx ✅
```

### 10.2 Server vs Client Components

```typescript
// ✅ Server Component پیش‌فرض است – فقط در صورت نیاز 'use client' بگذارید
// 'use client' فقط هنگام نیاز به: useState, useEffect, event handlers

// Server Component (پیش‌فرض):
export default async function ProductsPage() {
  const products = await fetchProducts(); // مستقیم fetch
  return <ProductList products={products} />;
}

// Client Component (فقط اگر state یا effect نیاز است):
'use client';
export const AddToCartButton = ({ productId }: { productId: number }) => {
  const [added, setAdded] = useState(false);
  // ...
};
```

### 10.3 Error Handling

```typescript
// هر صفحه باید error.tsx داشته باشد
// app/(shop)/products/error.tsx
'use client';
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="text-center py-12">
      <p>خطایی رخ داد: {error.message}</p>
      <Button onClick={reset}>تلاش مجدد</Button>
    </div>
  );
}

// Loading State
// app/(shop)/products/loading.tsx
export default function Loading() {
  return <ProductListSkeleton />;
}
```

### 10.4 API Calls

```typescript
// ✅ همه API calls از طریق lib/api استفاده می‌کنند
// ✅ هرگز مستقیم fetch در کامپوننت نزنید (جز Server Components)

// lib/api/endpoints/products.ts
export const productsApi = {
  getList: (params: ProductListParams) =>
    apiClient.get<PagedResponse<Product>>('/products', { params }),
    
  getBySlug: (slug: string) =>
    apiClient.get<Product>(`/products/${slug}`),
    
  create: (data: CreateProductDto) =>
    apiClient.post<Product>('/admin/products', data),
};
```

---

> **یادداشت برای توسعه‌دهنده:**
> - همیشه `'use client'` را به حداقل برسانید – Server Component به عملکرد بهتر می‌انجامد
> - هر `useEffect` با side effect باید Cleanup داشته باشد
> - فرم‌ها حتماً از Zod + React Hook Form استفاده کنند
> - تصاویر فقط از `next/image` – هرگز از `<img>` مستقیم
> - هر بخش (ادمین/فروشگاه/فروشنده) لایوت مستقل دارد
> - Responsive design از Mobile-First طراحی شود
