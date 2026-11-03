# مستند فنی و راهنمای جامع API شرکت کد پیام (CodePayam)

این مستند به منظور راهنمایی توسعه‌دهندگان، تسریع در یکپارچه‌سازی و پیاده‌سازی سرویس **CodePayam API** جهت ارسال پین‌کد و دریافت گزارش‌های سامانه تدوین شده است.

---

## ۱. مشخصات عمومی سرویس

* **معماری:** RESTful API
* **متد ارسال درخواست:** `GET` / `POST`
* **فرمت خروجی:** Plain Text (`text/plain`)
* **آدرس پایه (Base URL):**
  ```http
  https://{subdomain}.codepayam.ir/codepayamapi
  ```
  > **نکته:** عبارت `{subdomain}` (یا همان `xx`) نشان‌دهنده زیردامنه اختصاصی شرکت یا مجموعه مشتری در سامانه کد پیام است.

---

## ۲. امنیت و احراز هویت (IP Whitelisting)

دسترسی به این API **صرفاً از طریق IP اختصاصی سرور مشتری** امکان‌پذیر است.

* قبل از ارسال درخواست، لازم است IP سرور خود را جهت قرار گرفتن در فهرست مجاز (**Whitelist**) به پشتیبانی کد پیام اعلام نمایید.
* در صورتی که درخواستی از IP غیرمجاز یا ثبت‌نشده ارسال شود، سرویس پاسخ زیر را برمی‌گرداند:

```text
سرویس موقتا غیرفعال می باشد.
[IP_ADDRESS]
```
*(که در آن `[IP_ADDRESS]` برابر با IP سرور درخواست‌دهنده است).*

---

## ۳. محدودیت نرخ درخواست (Rate Limit)

* به طور پیش‌فرض، هیچ‌گونه **Rate Limit** عددی روی سرور تنظیم نشده است.
* **هشدار:** در صورت شناسایی فعالیت‌های غیرعادی (مانند ارسال درخواست‌های هم‌زمان/رباتیک و ایجاد ترافیک سنگین غیرمجاز)، IP سرور به‌صورت خودکار **مسدود (Block)** شده و پس از بررسی، محدودیت نرخ درخواست سخت‌گیرانه‌ای روی آن اعمال خواهد شد.

---

## ۴. پارامترهای ورودی (Query Parameters)

برای ارسال پین‌کد یا کدهای دستوری، پارامترهای زیر باید در رشته کوئری (Query String) ارسال شوند:

| نام پارامتر | نوع داده | اجباری | توضیحات |
| :--- | :--- | :---: | :--- |
| `Destination` | String | بله | مقدار ثابت `CodePayamAPI` |
| `Source` | String | بله | شماره تلفن همراه کاربر (به عنوان مثال: `989125702670`) |
| `ReceiveTime` | Float / String | بله | زمان ارسال درخواست به فرمت استاندارد **Unix Epoch Timestamp** (به عنوان مثال: `1437050760.129`) |
| `MsgBody` | String | بله | پین‌کد مدنظر کاربر **یا** کدهای گزارش‌گیری (ارقام ۱ تا ۹) |

---

## ۵. نحوه فراخوانی سرویس

### نمونه ساختار درخواست (HTTP GET)

```http
GET /codepayamapi?Destination=CodePayamAPI&Source=989125702670&ReceiveTime=1437050760.129&MsgBody=111 HTTP/1.1
Host: xx.codepayam.ir
```

---

## ۶. کدهای گزارش‌گیری و پیام‌های خروجی

### ۶.۱ کدهای رزرو شده (1 تا 9)
در صورتی که بخش گزارش‌ها برای حساب شما فعال باشد، کدهای ۱ تا ۹ جهت دریافت اطلاعات سیستمی اختصاص یافته‌اند. برای دریافت گزارش کافی است شماره کد موردنظر را در پارامتر `MsgBody` ارسال نمایید:

* **کد `1` (پیش‌فرض):** استعلام و نمایش میزان **موجودی/اعتبار مشتری** در سیستم باشگاه مشتریان کد پیام.

