# راهنمای جامع استفاده از وردپرس به عنوان Headless CMS برای Next.js

این راهنما به زبانی ساده شما را در تمام مراحل راه‌اندازی وردپرس به عنوان یک منبع محتوا (CMS) و اتصال آن به یک وب‌سایت مدرن که با Next.js ساخته شده، همراهی می‌کند. در این معماری، وردپرس پنل مدیریت قدرتمند شما خواهد بود و Next.js وظیفه نمایش سایت به کاربران را با بالاترین سرعت بر عهده خواهد داشت.

## فهرست مطالب

- [چرا از این معماری استفاده کنیم؟](#چرا-از-این-معماری-استفاده-کنیم)
- [پیش‌نیازها](#پیش‌نیازها)
- [گام ۱: نصب و تنظیمات اولیه وردپرس](#گام-۱-نصب-و-تنظیمات-اولیه-وردپرس)
- [گام ۲: آشنایی با API وردپرس و راهنمای Endpoints](#گام-۲-آشنایی-با-api-وردپرس-و-راهنمای-endpoints)
- [گام ۳: ساختاردهی محتوای سفارشی](#گام-۳-ساختاردهی-محتوای-سفارشی)
- [گام ۴: افزودن فیلدهای دلخواه به API (بدون افزونه)](#گام-۴-افزودن-فیلدهای-دلخواه-به-api-بدون-افزونه)
- [گام ۵: ایمن‌سازی و احراز هویت](#گام-۵-ایمن‌سازی-و-احراز-هویت)
- [گام ۶: اتصال Next.js به وردپرس](#گام-۶-اتصال-nextjs-به-وردپرس)
- [گام ۷: مدیریت کاربران (ثبت‌نام و ورود)](#گام-۷-مدیریت-کاربران-ثبت‌نام-و-ورود)
- [گام ۸: بهینه‌سازی برای سرعت و موتورهای جستجو (SEO)](#گام-۸-بهینه‌سازی-برای-سرعت-و-موتورهای-جستجو-seo)
- [گام ۹: آماده‌سازی برای انتشار نهایی](#گام-۹-آماده‌سازی-برای-انتشار-نهایی)

---

## 🏛️ چرا از این معماری استفاده کنیم؟

وقتی وردپرس را از ظاهر سایت جدا می‌کنیم (Headless)، مزایای بزرگی به دست می‌آریم:

- **سرعت فوق‌العاده**: سایت شما با Next.js ساخته می‌شود که یکی از سریع‌ترین فریمورک‌های وب است و تجربه کاربری بهتری ارائه می‌دهد.
- **امنیت بیشتر**: پنل مدیریت وردپرس شما از دید عموم پنهان می‌ماند و این کار امنیت آن را به شدت افزایش می‌دهد.
- **تکنولوژی مدرن**: شما می‌توانید از جدیدترین ابزارهای دنیای برنامه‌نویسی برای ساخت ظاهر سایت خود استفاده کنید، بدون اینکه به قالب‌های وردپرس محدود باشید.
- **انعطاف‌پذیری بالا**: محتوای شما از طریق یک API در دسترس است و می‌توانید علاوه بر وب‌سایت، برای اپلیکیشن موبایل یا هر پلتفرم دیگری از آن استفاده کنید.

---

## ✅ پیش‌نیازها

- یک دامنه یا زیردامنه برای نصب وردپرس (مثلاً `api.yourdomain.com`).
- فعال بودن SSL (پروتکل https) روی دامنه.
- دسترسی به هاست (مانند cPanel).
- آشنایی بسیار اولیه با مفاهیم وب.

---

## 🚀 گام ۱: نصب و تنظیمات اولیه وردپرس

ابتدا وردپرس را به عنوان انبار محتوای خود آماده می‌کنیم.

1. **نصب وردپرس**:
   - یک دیتابیس در هاست خود بسازید.
   - آخرین نسخه وردپرس را از [سایت رسمی](https://wordpress.org) دانلود و نصب کنید.
2. **تنظیمات اولیه**:
   - **آدرس سایت**: در پیشخوان وردپرس به _تنظیمات > عمومی_ بروید و مطمئن شوید آدرس سایت با `https` شروع می‌شود.
   - **پیوندهای یکتا**: به _تنظیمات > پیوندهای یکتا_ بروید و گزینه "نام نوشته" را انتخاب کنید. این کار باعث می‌شود آدرس‌های API شما خوانا و استاندارد باشند.
3. **تست API**:
   - آدرس `https://api.yourdomain.com/wp-json` را در مرورگر خود باز کنید.
   - اگر یک صفحه با متن‌های زیاد و ساختاریافته (JSON) دیدید، یعنی API شما فعال و آماده است.

---

## 📡 گام ۲: آشنایی با API وردپرس و راهنمای Endpoints

API وردپرس دروازه ارتباطی بین سایت Next.js شما و محتوای وردپرس است. شما با ارسال درخواست به آدرس‌های مشخص (که به آنها Endpoint می‌گوییم)، اطلاعات را دریافت یا ارسال می‌کنید.

**ساختار اصلی آدرس‌ها**: `https://api.yourdomain.com/wp-json/wp/v2/`

### دریافت اطلاعات (درخواست GET)

- **دریافت آخرین پست‌ها**: `/wp/v2/posts`
- **دریافت صفحات (Pages)**: `/wp/v2/pages`
- **دریافت دسته‌بندی‌ها (Categories)**: `/wp/v2/categories`

### پارامترهای کاربردی برای فیلتر کردن نتایج

- محدود کردن تعداد نتایج: `/posts?per_page=5`
- جستجو: `/posts?search=کلمه کلیدی`
- دریافت اطلاعات مرتبط (مثل تصویر شاخص): `/posts?_embed` (بسیار مهم و کاربردی)

### ارسال و ویرایش اطلاعات (درخواست‌های POST, PUT, DELETE)

برای این عملیات باید احراز هویت شده باشید (در گام ۵ توضیح داده می‌شود).

- **ایجاد یک پست جدید (POST)**: به `/wp/v2/posts` با بدنه JSON.
- **حذف یک پست (DELETE)**: به `/wp/v2/posts/123`.

---

## 🏗️ گام ۳: ساختاردهی محتوای سفارشی

وردپرس به طور پیش‌فرض فقط "نوشته‌ها" و "برگه‌ها" را دارد. اگر بخواهید نوع محتوای دیگری مانند "پروژه‌ها"، "اخبار" یا "محصولات" داشته باشید، باید یک **Custom Post Type** بسازید.

1. افزونه **Custom Post Type UI (CPT UI)** را نصب و فعال کنید.
2. در پیشخوان به منوی _CPT UI > Add/Edit Post Types_ بروید.
3. یک نوع پست جدید با شناسه‌ای مانند `project` و نام "پروژه‌ها" بسازید.
4. در تنظیمات آن:
   - گزینه `Show in REST API` را روی `True` تنظیم کنید.
   - `REST API Base Slug` را روی `projects` تنظیم کنید.

✅ **تبریک!** شما یک Endpoint جدید در آدرس `/wp-json/wp/v2/projects` ساختید.

---

## ⚙️ گام ۴: افزودن فیلدهای دلخواه به API (بدون افزونه)

فرض کنید برای هر "پروژه" می‌خواهیم یک فیلد "لینک پروژه" داشته باشیم. برای این کار از قابلیت داخلی وردپرس به نام **Post Meta** استفاده می‌کنیم.

1. وارد هاست خود شوید و فایل `functions.php` قالب فعال وردپرس خود را پیدا کنید.
2. کد زیر را به انتهای این فایل اضافه کنید:

```php
function register_project_meta_fields() {
    register_post_meta('project', 'project_link', array(
        'show_in_rest' => true, // این خط باعث نمایش فیلد در API می‌شود
        'single' => true,
        'type' => 'string',
        'auth_callback' => function() { // اجازه آپدیت فیلد به کاربران با دسترسی ویرایش پست
            return current_user_can('edit_posts');
        }
    ));
}
add_action('rest_api_init', 'register_project_meta_fields');
```

**این کد چه کاری انجام می‌دهد؟**

- یک فیلد سفارشی به نام `project_link` برای نوع پست `project` تعریف می‌کند.
- با `show_in_rest => true`، به وردپرس می‌گوید که این فیلد را در خروجی API نمایش بدهد.
- حالا وقتی یک پروژه را از طریق API دریافت می‌کنید، یک بخش جدید به نام `meta` خواهید دید که شامل `project_link` است.

---

## 🛡️ گام ۵: ایمن‌سازی و احراز هویت

امنیت API شما بسیار مهم است. در این بخش، روش‌های احراز هویت و تنظیمات دسترسی را بررسی می‌کنیم.

### بخش ۵.۱: روش‌های احراز هویت

#### ۱. رمز عبور برنامه (Application Password)

این روش برای ارتباط سرور به سرور (یعنی ارتباط Next.js با وردپرس) ایده‌آل است.

1. در پیشخوان وردپرس، به _کاربران > پروفایل_ بروید.
2. در انتهای صفحه، در بخش "Application Passwords"، یک نام برای برنامه خود (مثلاً `NextJS_Server`) وارد کنید.
3. یک رمز عبور جدید به شما نمایش داده می‌شود. این رمز را کپی و در جای امنی ذخیره کنید.
4. این اطلاعات را در فایل `.env.local` پروژه Next.js خود قرار دهید:

```plaintext
WP_API_URL="https://api.yourdomain.com/wp-json"
WP_APPLICATION_USER="your_admin_username"
WP_APPLICATION_PASSWORD="your-generated-password"
```

#### ۲. توکن JWT (برای ورود کاربران سایت)

وقتی کاربری در سایت شما لاگین می‌کند، از این روش استفاده می‌کنیم.

1. افزونه **JWT Authentication for WP REST API** را نصب کنید.
2. یک کلید مخفی در فایل `wp-config.php` وردپرس خود تعریف کنید:

```php
define('JWT_AUTH_SECRET_KEY', 'your-super-secret-key-that-is-long-and-random');
define('JWT_AUTH_CORS_ENABLE', true);
```

### بخش ۵.۲: تنظیم CORS (بسیار مهم برای توسعه‌دهندگان فرانت‌اند)

**CORS چیست؟** به زبان ساده، مرورگرها یک سیاست امنیتی دارند که اجازه نمی‌دهد یک سایت (`your-frontend.com`) به راحتی از یک سایت دیگر (`api.yourdomain.com`) داده درخواست کند، مگر اینکه سایت دوم صراحتاً اجازه داده باشد.

**چرا به آن نیاز دارید؟** چون برنامه Next.js شما که روی `localhost:3000` (در حین توسعه) یا `your-frontend.com` (بعد از انتشار) اجرا می‌شود، یک "مبدأ" متفاوت از API وردپرس شماست. بدون تنظیم CORS، تمام درخواست‌های شما از فرانت‌اند با خطا مواجه خواهد شد.

#### برای سرور Apache (بسیار رایج)

کد زیر را به فایل `.htaccess` در پوشه ریشه وردپرس خود اضافه کنید:

```apache
<IfModule mod_headers.c>
    # به دامنه‌های مشخص اجازه دسترسی بده
    SetEnvIf Origin "https?://(www\.)?(your-frontend-domain\.com|localhost:3000)$" AccessControlAllowOrigin=$0
    Header add Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
    Header set Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization"
</IfModule>
```

**نکات مهم:**

- `your-frontend-domain\.com` را با دامنه اصلی سایت Next.js خود جایگزین کنید.
- `localhost:3000` برای این است که بتوانید در سیستم خودتان به راحتی توسعه دهید. قبل از انتشار نهایی این بخش را حذف کنید.

#### برای سرور Nginx

اگر از Nginx استفاده می‌کنید، بلوک زیر را به فایل کانفیگ سایت خود اضافه کنید:

```nginx
location / {
    if ($http_origin ~* (https?://(www\.)?your-frontend-domain\.com|https?://localhost:3000)) {
        add_header 'Access-Control-Allow-Origin' "$http_origin";
        add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
    }
}
```

---

## 🔗 گام ۶: اتصال Next.js به وردپرس

حالا در پروژه Next.js، یک فایل مرکزی برای مدیریت تمام درخواست‌ها به وردپرس می‌سازیم.

```javascript
// lib/api.js
const API_URL = process.env.WP_API_URL;

// یک تابع اصلی برای ارسال درخواست‌ها
async function fetchAPI(endpoint, options = {}) {
  const headers = { "Content-Type": "application/json" };

  const res = await fetch(`${API_URL}${endpoint}`, {
    method: options.method || "GET",
    headers,
    body: options.body ? JSON.stringify(options.body) : null,
    next: { revalidate: 60 }, // کش کردن نتایج برای ۶۰ ثانیه
  });

  if (!res.ok) {
    console.error("خطا در ارتباط با API وردپرس");
    return null;
  }

  return res.json();
}

// توابع کمکی برای سادگی کار
export const getPosts = () => fetchAPI("/wp/v2/posts?_embed");
export const getPostBySlug = (slug) =>
  fetchAPI(`/wp/v2/posts?slug=${slug}&_embed`);
```

### نمایش پست‌ها در یک صفحه:

```javascript
// app/blog/page.jsx
import Link from "next/link";
import { getPosts } from "@/lib/api";

export default async function BlogPage() {
  const posts = await getPosts();

  return (
    <div>
      <h1>وبلاگ</h1>
      {posts &&
        posts.map((post) => (
          <article key={post.id}>
            <h2>
              <Link href={`/blog/${post.slug}`}>{post.title.rendered}</Link>
            </h2>
            <div dangerouslySetInnerHTML={{ __html: post.excerpt.rendered }} />
          </article>
        ))}
    </div>
  );
}
```

---

## 👤 گام ۷: مدیریت کاربران (ثبت‌نام و ورود)

با استفاده از **Route Handler**ها در Next.js، می‌توانید فرم‌های ثبت‌نام و ورود را مدیریت کنید.

### مثال: API Route برای ورود کاربر

```javascript
// app/api/auth/signin/route.js
import { NextResponse } from "next/server";
import { cookies } from "next/headers";

export async function POST(req) {
  const { username, password } = await req.json();

  const res = await fetch(`${process.env.WP_API_URL}/jwt-auth/v1/token`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ username, password }),
  });

  const data = await res.json();

  if (!res.ok) {
    return NextResponse.json(
      { error: "نام کاربری یا رمز عبور اشتباه است." },
      { status: 401 }
    );
  }

  // ذخیره توکن در کوکی امن HttpOnly
  cookies().set("token", data.token, {
    httpOnly: true,
    secure: process.env.NODE_ENV === "production",
    sameSite: "strict",
    path: "/",
  });

  return NextResponse.json({ message: "ورود موفقیت‌آمیز بود." });
}
```

---

## ⚡️ گام ۸: بهینه‌سازی برای سرعت و موتورهای جستجو (SEO)

- **کشینگ (Caching)**: Next.js به طور خودکار نتایج درخواست‌ها را کش می‌کند. این یعنی سایت شما برای بازدیدهای مکرر به سرعت بارگذاری می‌شود.
- **بهینه‌سازی تصاویر**: از کامپوننت `<Image>` خود Next.js به جای تگ `<img>` معمولی استفاده کنید. این کامپوننت تصاویر را بهینه، فشرده و با بارگذاری تنبل (Lazy-loading) ارائه می‌دهد.
- **SEO**:
  - افزونه **Yoast SEO** را در وردپرس نصب کنید.
  - عنوان و توضیحات متا را برای هر صفحه مدیریت کنید و از طریق API دریافت کرده و در تگ `<head>` سایت Next.js قرار دهید.

---

## 📦 گام ۹: آماده‌سازی برای انتشار نهایی

قبل از انتشار سایت، این موارد را بررسی کنید:

- **غیرفعال کردن حالت دیباگ**: در فایل `wp-config.php`، مقدار `WP_DEBUG` را روی `false` تنظیم کنید.
- **متغیرهای محیطی**: اطمینان حاصل کنید که تمام اطلاعات حساس (مانند رمز عبور برنامه) در متغیرهای محیطی سرور شما (مثلاً در Vercel یا Netlify) تنظیم شده‌اند.
- **پشتیبان‌گیری (Backup)**: همیشه یک نسخه پشتیبان از دیتابیس و فایل‌های وردپرس خود داشته باشید.
- **حذف localhost از تنظیمات CORS**: دسترسی از `localhost` را در فایل `.htaccess` یا کانفیگ Nginx غیرفعال کنید.
