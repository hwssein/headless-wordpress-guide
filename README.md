# راهنمای جامع استفاده از وردپرس به عنوان Headless CMS

این راهنما به شما کمک می‌کنه تا وردپرس رو به‌عنوان یه Headless CMS تنظیم کنید و از REST API اون برای اتصال به برنامه‌های فرانت‌اند (مثل Next.js) استفاده کنید. هدف، ساخت یه بک‌اند امن، سریع و مقیاس‌پذیر برای مدیریت محتواست.

---

## فهرست مطالب

- [پیش‌نیازها](#پیشنیازها)
- [قدم ۱: نصب و تنظیم وردپرس در cPanel](#قدم-۱-نصب-و-تنظیم-وردپرس-در-cpanel)
- [قدم ۲: نصب افزونه‌های ضروری](#قدم-۲-نصب-افزونههای-ضروری)
- [قدم ۳: تنظیم احراز هویت و امنیت API](#قدم-۳-تنظیم-احراز-هویت-و-امنیت-api)
- [قدم ۴: تنظیم CORS](#قدم-۴-تنظیم-cors)
- [قدم ۵: کار با داده‌های API](#قدم-۵-کار-با-دادههای-api)
- [قدم ۶: گسترش API با ACF و Custom Post Types](#قدم-۶-گسترش-api-با-acf-و-custom-prod-types)
- [قدم ۷: اتصال به Next.js](#قدم-۷-اتصال-به-nextjs)
- [قدم ۸: بهینه‌سازی و امنیت](#قدم-۸-بهینهسازی-و-امنیت)
- [قدم ۹: ملاحظات SEO](#قدم-۹-ملاحظات-seo)
- [قدم ۱۰: آماده‌سازی برای تولید](#قدم-۱۰-آمادهسازی-برای-تولید)
- [قدم ۱۱: نگهداری و به‌روزرسانی](#قدم-۱۱-نگهداری-و-بهروزرسانی)
- [نکات تکمیلی](#نکات-تکمیلی)

---

## پیش‌نیازها

- دامنه یا زیردامنه (مثل `api.example.com`) با گواهینامه SSL فعال (HTTPS).
- دسترسی به پنل هاست (مثل cPanel) برای مدیریت فایل‌ها و دیتابیس.
- پروژه فرانت‌اند آماده (مثل Next.js یا React).
- آشنایی اولیه با مدیریت هاست و وردپرس.

---

## قدم ۱: نصب و تنظیم وردپرس در cPanel

### 1.1 ساخت دیتابیس
1. توی cPanel، به بخش **MySQL Databases** برید.
2. یه دیتابیس جدید بسازید (مثل `wp_database`).
3. یه کاربر دیتابیس (مثل `wp_user`) با رمز قوی ایجاد کنید.
4. کاربر رو به دیتابیس اضافه کنید و **All Privileges** رو بهش بدید.
5. اطلاعات دیتابیس (نام دیتابیس، نام کاربر، رمز) رو جایی ذخیره کنید.

### 1.2 دانلود و آپلود وردپرس
1. آخرین نسخه وردپرس رو از [wordpress.org](https://wordpress.org) دانلود کنید.
2. توی cPanel، به **File Manager** برید و فایل زیپ وردپرس رو توی پوشه اصلی دامنه (مثل `public_html` یا زیردامنه) آپلود کنید.
3. فایل رو استخراج کنید و محتویات پوشه `wordpress` رو به ریشه دامنه منتقل کنید.

### 1.3 نصب وردپرس
1. دامنه یا زیردامنه (مثل `api.example.com`) رو توی مرورگر باز کنید.
2. مراحل نصب وردپرس رو دنبال کنید:
   - زبان رو انتخاب کنید (مثلاً فارسی).
   - اطلاعات دیتابیس (نام دیتابیس، نام کاربر، رمز، هاست: معمولاً `localhost`) رو وارد کنید.
   - یه نام سایت، نام کاربری ادمین و رمز قوی تنظیم کنید.
3. بعد از نصب، به پیشخوان وردپرس (مثل `api.example.com/wp-admin`) وارد شید.

### 1.4 تنظیمات اولیه
- توی پیشخوان، به **تنظیمات > عمومی** برید و مطمئن شید آدرس سایت روی HTTPS تنظیم شده.
- توی **تنظیمات > پیوندهای یکتا**، ساختار پیوندها رو به «نام نوشته» تغییر بدید.
- گواهینامه SSL رو توی cPanel (بخش **Security > SSL/TLS**) چک کنید و اگه نیازه، نصب کنید.

### 1.5 تست REST API
- برای اطمینان از کارکرد API، این درخواست رو تست کنید:
  ```
  GET https://api.example.com/wp-json/wp/v2/posts
  ```
  باید یه پاسخ JSON (لیست پست‌ها یا خطای احراز هویت) دریافت کنید.

---

## قدم ۲: نصب افزونه‌های ضروری

| افزونه                | کاربرد                            | تنظیمات پیشنهادی                                      |
|-----------------------|----------------------------------|-------------------------------------------------------|
| **W3 Total Cache**    | کش و بهینه‌سازی سرعت            | فعال کردن Page Cache، مستثنی کردن `/wp-json/*`       |
| **WP-Optimize**       | بهینه‌سازی دیتابیس             | بهینه‌سازی خودکار هفتگی                            |
| **Wordfence Security**| امنیت و محدود کردن درخواست‌ها    | فعال کردن Rate Limiting برای `/wp-json/`             |
| **Imagify**           | فشرده‌سازی تصاویر              | فشرده‌سازی خودکار تصاویر                           |
| **Application Passwords** | احراز هویت API             | برای تولید رمزهای API                               |
| **ACF to REST API**   | نمایش فیلدهای سفارشی در API    | برای استفاده از ACF در API                          |

**تنظیمات مهم**:
- **W3 Total Cache**: توی **Page Cache > Never cache the following pages**، مسیر `/wp-json/*` رو اضافه کنید.
- **Wordfence**: Rate Limiting رو برای `/wp-json/` تنظیم کنید (مثلاً 10 درخواست در دقیقه برای هر IP).

---

## قدم ۳: تنظیم احراز هویت و امنیت API

### 3.1 تولید Application Password
- توی پیشخوان وردپرس، به **کاربران > پروفایل** برید.
- توی بخش **Application Passwords**، یه اسم (مثل `Frontend-App`) وارد کنید و رمز تولید کنید (مثل `abcd-1234-efgh-5678`).
- رمز رو توی فایل `.env` پروژه فرانت‌اند ذخیره کنید:
  ```env
  WP_API_USERNAME=your_username
  WP_API_KEY=abcd-1234-efgh-5678
  WP_API_URL=https://api.example.com/wp-json
  ```

### 3.2 اجباری کردن احراز هویت
این کد رو به فایل `functions.php` قالب اضافه کنید:
```php
add_filter('rest_authentication_errors', function ($result) {
    if (!is_user_logged_in() && !current_user_can('read')) {
        return new WP_Error('rest_forbidden', 'Authentication required.', ['status' => 401]);
    }
    return $result;
});
```

### 3.3 استفاده از رمز در درخواست‌ها
هدر درخواست رو این‌جوری تنظیم کنید:
```http
Authorization: Basic base64(username:appPassword)
```
مثال:
```http
Authorization: Basic dXNlcm5hbWU6YWJjZC0xMjM0LWVmZ2gtNTY3OA==
```

---

## قدم ۴: تنظیم CORS

برای محدود کردن دسترسی به API فقط به دامنه‌های مجاز:
- توی فایل `.htaccess` وردپرس این کد رو اضافه کنید:
```apache
<IfModule mod_headers.c>
    SetEnvIf Origin "http(s)?://(www\.)?(example.com|localhost(:[0-9]+)?)$" AccessControlAllowOrigin=$0
    Header add Access-Control-Allow-Origin %{AccessControlAllowOrigin}e env=AccessControlAllowOrigin
    Header set Access-Control-Allow-Methods "GET, POST, OPTIONS"
    Header set Access-Control-Allow-Headers "Content-Type, Authorization"
</IfModule>
```
- `example.com` رو با دامنه فرانت‌اند عوض کنید.
- توی محیط تولید، `localhost(:[0-9]+)?` رو حذف کنید.

---

## قدم ۵: کار با داده‌های API

### 5.1 بررسی اتصال به API
برای چک کردن در دسترس بودن API:
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
نمونه درخواست برای ساخت پست با فیلدهای سفارشی:
```javascript
const createPost = async () => {
  try {
    const res = await fetch('https://api.example.com/wp-json/wp/v2/posts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: 'Basic ' + btoa('username:appPassword'),
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
Authorization: Basic base64(username:appPassword)
```

### 5.6 بررسی وجود کاربر
برای چک کردن وجود کاربر با نام کاربری یا ایمیل:
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
| کد خطا | توضیح                         | راه‌حل                                  |
|--------|-----------------------------|-----------------------------------------|
| 401    | احراز هویت ناموفق          | چک کردن نام کاربری و Application Password |
| 403    | دسترسی غیرمجاز             | بررسی سطح دسترسی کاربر (مثل `read`)    |
| 429    | تعداد درخواست زیاد         | تنظیم Rate Limiting در Wordfence        |
| 500    | خطای سرور                  | بررسی لاگ‌ها در `wp-content/debug.log`  |

---

## قدم ۶: گسترش API با ACF و Custom Post Types

### 6.1 استفاده از ACF
- افزونه‌های **Advanced Custom Fields (ACF)** و **ACF to REST API** رو نصب کنید.
- فیلدهای سفارشی رو به پست‌ها یا صفحات اضافه کنید تا توی API نمایش داده بشن.

#### مثال ۱: فیلد متنی (Summary)
1. توی پیشخوان، به **Custom Fields > Add New** برید.
2. یه گروه فیلد بسازید (مثل `Post Fields`) و یه فیلد متنی با نام `summary` اضافه کنید.
3. مکان رو روی «پست‌ها» تنظیم کنید.
4. با افزونه ACF to REST API، این فیلد توی پاسخ API ظاهر می‌شه:
   ```json
   {
     "id": 123,
     "title": { "rendered": "عنوان پست" },
     "acf": { "summary": "خلاصه پست" }
   }
   ```

#### مثال ۲: فیلد تصویر
1. یه فیلد تصویر با نام `main_image` بسازید.
2. توی API، آدرس تصویر رو دریافت کنید:
   ```json
   {
     "acf": {
       "main_image": {
         "url": "https://api.example.com/wp-content/uploads/image.jpg"
       }
     }
   }
   ```
3. توی Next.js، تصویر رو نمایش بدید:
   ```javascript
   <img src={post.acf.main_image.url} alt="Main Image" />
   ```

#### مثال ۳: فیلد URL
1. یه فیلد URL با نام `video_link` بسازید.
2. توی API:
   ```json
   {
     "acf": { "video_link": "https://youtube.com/video" }
   }
   ```
3. توی فرانت‌اند:
   ```javascript
   <a href={post.acf.video_link}>مشاهده ویدیو</a>
   ```

### 6.2 Custom Post Type
برای اضافه کردن نوع محتوای سفارشی (مثل اخبار):
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
- حالا می‌تونید اخبار رو از API بگیرید:
  ```http
  GET https://api.example.com/wp-json/wp/v2/news
  ```

---

## قدم ۷: اتصال به Next.js

نمونه کد برای دریافت و نمایش پست‌ها:
```javascript
// lib/getPosts.js
import striptags from 'striptags';

export async function getPosts() {
  try {
    const res = await fetch(`${process.env.WP_API_URL}/wp/v2/posts?per_page=10`, {
      headers: {
        Authorization: `Basic ${Buffer.from(`${process.env.WP_API_USERNAME}:${process.env.WP_API_KEY}`).toString('base64')}`
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

// page.js
import { getPosts } from './lib/getPosts';

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

---

## قدم ۸: بهینه‌سازی و امنیت

- **بهینه‌سازی**:
  - دیتابیس رو با **WP-Optimize** به‌صورت هفتگی بهینه کنید.
  - تصاویر رو با **Imagify** فشرده کنید.
- **امنیت**:
  - دسترسی به `/wp-admin` رو محدود کنید:
    ```apache
    <Directory "/path/to/wp-admin">
        Require ip 192.168.1.1
    </Directory>
    ```
  - مسیرهای حساس (مثل `/wp/v2/users`) رو غیرفعال کنید:
    ```php
    add_filter('rest_endpoints', function ($endpoints) {
        unset($endpoints['/wp/v2/users']);
        return $endpoints;
    });
    ```

---

## قدم ۹: ملاحظات SEO

- از افزونه **Yoast SEO** برای تولید متادیتا استفاده کنید.
- متادیتا رو از API بگیرید:
  ```http
  GET https://api.example.com/wp-json/yoast/v1/get_head?url=/post-slug
  ```
- توی Next.js:
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

## قدم ۱۰: آماده‌سازی برای تولید

- **تست‌ها**:
  - درخواست بدون احراز هویت → خطای 401.
  - درخواست از دامنه غیرمجاز → خطای CORS.
  - دسترسی به پست‌های منتشرنشده → خطای 403.
- **تنظیمات**:
  - توی `wp-config.php`، دیباگ رو غیرفعال کنید:
    ```php
    define('WP_DEBUG', false);
    ```

---

## قدم ۱۱: نگهداری و به‌روزرسانی

- وردپرس، افزونه‌ها و قالب‌ها رو به‌روز نگه دارید.
- با **UpdraftPlus** بک‌آپ‌های روزانه یا هفتگی بگیرید و بازیابی رو تست کنید.
- لاگ‌های Wordfence رو برای شناسایی حملات بررسی کنید.

---

## نکات تکمیلی

- از **SWR** یا **React Query** برای کش کلاینت استفاده کنید.
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
- برای تصاویر بهینه، از `srcset` توی فرانت‌اند استفاده کنید.