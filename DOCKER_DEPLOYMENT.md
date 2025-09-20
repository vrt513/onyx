# راه‌اندازی پروژه Onyx با Docker

این راهنما برای راه‌اندازی پروژه Onyx با استفاده از Docker Compose در محیط production توضیح داده شده است.

## پیش‌نیازها

- Docker و Docker Compose نصب شده باشند
- حداقل 8GB RAM
- حداقل 50GB فضای دیسک
- اتصال اینترنت برای دانلود تصاویر

## مراحل راه‌اندازی

### ۱. کلون کردن پروژه

```bash
git clone <repository-url>
cd onyx
```

### ۲. تنظیم متغیرهای محیطی

#### روش اول: استفاده از فایل template

```bash
# کپی فایل template
cp .env.template .env
```

#### روش دوم: ایجاد فایل .env به صورت دستی

یک فایل `.env` در ریشه پروژه ایجاد کنید و محتوای زیر را در آن قرار دهید:

```env
# تنظیمات اصلی
WEB_DOMAIN=http://localhost:3000

# تنظیمات AI API (حداقل یکی از این‌ها را تنظیم کنید)
OPENAI_API_KEY=your_openai_api_key_here
ANTHROPIC_API_KEY=your_anthropic_api_key_here
GEMINI_API_KEY=your_gemini_api_key_here

# تنظیمات Authentication
AUTH_TYPE=google_oauth
GOOGLE_OAUTH_CLIENT_ID=your_google_client_id
GOOGLE_OAUTH_CLIENT_SECRET=your_google_client_secret
SECRET=your_random_secret_key

# تنظیمات Database
POSTGRES_USER=postgres
POSTGRES_PASSWORD=secure_password_change_this

# تنظیمات MinIO
MINIO_ROOT_PASSWORD=secure_minio_password_change_this
```

### ۳. تنظیم کلیدهای API