### ۶.۲ مدیریت پیام‌های خروجی
* تمامی پیام‌های متنی خروجی (اعم از پیام ثبت موفق، اشتباه بودن کد، تکراری بودن کد و...) در **پنل اختصاصی مشتری** قابل تنظیم و سفارشی‌سازی هستند.
* تنظیمات اعمال‌شده در پنل به صورت یکپارچه روی تمامی زیرسیستم‌ها (**API**، **ربات / Bot** و **SMS**) اعمال خواهند شد.

---

## ۷. مدیریت خطاها و پاسخ‌دهی سیستم

هنگام بروز خطا در پردازش درخواست‌ها، سیستم کد پیام مکانیزم‌های زیر را جهت عیب‌یابی و پایش اعمال می‌کند:

* **ثبت لاگ و تیکت خودکار:** در صورت بروز خطاهای سیستمی یا نامعتبر، یک لاگ سیستمی و یک **تیکت اختصاصی** در سامانه کد پیام ثبت می‌گردد تا تیم پشتیبانی یا فنی بتوانند موضوع را پیگیری کنند.
* **خروجی متنی (Plain Text):** اطلاعات کلی و پیام خطای مربوطه مستقیماً در قالب متن خالص (`text/plain`) به عنوان پاسخ به درخواست‌کننده ارسال می‌شود.

### جدول سناریوها و پاسخ‌های رایج سیستم

| وضعیت / خطا | نمونه پاسخ خروجی سرور (`text/plain`) | توضیحات و اقدامات لازم |
| :--- | :--- | :--- |
| **دسترسی غیرمجاز (IP غیرمجاز)** | `سرویس موقتا غیرفعال می باشد. [IP]` | آی‌پی سرور شما در Whitelist کد پیام ثبت نشده است. (اعلام IP به پشتیبانی جهت ثبت) |
| **پین‌کد تکراری** | *(مقدار تنظیم‌شده در پنل مشتری)* | این کد قبلاً ثبت/مصرف شده است. پیام خروجی از طریق پنل اختصاصی قابل سفارشی‌سازی است. |
| **پین‌کد نامعتبر / اشتباه** | *(مقدار تنظیم‌شده در پنل مشتری)* | پین‌کد وارد شده معتبر نیست یا در سیستم وجود ندارد. |
| **موجودی حساب (گزارش کد ۱)** | *(عدد یا متن موجودی اعتباری)* | نمایش میزان اعتبار یا موجودی فعلی در باشگاه مشتریان. |
| **خطای سیستمی / سرور** | `خطای سیستمی رخ داد. (تیکت #ID ثبت شد)` | بروز خطای پیش‌بینی نشده؛ سیستم به‌طور خودکار تیکت پشتیبانی ایجاد می‌کند. |

---

## ۸. نکات امنیتی و اعتبارسنجی سمت مشتری (Client-Side Validation)

تمامی الزامات صحت‌سنجی و اعتبارسنجی ورودی‌ها پیش از ارسال درخواست به سرور کد پیام باید **توسط سیستم مشتری** پیاده‌سازی و کنترل شود تا از ایجاد ترافیک نامعتبر جلوگیری گردد.

### ۸.۱ اعتبارسنجی شماره موبایل کاربر (`Source`)
* **فرمت عددی:** پارامتر `Source` باید صرفاً شامل ارقام (بدون کاراکترهای اضافی مثل `+`، خط فاصله یا فاصله خالی) باشد.
* **ساختار شماره:** بر اساس استانداردهای مخابراتی، شماره‌ها باید با کد کشور آغاز شوند (مانند `989125702670` برای ایران).
* **طول و الگو:** بررسی طول استاندارد شماره موبایل (به عنوان مثال برای ایران، شروع با `98` و داشتن طول کلی ۱۲ رقم).

### ۸.۲ اعتبارسنجی پین‌کد (`MsgBody`)
* **بررسی کدهای گزارش (۱ تا ۹):** 
  پین‌کد وارد شده توسط کاربر **نباید** جزو کدهای رزرو شده گزارش‌گیری (اعداد ۱ تا ۹) باشد، مگر اینکه کاربر قصد دریافت گزارش‌های سیستمی را داشته باشد. در صورت ارسال پین‌کد معمولی، مقدار آن نباید در این بازه تک‌رقمی قرار گیرد.
