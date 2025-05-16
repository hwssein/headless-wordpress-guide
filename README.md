# راهنمای جامع استفاده از وردپرس به عنوان Headless CMS

این راهنما به شما کمک می‌کنه تا وردپرس رو به‌عنوان یه Headless CMS تنظیم کنید و از REST API اون برای اتصال به برنامه‌های فرانت‌اند (مثل Next.js) استفاده کنید. هدف، ساخت یه بک‌اند امن، سریع و مقیاس‌پذیر برای مدیریت محتواست.

---

## فهرست مطالب

- [پیش‌نیازها](#پیشنیازها)
- [قدم ۱: نصب وردپرس و افزونه‌های ضروری](#قدم-۱-نصب-وردپرس-و-افزونههای-ضروری)
- [قدم ۲: تنظیم احراز هویت و امنیت API](#قدم-۲-تنظیم-احراز-هویت-و-امنیت-api)
- [قدم ۳: تنظیم CORS](#قدم-۳-تنظیم-cors)
- [قدم ۴: کار با داده‌های API](#قدم-۴-کار-با-دادههای-api)
- [قدم ۵: گسترش API](#قدم-۵-گسترش-api)
- [قدم ۶: اتصال به Next.js](#قدم-۶-اتصال-به-nextjs)
- [قدم ۷: بهینه‌سازی و امنیت](#قدم-۷-بهینهسازی-و-امنیت)
- [قدم ۸: ملاحظات SEO](#قدم-۸-ملاحظات-seo)
- [قدم ۹: آماده‌سازی برای تولید](#قدم-۹-آمادهسازی-برای-تولید)
- [قدم ۱۰: نگهداری و به‌روزرسانی](#قدم-۱۰-نگهداری-و-بهروزرسانی)
- [نکات تکمیلی](#نکات-تکمیلی)

---

## پیش‌نیازها

- نصب وردپرس روی دامنه یا زیردامنه (مثل `api.example.com`).
- دسترسی به پنل هاست (مثل cPanel) برای مدیریت فایل‌ها و دیتابیس.
- پروژه فرانت‌اند آماده (مثل Next.js یا React).
- گواهینامه SSL فعال (HTTPS) برای امنیت.

---

## قدم ۱: نصب وردپرس و افزونه‌های ضروری

### 1.1 نصب وردپرس
- وردپرس رو از طریق cPanel یا FTP نصب کنید.
- بعد از نصب، REST API رو تست کنید:
  ```
  GET https://api.example.com/wp-json/wp/v2/posts
  ```
  پاسخ باید JSON باشه یا خطای احراز هویت بده.

### 1.2 افزونه‌های پیشنهادی
| افزونه                | کاربرد                            | تنظیمات پیشنهادی                                      |
|-----------------------|----------------------------------|-------------------------------------------------------|
| **W3 Total Cache**    | کش و بهینه‌سازی سرعت            | فعال کردن Page Cache، مستثنی کردن `/wp-json/*`       |
| **WP-Optimize**       | بهینه‌سازی دیتابیس             | بهینه‌سازی خودکار هفتگی                            |
| **Wordfence Security**| امنیت و محدود کردن درخواست‌ها    | فعال کردن Rate Limiting برای `/wp-json/`             |
| **Imagify**           | فشرده‌سازی تصاویر              | فشرده‌سازی خودکار تصاویر                           |
| **Application Passwords** | احراز هویت API             | برای تولید رمزهای API                               |
| **ACF to REST API**   | نمایش فیلدهای سفارشی در API    | برای استفاده از ACF در API                          |

**تنظیمات مهم**:
- **W3 Total Cache**: مسیر `/wp-json/*` رو از کش مستثنی کنید.
- **Wordfence**: محدودیت درخواست (مثلاً 10 درخواست در دقیقه برای هر IP).

---

## قدم ۲: تنظیم احراز هویت و امنیت API

### 2.1 تولید Application Password
- توی پیشخوان وردپرس، برید به **کاربران > پروفایل**.
- توی بخش **Application Passwords**، یه اسم (مثل `Frontend-App`) وارد کنید و رمز تولید کنید.
- رمز رو توی فایل `.env` پروژه فرانت‌اند ذخیره کنید:
  ```env
  WP_API_USERNAME=your_username
  WP_API_KEY=abcd-1234-efgh-5678
  WP_API_URL=https://api.example.com/wp-json
  ```

### 2.2 اجباری کردن احراز هویت
این کد رو به `functions.php` اضافه کنید:
```php
add_filter('rest_authentication_errors', function ($result) {
    if (!is_user_logged_in() && !current_user_can('read')) {
        return new WP_Error('rest_forbidden', 'Authentication required.', ['status' => 401]);
    }
    return $result;
});
```

### 2.3 استفاده از رمز در درخواست‌ها
هدر درخواست رو این‌جوری تنظیم کنید:
```http
Authorization: Basic base64(username:appPassword)
```

---

## قدم ۳: تنظیم CORS

برای محدود کردن دسترسی به API، توی `.htaccess` این کد رو اضافه کنید:
```apache
<IfModule mod_headers.c>
    SetEnvIf Origin "http(s)?://(www\.)?(example.com|localhost(:[0-9]+)?)$" AccessControlAllowOrigin=$0
    Header add Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
    Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization"
</IfModule>
```
- `example.com` رو با دامنه فرانت‌اند عوض کنید.
- توی تولید، `localhost` رو حذف کنید.

---

## قدم ۴: کار با داده‌های API

### 4.1 دریافت داده‌ها
- پست‌ها:
  ```http
  GET https://api.example.com/wp-json/wp/v2/posts?per_page=5
  ```
- دسته‌بندی‌ها:
  ```http
  GET https://api.example.com/wp-json/wp/v2/categories
  ```

### 4.2 پاکسازی HTML
توی فرانت‌اند از `striptags` استفاده کنید:
```javascript
import striptags from 'striptags';
const cleanContent = striptags(post.content.rendered);
```

### 4.3 ساخت پست
```javascript
await fetch('https://api.example.com/wp-json/wp/v2/posts', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Authorization: 'Basic ' + btoa('username:appPassword'),
  },
  body: JSON.stringify({
    title: 'پست جدید',
    content: 'محتوا',
    status: 'publish',
    categories: [1],
    acf: { custom_field: 'مقدار' }
  })
});
```

### 4.4 بررسی وجود کاربر
یه endpoint سفارشی بسازید:
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
const res = await fetch('https://api.example.com/wp-json/custom/v1/user-exists', {
  method: 'POST',
  body: JSON.stringify({ username: 'test', email: 'test@example.com' }),
  headers: { 'Content-Type': 'application/json' }
});
```

---

## قدم ۵: گسترش API

### 5.1 استفاده از ACF
- افزونه **ACF** و **ACF to REST API** رو نصب کنید.
- فیلدهای سفارشی رو به API اضافه کنید.

### 5.2 Custom Post Type
```php
add_action('init', function () {
    register_post_type('news', [
        'public' => true,
        'show_in_rest' => true,
        'supports' => ['title', 'editor', 'thumbnail']
    ]);
});
```

---

## قدم ۶: اتصال به Next.js

```javascript
// lib/getPosts.js
export async function getPosts() {
  const res = await fetch(`${process.env.WP_API_URL}/wp/v2/posts`, {
    headers: { Authorization: `Basic ${Buffer.from(`${process.env.WP_API_USERNAME}:${process.env.WP_API_KEY}`).toString('base64')}` }
  });
  const posts = await res.json();
  return posts.map(post => ({
    id: post.id,
    title: post.title.rendered,
    content: striptags(post.content.rendered)
  }));
}

// page.js
import { getPosts } from './lib/getPosts';
export default async function Home() {
  const posts = await getPosts();
  return (
    <div>
      {posts.map(post => <div key={post.id}>{post.title}</div>)}
    </div>
  );
}
```

---

## قدم ۷: بهینه‌سازی و امنیت

- **بهینه‌سازی**: دیتابیس رو با WP-Optimize بهینه کنید.
- **امنیت**: دسترسی به `/wp-admin` رو محدود کنید:
  ```apache
  <Directory "/path/to/wp-admin">
      Require ip 192.168.1.1
  </Directory>
  ```

---

## قدم ۸: ملاحظات SEO

- از **Yoast SEO** استفاده کنید و متادیتا رو از API بگیرید:
  ```http
  GET https://api.example.com/wp-json/yoast/v1/get_head?url=/post-slug
  ```

---

## قدم ۹: آماده‌سازی برای تولید

- **تست**: مطمئن شید درخواست بدون احراز هویت خطای 401 می‌ده.
- **تنظیمات**: `WP_DEBUG` رو غیرفعال کنید:
  ```php
  define('WP_DEBUG', false);
  ```

---

## قدم ۱۰: نگهداری و به‌روزرسانی

- وردپرس و افزونه‌ها رو به‌روز نگه دارید.
- بک‌آپ‌ها رو با **UpdraftPlus** تست کنید.

---

## نکات تکمیلی

- از `SWR` برای کش کلاینت استفاده کنید.
- مسیرهای حساس (مثل `/wp/v2/users`) رو غیرفعال کنید:
  ```php
  add_filter('rest_endpoints', function ($endpoints) {
      unset($endpoints['/wp/v2/users']);
      return $endpoints;
  });
  ```