#### OpenAI API
1. به [OpenAI Platform](https://platform.openai.com/api-keys) بروید
2. یک API key جدید ایجاد کنید
3. آن را در فایل `.env` قرار دهید:

```env
OPENAI_API_KEY=sk-your-openai-api-key-here
```

#### Anthropic API
1. به [Anthropic Console](https://console.anthropic.com/) بروید
2. یک API key ایجاد کنید
3. آن را در فایل `.env` قرار دهید:

```env
ANTHROPIC_API_KEY=sk-ant-your-anthropic-api-key-here
```

#### Google Gemini API
1. به [Google AI Studio](https://makersuite.google.com/app/apikey) بروید
2. یک API key ایجاد کنید
3. آن را در فایل `.env` قرار دهید:

```env
GEMINI_API_KEY=your-gemini-api-key-here
```

### ۴. تنظیم Google OAuth (اختیاری)

اگر می‌خواهید از Google OAuth استفاده کنید:

1. به [Google Cloud Console](https://console.cloud.google.com/) بروید
2. یک پروژه جدید ایجاد کنید یا پروژه موجود را انتخاب کنید
3. OAuth 2.0 Client ID ایجاد کنید
4. Authorized redirect URIs را تنظیم کنید:
   - `http://localhost:3000/auth/callback/google`
5. Client ID و Client Secret را در فایل `.env` قرار دهید:

```env
GOOGLE_OAUTH_CLIENT_ID=your-google-client-id
GOOGLE_OAUTH_CLIENT_SECRET=your-google-client-secret
```

### ۵. راه‌اندازی پروژه

```bash
# ساخت و راه‌اندازی تمام سرویس‌ها
docker-compose up -d

# مشاهده لاگ‌ها
docker-compose logs -f

# متوقف کردن تمام سرویس‌ها
docker-compose down
```

### ۶. بررسی وضعیت راه‌اندازی

#### بررسی سرویس‌ها

```bash
# مشاهده وضعیت تمام سرویس‌ها
docker-compose ps

# بررسی سلامت سرویس‌ها
docker-compose exec api_server curl -f http://localhost:8080/health
docker-compose exec web_server curl -f http://localhost:3000
```

#### دسترسی به سرویس‌ها

- **Web Interface**: http://localhost:3000
- **API Server**: http://localhost:8080
- **MinIO Console**: http://localhost:9001
  - Username: minioadmin
  - Password: مقدار MINIO_ROOT_PASSWORD در فایل .env
- **Vespa**: http://localhost:19071
- **Redis**: localhost:6379

## تنظیمات پیشرفته

### استفاده از GPU

اگر GPU در دسترس دارید، می‌توانید از فایل docker-compose.gpu-dev.yml استفاده کنید:

```bash
# برای سیستم‌های با GPU
docker-compose -f docker-compose.gpu-dev.yml up -d
```

### تنظیمات شبکه

اگر نیاز به تنظیمات شبکه خاصی دارید، از فایل `docker-compose.override.yml` استفاده کنید.

### تنظیمات SSL/HTTPS

برای محیط production واقعی، توصیه می‌شود از reverse proxy مانند Nginx با Let's Encrypt استفاده کنید.

## عیب‌یابی

### مشکلات رایج

#### ۱. خطای اتصال به پایگاه داده

```
Error: connect ECONNREFUSED 127.0.0.1:5432
```

**راه‌حل**: مطمئن شوید PostgreSQL container در حال اجراست:

```bash
docker-compose ps relational_db
docker-compose logs relational_db
```

#### ۲. خطای API Key

```
Error: Invalid API key
```

**راه‌حل**: بررسی کنید که API key های شما معتبر هستند و به درستی در فایل `.env` تنظیم شده‌اند.

#### ۳. خطای پورت اشغال شده

```
Error: Port already in use
```

**راه‌حل**: پورت‌های مورد استفاده را تغییر دهید یا سرویس‌های دیگر را متوقف کنید.

### پاک کردن داده‌ها

برای پاک کردن تمام داده‌ها و شروع دوباره:

```bash
# متوقف کردن سرویس‌ها
docker-compose down

# پاک کردن volumes
docker-compose down -v

# پاک کردن images (اختیاری)
docker-compose down --rmi all
```

### لاگ‌ها

برای مشاهده لاگ‌های سرویس‌ها:

```bash
# لاگ‌های تمام سرویس‌ها
docker-compose logs

# لاگ‌های یک سرویس خاص
docker-compose logs api_server

# دنبال کردن لاگ‌ها به صورت real-time
docker-compose logs -f api_server
```

## پشتیبانی

اگر با مشکل مواجه شدید:

1. لاگ‌های کامل را بررسی کنید: `docker-compose logs`
2. وضعیت سرویس‌ها را بررسی کنید: `docker-compose ps`
3. مطمئن شوید تمام متغیرهای محیطی به درستی تنظیم شده‌اند
4. بررسی کنید که پورت‌های مورد نیاز آزاد هستند

## تنظیمات امنیتی

### برای محیط Production

1. **رمزهای عبور قوی**: از رمزهای عبور قوی و تصادفی استفاده کنید
2. **شبکه**: از firewall استفاده کنید و فقط پورت‌های لازم را باز کنید
3. **SSL/TLS**: از HTTPS استفاده کنید
4. **بروزرسانی منظم**: Docker images را به طور منظم بروزرسانی کنید
5. **Backup**: از داده‌های خود backup منظم بگیرید

### متغیرهای امنیتی مهم

```env
# استفاده از رمزهای عبور قوی
POSTGRES_PASSWORD=very_strong_password_123!
MINIO_ROOT_PASSWORD=another_strong_password_456!
SECRET=very_long_random_secret_key_789!

# محدود کردن دسترسی
VALID_EMAIL_DOMAINS=yourcompany.com
REQUIRE_EMAIL_VERIFICATION=true
```

## بهینه‌سازی عملکرد

### تنظیمات Memory

اگر با محدودیت memory مواجه هستید:

```yaml
# در docker-compose.override.yml
services:
  relational_db:
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G
```

### تنظیمات CPU

```yaml
services:
  inference_model_server:
    deploy:
      resources:
        limits:
          cpus: '2.0'
```

## مهاجرت به Production

وقتی آماده مهاجرت به محیط production واقعی بودید:

1. دامنه واقعی تنظیم کنید
2. SSL certificate راه‌اندازی کنید
3. Database را backup کنید
4. تنظیمات امنیتی را تقویت کنید
5. Monitoring راه‌اندازی کنید
