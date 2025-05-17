# راهنمای جامع استفاده از وردپرس به‌عنوان Headless CMS

این راهنما به‌صورت گام‌به‌گام شما را در تنظیم وردپرس به‌عنوان یک Headless CMS با استفاده از REST API راهنمایی می‌کند تا بتوانید بک‌اندی امن، سریع و مقیاس‌پذیر برای برنامه‌های فرانت‌اند (مانند Next.js) ایجاد کنید.

---

## فهرست مطالب

- [پیش‌نیازها](#پیشنیازها)
- [گام ۱: نصب و تنظیم وردپرس در cPanel](#گام-۱-نصب-و-تنظیم-وردپرس-در-cpanel)
- [گام ۲: نصب افزونه‌های ضروری](#گام-۲-نصب-افزونههای-ضروری)
- [گام ۳: تنظیم احراز هویت و امنیت API](#گام-۳-تنظیم-احراز-هویت-و-امنیت-api)
- [گام ۴: تنظیم CORS](#گام-۴-تنظیم-cors)
- [گام ۵: کار با داده‌های API](#گام-۵-کار-با-دادههای-api)
- [گام ۶: گسترش API با ACF و Custom Post Types](#گام-۶-گسترش-api-با-acf-و-custom-post-types)
- [گام ۷: اتصال به Next.js](#گام-۷-اتصال-به-nextjs)
- [گام ۸: بهینه‌سازی و امنیت](#گام-۸-بهینهسازی-و-امنیت)
- [گام ۹: ملاحظات SEO](#گام-۹-ملاحظات-seo)
- [گام ۱۰: آماده‌سازی برای تولید](#گام-۱۰-آمادهسازی-برای-تولید)
- [گام ۱۱: نگهداری و به‌روزرسانی](#گام-۱۱-نگهداری-و-بهروزرسانی)
- [نکات تکمیلی](#نکات-تکمیلی)

---

## پیش‌نیازها

- دامنه یا زیردامنه (مانند `api.example.com`) با گواهینامه SSL فعال (HTTPS).
- دسترسی به پنل هاست (مانند cPanel) برای مدیریت فایل‌ها و دیتابیس.
- پروژه فرانت‌اند آماده (مانند Next.js یا React).
- آشنایی اولیه با مدیریت هاست، وردپرس و API.

---

## گام ۱: نصب و تنظیم وردپرس در cPanel

### 1.1 ساخت دیتابیس
1. در cPanel، به بخش **MySQL Databases** بروید.
2. یک دیتابیس جدید ایجاد کنید (مثلاً `wp_database`).
3. یک کاربر دیتابیس (مثلاً `wp_user`) با رمز قوی بسازید.
4. کاربر را به دیتابیس اضافه کرده و تمام دسترسی‌ها (**All Privileges**) را به او اختصاص دهید.
5. اطلاعات دیتابیس (نام دیتابیس، نام کاربر، رمز) را ذخیره کنید.

### 1.2 دانلود و آپلود وردپرس
1. آخرین نسخه وردپرس را از [wordpress.org](https://wordpress.org) دانلود کنید.
2. در cPanel، به **File Manager** بروید و فایل زیپ وردپرس را در پوشه اصلی دامنه (مانند `public_html` یا زیردامنه) آپلود کنید.
3. فایل را استخراج کرده و محتویات پوشه `wordpress` را به ریشه دامنه منتقل کنید.

### 1.3 نصب وردپرس
1. دامنه یا زیردامنه (مانند `api.example.com`) را در مرورگر باز کنید.
2. مراحل نصب وردپرس را دنبال کنید:
   - زبان موردنظر (مانند فارسی) را انتخاب کنید.
   - اطلاعات دیتابیس (نام دیتابیس، نام کاربر، رمز، هاست: معمولاً `localhost`) را وارد کنید.
   - نام سایت، نام کاربری ادمین و رمز قوی تنظیم کنید.
3. پس از نصب، به پیشخوان وردپرس (مانند `api.example.com/wp-admin`) وارد شوید.

### 1.4 تنظیمات اولیه
- در پیشخوان، به **تنظیمات > عمومی** بروید و اطمینان حاصل کنید که آدرس سایت روی HTTPS تنظیم شده است.
- در **تنظیمات > پیوندهای یکتا**، ساختار پیوندها را به «نام نوشته» تغییر دهید.
- گواهینامه SSL را در cPanel (بخش **Security > SSL/TLS**) بررسی کنید و در صورت نیاز نصب کنید.

### 1.5 تست REST API
برای اطمینان از عملکرد API:
```
GET https://api.example.com/wp-json/wp/v2/posts
```
پاسخ باید یک JSON (لیست پست‌ها یا خطای احراز هویت) باشد.

---

## گام ۲: نصب افزونه‌های ضروری

| افزونه                     | کاربرد                              | تنظیمات پیشنهادی                                   |
|----------------------------|------------------------------------|---------------------------------------------------|
| **W3 Total Cache**         | کش و بهینه‌سازی سرعت              | فعال‌سازی Page Cache، مستثنی کردن `/wp-json/*`   |
| **WP-Optimize**            | بهینه‌سازی دیتابیس               | بهینه‌سازی خودکار هفتگی                         |
| **Wordfence Security**     | امنیت و محدود کردن درخواست‌ها     | فعال‌سازی Rate Limiting برای `/wp-json/`         |
| **Imagify**                | فشرده‌سازی تصاویر                | فشرده‌سازی خودکار تصاویر                        |
| **Application Passwords**  | احراز هویت API                   | برای تولید رمزهای API                           |
| **ACF to REST API**        | نمایش فیلدهای سفارشی در API      | برای استفاده از ACF در API                       |
| **JWT Authentication for WP REST API** | احراز هویت با JWT | تنظیم کلید مخفی و فعال‌سازی CORS              |

**تنظیمات مهم**:
- **W3 Total Cache**: مسیر `/wp-json/*` را از کش مستثنی کنید.
- **Wordfence**: محدودیت درخواست (مثلاً 10 درخواست در دقیقه برای هر IP) تنظیم کنید.

---

## گام ۳: تنظیم احراز هویت و امنیت API

### 3.1 احراز هویت با Application Password
- در پیشخوان وردپرس، به **کاربران > پروفایل** بروید.
- در بخش **Application Passwords**، یک نام (مانند `Frontend-App`) وارد کرده و رمز تولید کنید (مثل `abcd-1234-efgh-5678`).
- رمز را در فایل `.env` پروژه فرانت‌اند ذخیره کنید:
  ```env
  WP_API_USERNAME=your_username
  WP_API_KEY=abcd-1234-efgh-5678
  WP_API_URL=https://api.example.com/wp-json
  ```
- در درخواست‌ها، از هدر زیر استفاده کنید:
  ```http
  Authorization: Basic base64(username:appPassword)
  ```

### 3.2 احراز هویت با JWT
برای احراز هویت کاربران نهایی (مانند فرم لاگین):
1. افزونه **JWT Authentication for WP REST API** را از [مخزن وردپرس](https://wordpress.org/plugins/jwt-authentication-for-wp-rest-api/) نصب و فعال کنید.
2. در فایل `wp-config.php` (قبل از خط `/* That's all, stop editing! */`) کلید مخفی اضافه کنید:
   ```php
   define('JWT_AUTH_SECRET_KEY', 'your-very-secure-random-key-64-chars');
   define('JWT_AUTH_CORS_ENABLE', true);
   ```
   کلید باید رشته‌ای طولانی و تصادفی باشد (از ابزارهایی مثل [random.org](https://www.random.org) استفاده کنید).
3. برای سرور Apache، در فایل `.htaccess` این کد را اضافه کنید:
   ```apache
   RewriteEngine On
   RewriteCond %{HTTP:Authorization} ^(.*)
   RewriteRule ^(.*) - [E=HTTP_AUTHORIZATION:%1]
   ```
4. برای دریافت توکن، درخواست زیر را ارسال کنید:
   ```http
   POST https://api.example.com/wp-json/jwt-auth/v1/token
   Content-Type: application/json
   ```
   بدنه درخواست:
   ```json
   {
     "username": "user@example.com",
     "password": "userpassword"
   }
   ```
   پاسخ موفق:
   ```json
   {
     "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
     "user_email": "user@example.com",
     "user_nicename": "username",
     "user_display_name": "User Name"
   }
   ```
5. توکن را در کوکی یا localStorage ذخیره کنید (ترجیحاً کوکی HTTP-only برای امنیت بیشتر):
   ```javascript
   cookies().set('token', token, {
     httpOnly: true,
     secure: true,
     sameSite: 'strict',
     maxAge: 3 * 24 * 60 * 60
   });
   ```
6. در درخواست‌های بعدی، توکن را در هدر ارسال کنید:
   ```http
   Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
   ```

### 3.3 مقایسه JWT و Application Password
| ویژگی                  | JWT                              | Application Password             |
|-----------------------|----------------------------------|----------------------------------|
| **کاربرد**           | لاگین کاربران نهایی            | دسترسی سرور به سرور یا ابزارها  |
| **مسیر**             | `/jwt-auth/v1/token`            | مستقیم در تمام APIها            |
| **ذخیره‌سازی**       | کوکی یا localStorage           | فایل `.env`                     |
| **هدر**              | `Bearer <token>`               | `Basic base64(user:appPassword)`|
| **انقضا**            | معمولاً موقت (مثلاً 72 ساعت)   | دائمی تا لغو                   |

**نکته**: می‌توانید از هر دو روش به‌صورت هم‌زمان استفاده کنید بدون تداخل.

### 3.4 اجباری کردن احراز هویت برای مسیرهای خاص
برای محدود کردن دسترسی به مسیر `/wp/v2/users`:
```php
add_filter('rest_authentication_errors', function ($result) {
    if (!empty($result)) {
        return $result;
    }
    $request_uri = $_SERVER['REQUEST_URI'];
    if (strpos($request_uri, '/wp-json/wp/v2/users') !== false) {
        if (!is_user_logged_in() || !current_user_can('list_users')) {
            return new WP_Error(
                'rest_forbidden',
                'Authentication required to access users.',
                ['status' => 401]
            );
        }
    }
    return $result;
});
```

---

## گام ۴: تنظیم CORS

برای محدود کردن دسترسی API به دامنه‌های مجاز، در فایل `.htaccess` این کد را اضافه کنید:
```apache
<IfModule mod_headers.c>
    SetEnvIf Origin "http(s)?://(www\.)?(example.com|localhost(:[0-9]+)?)$" AccessControlAllowOrigin=$0
    Header add Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
    Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization"
</IfModule>
```
- `example.com` را با دامنه فرانت‌اند جایگزین کنید.
- در محیط تولید، `localhost(:[0-9]+)?` را حذف کنید.

---

## گام ۵: کار با داده‌های API

### 5.1 بررسی اتصال به API
برای بررسی دسترسی به API:
```javascript
const isWpAvailable = async () => {
  try {
    const res = await fetch('https://api.example.com/wp-json');
    return res.ok;
  } catch (err) {
    return false;
  }
};
```

### 5.2 دریافت داده‌ها
- **پست‌ها**:
  ```http
  GET https://api.example.com/wp-json/wp/v2/posts?per_page=5&orderby=date
  ```
- **دسته‌بندی‌ها**:
  ```http
  GET https://api.example.com/wp-json/wp/v2/categories
  ```
- **پست‌ها با دسته خاص**:
  ```http
  GET https://api.example.com/wp-json/wp/v2/posts?categories=1
  ```

### 5.3 پاکسازی HTML
برای حذف تگ‌های HTML از محتوا:
```javascript
import striptags from 'striptags';
const cleanContent = striptags(post.content.rendered);
```

### 5.4 ساخت پست
نمونه درخواست برای ایجاد پست:
```javascript
const createPost = async () => {
  try {
    const res = await fetch('https://api.example.com/wp-json/wp/v2/posts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...'
      },
      body: JSON.stringify({
        title: 'پست جدید',
        content: 'محتوای پست',
        status: 'publish',
        categories: [1],
        acf: { custom_field: 'مقدار سفارشی' }
      })
    });
    if (!res.ok) throw new Error('Failed to create post');
    return await res.json();
  } catch (error) {
    console.error('Error:', error);
    return null;
  }
};
```

### 5.5 حذف پست
```http
DELETE https://api.example.com/wp-json/wp/v2/posts/123?force=true
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...
```

### 5.6 بررسی وجود کاربر
برای بررسی وجود کاربر با ایمیل یا نام کاربری:
```php
add_action('rest_api_init', function () {
    register_rest_route('custom/v1', '/user-exists', [
        'methods' => 'POST',
        'callback' => function ($request) {
            $username = sanitize_text_field($request->get_param('username'));
            $email = sanitize_email($request->get_param('email'));
            return ['exists' => username_exists($username) || email_exists($email)];
        },
        'permission_callback' => '__return_true'
    ]);
});
```
درخواست:
```javascript
const checkUser = async () => {
  const res = await fetch('https://api.example.com/wp-json/custom/v1/user-exists', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ username: 'test', email: 'test@example.com' })
  });
  const data = await res.json();
  return data.exists;
};
```

### 5.7 مدیریت خطاها
| کد خطا | توضیح                         | راه‌حل                                              |
|--------|-----------------------------|---------------------------------------------------|
| 401    | احراز هویت ناموفق          | بررسی نام کاربری/رمز یا توکن JWT                  |
| 403    | دسترسی غیرمجاز             | بررسی سطح دسترسی کاربر (مثل `list_users`)         |
| 429    | تعداد درخواست زیاد         | تنظیم Rate Limiting در Wordfence                  |
| 500    | خطای سرور                  | بررسی لاگ‌ها در `wp-content/debug.log`            |

---

## گام ۶: گسترش API با ACF و Custom Post Types

### 6.1 استفاده از ACF
- افزونه‌های **Advanced Custom Fields (ACF)** و **ACF to REST API** را نصب کنید.
- فیلدهای سفارشی را به پست‌ها یا صفحات اضافه کنید.

#### مثال ۱: فیلد متنی (Summary)
1. در پیشخوان، به **Custom Fields > Add New** بروید.
2. گروه فیلد بسازید (مثل `Post Fields`) و فیلد متنی با نام `summary` اضافه کنید.
3. مکان را روی «پست‌ها» تنظیم کنید.
4. پاسخ API:
   ```json
   {
     "id": 123,
     "title": { "rendered": "عنوان پست" },
     "acf": { "summary": "خلاصه پست" }
   }
   ```

#### مثال ۲: فیلد تصویر
1. فیلد تصویر با نام `main_image` بسازید.
2. پاسخ API:
   ```json
   {
     "acf": { "main_image": { "url": "https://api.example.com/wp-content/uploads/image.jpg" } }
   }
   ```
3. در Next.js:
   ```javascript
   <img src={post.acf.main_image.url} alt="Main Image" />
   ```

#### مثال ۳: فیلد URL
1. فیلد URL با نام `video_link` بسازید.
2. پاسخ API:
   ```json
   {
     "acf": { "video_link": "https://youtube.com/video" }
   }
   ```
3. در فرانت‌اند:
   ```javascript
   <a href={post.acf.video_link}>مشاهده ویدیو</a>
   ```

### 6.2 Custom Post Type
برای نوع محتوای سفارشی (مثل اخبار):
```php
add_action('init', function () {
    register_post_type('news', [
        'public' => true,
        'show_in_rest' => true,
        'rest_base' => 'news',
        'labels' => ['name' => 'اخبار'],
        'supports' => ['title', 'editor', 'thumbnail']
    ]);
});
```
دریافت از API:
```http
GET https://api.example.com/wp-json/wp/v2/news
```

---

## گام ۷: اتصال به Next.js

### 7.1 دریافت و نمایش پست‌ها
```javascript
// lib/getPosts.js
import striptags from 'striptags';

export async function getPosts() {
  try {
    const res = await fetch(`${process.env.WP_API_URL}/wp/v2/posts?per_page=10`, {
      headers: {
        Authorization: `Bearer ${process.env.JWT_TOKEN}`
      },
      cache: 'force-cache'
    });
    if (!res.ok) throw new Error('Failed to fetch posts');
    const posts = await res.json();
    return posts.map(post => ({
      id: post.id,
      title: post.title.rendered,
      content: striptags(post.content.rendered),
      summary: post.acf?.summary || ''
    }));
  } catch (error) {
    console.error('Error fetching posts:', error);
    return [];
  }
}

// pages/index.js
import { getPosts } from '../lib/getPosts';

export default async function Home() {
  const posts = await getPosts();
  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-4">پست‌ها</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id} className="mb-4">
            <h2 className="text-xl">{post.title}</h2>
            <p>{post.summary || post.content.slice(0, 100)}...</p>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 7.2 ثبت‌نام کاربر
```javascript
// pages/api/signup.js
import { NextResponse } from 'next/server';

export async function POST(req) {
  try {
    const { email, password } = await req.json();
    const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    if (!emailRegex.test(email) || !password || password.length < 4) {
      return NextResponse.json(
        { error: 'اطلاعات وارد شده معتبر نیست.' },
        { status: 422 }
      );
    }

    const checkUser = await fetch('https://api.example.com/wp-json/custom/v1/user-exists', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email })
    });
    const { exists } = await checkUser.json();
    if (exists) {
      return NextResponse.json(
        { error: 'کاربری با این ایمیل وجود دارد.' },
        { status: 409 }
      );
    }

    const userData = {
      username: email.split('@')[0],
      email,
      password,
      role: 'subscriber'
    };
    const res = await fetch('https://api.example.com/wp-json/wp/v2/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Basic ${Buffer.from(`${process.env.WP_API_USERNAME}:${process.env.WP_API_KEY}`).toString('base64')}`
      },
      body: JSON.stringify(userData)
    });

    if (res.status !== 201) {
      return NextResponse.json(
        { error: 'مشکلی در ثبت‌نام پیش آمد.' },
        { status: 400 }
      );
    }

    return NextResponse.json(
      { message: 'با موفقیت ثبت‌نام شد.' },
      { status: 201 }
    );
  } catch (error) {
    console.error(error);
    return NextResponse.json(
      { error: 'خطای سرور رخ داد.' },
      { status: 500 }
    );
  }
}
```

### 7.3 ورود کاربر با JWT
```javascript
// pages/api/signin.js
import { NextResponse, cookies } from 'next/server';

export async function POST(req) {
  try {
    const { email, password } = await req.json();
    const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
    if (!emailRegex.test(email) || !password || password.length < 4) {
      return NextResponse.json(
        { error: 'اطلاعات وارد شده معتبر نیست.' },
        { status: 422 }
      );
    }

    const res = await fetch('https://api.example.com/wp-json/jwt-auth/v1/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ username: email, password })
    });
    const data = await res.json();
    if (!data.token) {
      return NextResponse.json(
        { error: 'اطلاعات وارد شده معتبر نیست.' },
        { status: 422 }
      );
    }

    cookies().set({
      name: 'token',
      value: data.token,
      path: '/',
      secure: true,
      httpOnly: true,
      sameSite: 'strict',
      maxAge: 60 * 60 * 72
    });

    return NextResponse.json({ message: 'ورود موفق' }, { status: 200 });
  } catch (error) {
    console.error(error);
    return NextResponse.json(
      { error: 'خطای سرور رخ داد.' },
      { status: 500 }
    );
  }
}
```

### 7.4 اعتبارسنجی توکن در Next.js
برای محافظت از مسیرهای API:
```javascript
// middleware.js
import { NextResponse } from 'next/server';
import { verify } from 'jsonwebtoken';

export async function middleware(req) {
  const token = req.cookies.get('token')?.value || req.headers.get('Authorization')?.split(' ')[1];
  if (!token) {
    return NextResponse.json({ error: 'احراز هویت لازم است.' }, { status: 401 });
  }

  try {
    const secretKey = process.env.JWT_SECRET_KEY;
    verify(token, secretKey);
    return NextResponse.next();
  } catch (err) {
    return NextResponse.json({ error: 'توکن نامعتبر است.' }, { status: 401 });
  }
}
```

---

## گام ۸: بهینه‌سازی و امنیت

- **بهینه‌سازی**:
  - دیتابیس را با **WP-Optimize** به‌صورت هفتگی بهینه کنید.
  - تصاویر را با **Imagify** فشرده کنید.
- **امنیت**:
  - دسترسی به `/wp-admin` را محدود کنید:
    ```apache
    <Directory "/path/to/wp-admin">
        Require ip 192.168.1.1
    </Directory>
    ```
  - رمزهای Application Password و کلید JWT را هر ۳-۶ ماه تغییر دهید.

---

## گام ۹: ملاحظات SEO

- از افزونه **Yoast SEO** برای تولید متادیتا استفاده کنید.
- متادیتا را از API دریافت کنید:
  ```http
  GET https://api.example.com/wp-json/yoast/v1/get_head?url=/post-slug
  ```
- در Next.js:
  ```javascript
  import Head from 'next/head';

  export default function Post({ post, seo }) {
    return (
      <>
        <Head>
          <title>{seo.title}</title>
          <meta name="description" content={seo.description} />
          <meta property="og:title" content={seo.og_title} />
        </Head>
        <h1>{post.title.rendered}</h1>
      </>
    );
  }
  ```

---

## گام ۱۰: آماده‌سازی برای تولید

- **تست‌ها**:
  - درخواست بدون احراز هویت → خطای 401.
  - درخواست از دامنه غیرمجاز → خطای CORS.
  - دسترسی به پست‌های منتشرنشده → خطای 403.
- **تنظیمات**:
  - در `wp-config.php`، دیباگ را غیرفعال کنید:
    ```php
    define('WP_DEBUG', false);
    ```

---

## گام ۱۱: نگهداری و به‌روزرسانی

- وردپرس، افزونه‌ها و قالب‌ها را به‌روز نگه دارید.
- با **UpdraftPlus** بک‌آپ‌های دوره‌ای بگیرید و بازیابی را تست کنید.
- لاگ‌های Wordfence را برای شناسایی حملات بررسی کنید.

---

## نکات تکمیلی

- از **SWR** یا **React Query** برای کش سمت کلاینت استفاده کنید.
- برای APIهای پیچیده، مسیرهای سفارشی بسازید:
  ```php
  add_action('rest_api_init', function () {
      register_rest_route('custom/v1', '/stats', [
          'methods' => 'GET',
          'callback' => function () {
              return ['visits' => 1000];
          },
          'permission_callback' => '__return_true'
      ]);
  });
  ```
- برای تصاویر بهینه، از `srcset` در فرانت‌اند استفاده کنید.