* **طول و ساختار کاراکترها:** 
  پین‌کدها باید کاملاً عددی بوده و طول آن‌ها بر اساس استاندارد تعریف‌شده در پنل مشتری بررسی شود (به عنوان مثال، **فقط ارقام با طول بین ۷ تا ۲۰ رقم**).
* **مطابقت پیشوند محصول (Product Code Prefix):** 
  بررسی اینکه پین‌کد ارسالی با لیست کدهای ۳ رقمی معتبر محصولات که در پنل کد پیام تعریف شده‌اند، مطابقت داشته باشد (پیشوند پین‌کد باید با یکی از کدهای شناخته‌شده در پنل همخوانی داشته باشد).
* **محافظت از فرم‌های وب (مشتری / کلاینت):** 
  در صورتی که پین‌کد از طریق فرم‌های وب یا اینترنتی از کاربر دریافت می‌شود، پیاده‌سازی مکانیزم‌های امنیتی نظیر **کپچا (CAPTCHA)** و **محدودسازی نرخ درخواست (Rate Limit)** در سطح فرم جهت جلوگیری از ارسال درخواست‌های رباتیک و اسپم الزامی است.

---

## ۹. نمونه کدهای پیاده‌سازی

### ۹.۱ cURL
```bash
curl -X GET "https://xx.codepayam.ir/codepayamapi?Destination=CodePayamAPI&Source=989125702670&ReceiveTime=1700000000.000&MsgBody=111"
```

### ۹.۲ Python (`requests`)
```python
import requests
import time

subdomain = "xx"  # زیردامنه اختصاصی شما
url = f"https://{subdomain}.codepayam.ir/codepayamapi"

params = {
    "Destination": "CodePayamAPI",
    "Source": "989125702670",
    "ReceiveTime": f"{time.time():.3f}",
    "MsgBody": "111"  # پین‌کد یا کد گزارش
}

response = requests.get(url, params=params)
print("Response Status:", response.status_code)
print("Response Text:", response.text)
```

### ۹.۳ C# (`HttpClient`)
```csharp
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;

class CodePayamClient
{
    private static readonly HttpClient client = new HttpClient();

    public static async Task Main(string[] args)
    {
        string subdomain = "xx"; // زیردامنه اختصاصی شرکت
        string url = $"https://{subdomain}.codepayam.ir/codepayamapi";

        // محاسبه زمان به فرمت Unix Epoch Timestamp
        string receiveTime = DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString() + "." + DateTime.Now.Millisecond;

        var queryParams = new Dictionary<string, string>
        {
            { "Destination", "CodePayamAPI" },
            { "Source", "989125702670" },
            { "ReceiveTime", receiveTime },
            { "MsgBody", "111" } // پین‌کد یا کد گزارش
        };

        // ساخت URL همراه با پارامترها
        var content = new FormUrlEncodedContent(queryParams);
        string queryString = await content.ReadAsStringAsync();
        string fullUrl = $"{url}?{queryString}";

        try
        {
            HttpResponseMessage response = await client.GetAsync(fullUrl);
            string responseString = await response.Content.ReadAsStringAsync();

            Console.WriteLine("Response Status: " + response.StatusCode);
            Console.WriteLine("Response Text: " + responseString);
        }
        catch (HttpRequestException e)
        {
            Console.WriteLine("Request error: " + e.Message);
        }
    }
}
```

### ۹.۴ PHP (`cURL`)
```php
<?php

$subdomain = "xx"; // زیردامنه اختصاصی شرکت
$baseUrl = "https://{$subdomain}.codepayam.ir/codepayamapi";

// آماده‌سازی پارامترهای ورودی
$params = [
    'Destination' => 'CodePayamAPI',
    'Source'      => '989125702670',
    'ReceiveTime' => microtime(true), // زمان ارسال به فرمت Unix Epoch با اعشار
    'MsgBody'     => '111'            // پین‌کد یا کد گزارش (مثلاً 1 برای موجودی)
];

// ساخت URL نهایی همراه با کوئری‌استرینگ
$url = $baseUrl . '?' . http_build_query($params);

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_TIMEOUT, 30);

$response = curl_exec($ch);
$httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);

if (curl_errno($ch)) {
    echo "cURL Error: " . curl_error($ch);
} else {
    echo "HTTP Status Code: " . $httpCode . "\n";
    echo "Response Text: " . $response . "\n";
}

curl_close($ch);

?>
```