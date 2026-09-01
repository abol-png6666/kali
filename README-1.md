# Kali Web Desktop with Selkies & Cloudflare Tunnel

یک محیط **Kali Linux Web Desktop** مبتنی بر **Selkies** که داخل GitHub Actions اجرا می‌شود و از طریق **Cloudflare Quick Tunnel** به اینترنت قابل دسترسی است.

این پروژه برای اجرای یک دسکتاپ گرافیکی Kali داخل Runner گیت‌هاب و دسترسی به آن از طریق مرورگر طراحی شده است. در کنار دسکتاپ، تنظیمات مخصوص کاهش مصرف پهنای‌باند، 60 FPS، رزولوشن 1280×720 و چند لایه اصلاح مشکلات کیبورد هم دارد.

> **نکته امنیتی:** این پروژه یک دسکتاپ را از طریق یک آدرس عمومی موقت روی اینترنت قرار می‌دهد. از رمز قوی استفاده کنید و اطلاعات حساس را داخل دسکتاپ باز نکنید.

---

## ✨ امکانات

- اجرای Kali Linux در GitHub Actions
- دسکتاپ گرافیکی تحت وب با Selkies
- ایجاد لینک عمومی موقت با Cloudflare Quick Tunnel
- اجرای Workflow فقط با `workflow_dispatch`
- نام کاربری پیش‌فرض دسکتاپ: `desktop`
- رمز عبور از GitHub Actions Secret خوانده می‌شود
- رزولوشن ثابت:
  - `1280×720`
- نرخ فریم:
  - `60 FPS`
- تنظیمات ویدیویی کم‌مصرف:
  - H.264 / x264
  - CRF 28
  - Paint-over CRF 20
  - Bitrate حدود 2 Mbps
  - Congestion Control فعال
- Audio فعال با bitrate برابر `128 kbps`
- Microphone غیرفعال
- Firefox ESR نصب‌شده
- فونت‌های Noto و Emoji
- ابزارهای X11 برای مدیریت کیبورد و ورودی
- سه لایه برای کاهش مشکلات Shift / CapsLock / Keyboard Mapping
- Wallpaper خودکار
- تست کامل سرویس محلی Selkies قبل از ساخت Tunnel
- تست دوباره سرویس از طریق Public URL
- نمایش لینک دسکتاپ داخل GitHub Actions Summary

---

# 🧱 معماری پروژه

جریان اجرای پروژه به شکل زیر است:

```text
GitHub Actions Runner
        │
        ▼
   Docker Build
        │
        ▼
 Kali + Selkies Image
        │
        ▼
 selkies-desktop container
        │
        ├── 127.0.0.1:3000
        │
        ▼
 Local HTTP Test
        │
        ▼
 Cloudflare Quick Tunnel
        │
        ▼
 https://xxxxx.trycloudflare.com
        │
        ▼
 Browser
```

در سمت Container، تصویر Kali از این image ساخته می‌شود:

```text
lscr.io/linuxserver/kali-linux:latest
```

و بعد ابزارها، Firefox، تنظیمات Keyboard و اسکریپت‌های سفارشی به آن اضافه می‌شوند.

---

# 📁 ساختار پیشنهادی Repository

حداقل فایل موردنیاز:

```text
.github/
└── workflows/
    └── kali-desktop.yml
```

فایل Workflow همان YAML اصلی پروژه است.

در زمان اجرای Workflow، فایل‌ها و Dockerfile لازم به‌صورت داینامیک ساخته می‌شوند:

```text
Dockerfile

root/
├── defaults/
│   └── .config/
│       └── autostart/
│           ├── wallpaper.desktop
│           ├── keyboard-fix.desktop
│           ├── shift-repair.desktop
│           └── keymap-daemon.desktop
│
├── etc/
│   └── xdg/
│       └── autostart/
│           ├── wallpaper.desktop
│           ├── keyboard-fix.desktop
│           ├── shift-repair.desktop
│           └── keymap-daemon.desktop
│
└── usr/
    └── local/
        └── bin/
            ├── set-wallpaper
            ├── fix-keyboard
            ├── shift-repair.sh
            └── keymap-daemon.py
```

این ساختار روی Runner ساخته می‌شود و لازم نیست همه این فایل‌ها را از قبل داخل ریپو قرار دهید.

---

# ✅ پیش‌نیازها

برای اجرای پروژه به این موارد نیاز دارید:

1. یک GitHub Repository
2. GitHub Actions فعال
3. Secret با نام:

```text
SELKIES_PASSWORD
```

4. یک فایل Workflow در:

```text
.github/workflows/
```

---

# 🔐 ساخت Secret

برای ساخت رمز دسکتاپ:

### 1. وارد Repository شوید

به این مسیر بروید:

```text
Settings
→ Secrets and variables
→ Actions
```

### 2. روی:

```text
New repository secret
```

بزنید.

### 3. مقدارها:

Name:

```text
SELKIES_PASSWORD
```

Secret:

```text
رمز قوی خودتان
```

### مثال

```text
Name: SELKIES_PASSWORD
Value: MyStrongDesktopPassword_2026
```

> رمز را مستقیم داخل YAML قرار ندهید.

---

# ▶️ اجرای Workflow

چون Workflow با این Trigger اجرا می‌شود:

```yaml
on:
  workflow_dispatch:
```

باید آن را به‌صورت دستی اجرا کنید.

در GitHub:

```text
Actions
→ Kali Web Desktop with Selkies & Cloudflare Tunnel
→ Run workflow
```

بعد Workflow را اجرا کنید.

---

# 🚀 مراحل اجرای Workflow

Workflow به‌ترتیب این مراحل را انجام می‌دهد:

## 1. بررسی Secret

ابتدا بررسی می‌شود که:

```text
SELKIES_PASSWORD
```

وجود داشته باشد.

اگر Secret خالی باشد Workflow متوقف می‌شود.

خطای احتمالی:

```text
Secret SELKIES_PASSWORD is missing.
```

---

# 2. ساخت فایل‌های سفارشی Kali

Workflow این پوشه‌ها را ایجاد می‌کند:

```bash
mkdir -p root/defaults/.config/autostart \
         root/usr/local/bin \
         root/etc/xdg/autostart
```

بعد Dockerfile و اسکریپت‌های مربوط به Keyboard و Wallpaper ساخته می‌شوند.

---

# 3. ساخت Docker Image

Docker image با دستور زیر ساخته می‌شود:

```bash
docker build --pull --tag kali-selkies:local .
```

Image نهایی:

```text
kali-selkies:local
```

---

# 4. اجرای Container

Container با این نام اجرا می‌شود:

```text
selkies-desktop
```

و پورت داخلی Selkies:

```text
3000
```

فقط روی localhost Runner expose می‌شود:

```text
127.0.0.1:3000:3000
```

یعنی سرویس مستقیماً روی اینترنت باز نمی‌شود و بعداً Cloudflare Tunnel آن را در دسترس قرار می‌دهد.

---

# 🖥️ تنظیمات Desktop

متغیرهای اصلی:

```text
DESKTOP_USER=desktop
```

و رمز:

```text
DESKTOP_PASSWORD=${{ secrets.SELKIES_PASSWORD }}
```

در Container:

```text
CUSTOM_USER=desktop
PASSWORD=<secret>
```

عنوان دسکتاپ:

```text
Kali Web Desktop
```

---

# 🎮 تنظیمات Video

پروژه با هدف داشتن تصویر نسبتاً روان و مصرف اینترنت کنترل‌شده تنظیم شده است.

تنظیمات:

```text
Framerate:
60 FPS

Resolution:
1280×720

Encoder:
x264 / JPEG

H.264 CRF:
28

Paint-over CRF:
20

Video bitrate:
2000 kbps

JPEG quality:
55

Paint-over JPEG quality:
85

Congestion control:
Enabled
```

متغیرهای مربوط:

```yaml
-e SELKIES_FRAMERATE="60"
-e SELKIES_H264_CRF="28"
-e SELKIES_H264_PAINTOVER_CRF="20"
-e SELKIES_VIDEO_BITRATE="2000"
-e SELKIES_JPEG_QUALITY="55"
-e SELKIES_USE_PAINT_OVER_QUALITY="true"
-e SELKIES_PAINT_OVER_JPEG_QUALITY="85"
-e SELKIES_CONGESTION_CONTROL="true"
```

---

# 📐 Resolution ثابت

برای جلوگیری از تغییر خودکار رزولوشن:

```text
SELKIES_IS_MANUAL_RESOLUTION_MODE="true|locked"
SELKIES_MANUAL_WIDTH="1280"
SELKIES_MANUAL_HEIGHT="720"
```

و resize غیرفعال است:

```text
SELKIES_ENABLE_RESIZE="false"
```

در نتیجه دسکتاپ روی:

```text
1280×720
```

قفل می‌شود.

---

# 🔊 Audio

Audio فعال است:

```text
SELKIES_AUDIO_ENABLED=true|locked
```

Bitrate:

```text
128000
```

یعنی:

```text
128 kbps
```

Microphone غیرفعال است:

```text
SELKIES_MICROPHONE_ENABLED=false|locked
```

---

# ⌨️ سیستم اصلاح Keyboard

یکی از بخش‌های اصلی پروژه، اصلاح مشکلات Keyboard در Selkies و مخصوصاً بعضی Clientهای موبایل است.

این سیستم سه لایه دارد:

```text
Layer 1
setxkbmap loop

Layer 2
CapsLock / modifier repair

Layer 3
Python X11 / XTest daemon
```

---

# Layer 1 — fix-keyboard

فایل:

```text
/usr/local/bin/fix-keyboard
```

این اسکریپت منتظر بالا آمدن X Server می‌ماند و سپس مرتب Keymap را به حالت استاندارد US برمی‌گرداند.

تنظیمات:

```text
Model:
pc105

Layout:
us

Variant:
empty

Options:
empty
```

دستورهای اصلی:

```bash
setxkbmap -option ""
setxkbmap -model pc105 -layout us -variant "" -option ""
```

همچنین:

```bash
xset r rate 300 30
```

برای تنظیم auto-repeat استفاده می‌شود.

---

# Layer 2 — shift-repair.sh

فایل:

```text
/usr/local/bin/shift-repair.sh
```

این اسکریپت به‌صورت مداوم وضعیت CapsLock را بررسی می‌کند.

اگر CapsLock به‌اشتباه روشن شده باشد، آن را خاموش می‌کند.

منطق اصلی:

```bash
if xset q | grep -q "Caps Lock:.*on"; then
    xdotool key --clearmodifiers Caps_Lock
fi
```

این لایه مخصوصاً برای بعضی رفتارهای نامناسب کیبوردهای نرم‌افزاری موبایل مفید است.

---

# Layer 3 — keymap-daemon.py

فایل:

```text
/usr/local/bin/keymap-daemon.py
```

این بخش با Python و X11 کار می‌کند و برای مانیتور کردن Eventهای KeyPress طراحی شده است.

کتابخانه‌های استفاده‌شده:

```text
python3-xlib
X11 RECORD
XTEST
```

Daemon بررسی می‌کند آیا وضعیت modifierها با key eventها ناسازگاری دارد یا نه.

همچنین در صورت مشاهده بعضی حالت‌های مشکل‌ساز CapsLock، تلاش می‌کند وضعیت را اصلاح کند.

اگر extension مربوط به RECORD در X Server موجود نباشد، برنامه خطا می‌دهد و از اجرای خود خارج می‌شود، اما باعث Crash شدن کل Desktop نمی‌شود.

---

# 🖼️ Wallpaper

فایل:

```text
/usr/local/bin/set-wallpaper
```

در شروع Desktop دنبال imageهایی در:

```text
/usr/share/backgrounds/
```

می‌گردد.

فرمت‌های بررسی‌شده:

```text
.jpg
.png
```

سپس در صورت امکان از این ابزارها استفاده می‌کند:

```text
xfconf-query
plasma-apply-wallpaperimage
feh
```

بنابراین Wallpaper در محیط‌های مختلف تا حد ممکن تنظیم می‌شود.

---

# 🌐 Cloudflare Quick Tunnel

بعد از اینکه Selkies روی localhost آماده شد، `cloudflared` نصب می‌شود.

دانلود:

```text
https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
```

سپس Tunnel اجرا می‌شود:

```bash
cloudflared tunnel \
  --url http://127.0.0.1:3000 \
  --protocol http2 \
  --no-autoupdate
```

Cloudflare یک آدرس موقت مانند این ایجاد می‌کند:

```text
https://example-random.trycloudflare.com
```

این آدرس داخل متغیر:

```text
PUBLIC_URL
```

قرار می‌گیرد.

---

# 🧪 تست Local

قبل از ساخت لینک عمومی، Workflow سرویس داخلی را بررسی می‌کند:

```text
http://127.0.0.1:3000/
```

احراز هویت با:

```text
desktop
+
SELKIES_PASSWORD
```

انجام می‌شود.

سپس بررسی می‌شود که HTML واقعاً متعلق به Web Client سلکیس باشد.

عبارت‌هایی که برای تشخیص استفاده می‌شوند:

```text
id="app"
manifest.json
apple-touch-icon
assets/index-*.js
```

---

# 🧪 تست Public

بعد از ساخته شدن Tunnel، همان تست روی:

```text
PUBLIC_URL
```

انجام می‌شود.

اگر Cloudflare URL کار نکند:

```text
Public URL never responded.
```

نمایش داده می‌شود و Workflow خطا می‌دهد.

---

# 📋 نمایش اطلاعات در GitHub Actions

در مرحله:

```text
Publish desktop link
```

اطلاعات مهم داخل:

```text
GITHUB_STEP_SUMMARY
```

نوشته می‌شود.

شامل:

```text
Desktop URL
Username
Resolution
FPS
Video settings
Audio settings
Keyboard fixes
```

رمز واقعی داخل Summary چاپ نمی‌شود.

---

# 🔄 Keep Alive

آخرین Step:

```text
Keep desktop alive
```

تا زمانی که Job فعال است، هر 60 ثانیه بررسی می‌کند Container هنوز اجرا می‌شود:

```bash
docker ps
```

اگر Container متوقف شده باشد:

```text
Selkies container stopped.
```

نمایش داده می‌شود و Workflow متوقف می‌شود.

---

# ⏱️ مدت اجرای Job

در YAML:

```yaml
timeout-minutes: 355
```

قرار داده شده است.

یعنی Job حداکثر تا:

```text
355 دقیقه
```

اجرا می‌شود.

این مقدار به محدودیت‌های GitHub Actions و نوع Runner شما وابسته است و نمی‌توان آن را به‌عنوان اجرای دائمی در نظر گرفت.

---

# ⚙️ Environment Variables مهم

## Desktop

```text
DESKTOP_USER=desktop
DESKTOP_PASSWORD=${{ secrets.SELKIES_PASSWORD }}
```

## Keyboard

```text
KEYBOARD=us
SELKIES_ENABLE_CLIPBOARD=true
```

## Video

```text
SELKIES_USE_CPU=true|locked
SELKIES_ENCODER=x264enc,jpeg
SELKIES_FRAMERATE=60
SELKIES_H264_CRF=28
SELKIES_H264_PAINTOVER_CRF=20
SELKIES_H264_FULLCOLOR=false
SELKIES_H264_STREAMING_MODE=true
SELKIES_VIDEO_BITRATE=2000
SELKIES_JPEG_QUALITY=55
SELKIES_USE_PAINT_OVER_QUALITY=true
SELKIES_PAINT_OVER_JPEG_QUALITY=85
SELKIES_USE_CSS_SCALING=true
SELKIES_CONGESTION_CONTROL=true
```

## Resolution

```text
SELKIES_IS_MANUAL_RESOLUTION_MODE=true|locked
SELKIES_MANUAL_WIDTH=1280
SELKIES_MANUAL_HEIGHT=720
SELKIES_ENABLE_RESIZE=false
```

## Audio

```text
SELKIES_AUDIO_ENABLED=true|locked
SELKIES_AUDIO_BITRATE=128000
SELKIES_MICROPHONE_ENABLED=false|locked
SELKIES_UI_SIDEBAR_SHOW_AUDIO_SETTINGS=true
```

---

# 📦 نرم‌افزارهای نصب‌شده داخل Image

Dockerfile این بسته‌ها را نصب می‌کند:

```text
firefox-esr
fonts-noto
fonts-noto-color-emoji
feh
x11-xkb-utils
x11-xserver-utils
xkb-data
xdotool
xbindkeys
xautomation
xclip
xsel
python3
python3-xlib
dbus-x11
```

و در صورت امکان یکی از این‌ها:

```text
libfuse2
```

یا:

```text
libfuse2t64
```

این بخش برای اجرای بهتر بعضی برنامه‌های AppImage/Electron در Container در نظر گرفته شده است.

---

# 🧩 چرا این متغیرهای Sandbox وجود دارند؟

در Docker / Web Desktop ممکن است بعضی برنامه‌های Electron، AppImage و Firefox با Sandbox خودشان مشکل داشته باشند.

برای همین در Dockerfile این موارد قرار گرفته‌اند:

```text
ELECTRON_DISABLE_SANDBOX=1
APPIMAGE_EXTRACT_AND_RUN=1
MOZ_DISABLE_CONTENT_SANDBOX=1
MOZ_DISABLE_GMP_SANDBOX=1
MOZ_DISABLE_RDD_SANDBOX=1
MOZ_DISABLE_SOCKET_PROCESS_SANDBOX=1
```

> غیرفعال کردن Sandbox می‌تواند امنیت بعضی برنامه‌ها را کاهش دهد. فقط در محیطی که به آن اعتماد دارید استفاده کنید.

---

# 🛠️ عیب‌یابی

## مشکل 1 — Secret پیدا نمی‌شود

خطا:

```text
Secret SELKIES_PASSWORD is missing.
```

راه‌حل:

در Repository بروید:

```text
Settings
→ Secrets and variables
→ Actions
```

و Secret زیر را بسازید:

```text
SELKIES_PASSWORD
```

---

## مشکل 2 — Container سریع متوقف می‌شود

اول Logs را ببینید:

```bash
docker logs selkies-desktop
```

در Workflow نیز در صورت توقف Container، Log چاپ می‌شود.

---

## مشکل 3 — Selkies روی پورت 3000 بالا نمی‌آید

تست:

```bash
curl -I http://127.0.0.1:3000/
```

و:

```bash
docker ps
```

اگر Container بالا نیست:

```bash
docker logs selkies-desktop
```

---

## مشکل 4 — Cloudflare URL ساخته نمی‌شود

Logهای زیر بررسی شوند:

```text
/tmp/cloudflared.log
/tmp/cloudflared.stdout
```

همچنین:

```bash
cloudflared --version
```

را اجرا کنید تا مطمئن شوید binary سالم نصب شده است.

---

## مشکل 5 — Keyboard دوباره خراب شد

بررسی کنید این Processها اجرا هستند:

```bash
ps aux | grep fix-keyboard
ps aux | grep shift-repair
ps aux | grep keymap-daemon
```

همچنین داخل Desktop:

```bash
setxkbmap -query
```

باید چیزی مشابه این برگرداند:

```text
rules:      evdev
model:      pc105
layout:     us
```

---

## مشکل 6 — CapsLock رفتار عجیب دارد

وضعیت X:

```bash
xset q
```

را بررسی کنید.

در صورت فعال بودن CapsLock:

```bash
xdotool key --clearmodifiers Caps_Lock
```

می‌تواند آن را خاموش کند.

---

## مشکل 7 — کیفیت تصویر پایین است

مقدار:

```text
SELKIES_H264_CRF
```

را کاهش دهید.

مثلاً:

```text
28 → 24
```

کیفیت بهتر می‌شود اما معمولاً مصرف bandwidth بیشتر خواهد شد.

---

## مشکل 8 — مصرف اینترنت زیاد است

می‌توانید:

```text
SELKIES_VIDEO_BITRATE
```

را پایین‌تر کنید.

مثلاً:

```text
2000 → 1500
```

همچنین می‌توانید JPEG quality را کاهش دهید.

---

## مشکل 9 — FPS پایین است

در این پروژه مقدار هدف:

```text
60 FPS
```

است، اما FPS واقعی فقط به این متغیر بستگی ندارد.

موارد مؤثر:

```text
CPU Runner
Encoder
Browser
Network
Cloudflare
Desktop workload
```

همچنین این پروژه CPU encoding را فعال کرده است:

```text
SELKIES_USE_CPU=true|locked
```

در نتیجه برنامه‌های سنگین ممکن است FPS واقعی را پایین بیاورند.

---

# 🔧 تغییر کیفیت برای اینترنت ضعیف

پیشنهاد پایه:

```text
Resolution: 1280×720
FPS: 60
CRF: 28
Bitrate: 2000 kbps
JPEG: 55
```

برای اینترنت ضعیف‌تر می‌توان تنظیمات را به شکل زیر تغییر داد:

```text
Resolution: 1280×720
FPS: 30
CRF: 30
Bitrate: 1200–1500 kbps
JPEG: 45–50
```

برای کیفیت بالاتر:

```text
Resolution: 1280×720
FPS: 60
CRF: 23–25
Bitrate: 2500–4000 kbps
JPEG: 65–75
```

این اعداد نمونه تنظیم هستند و نتیجه واقعی با توجه به شبکه و بار سیستم متفاوت خواهد بود.

---

# 📱 استفاده با موبایل

پروژه برای استفاده از طریق Browser مناسب است.

مراحل کلی:

```text
1. Workflow را Run کنید
2. صبر کنید Docker و Selkies بالا بیایند
3. Cloudflare URL ساخته شود
4. URL را از GitHub Actions Summary بردارید
5. URL را داخل مرورگر موبایل باز کنید
6. Username را وارد کنید:
   desktop
7. Password همان Secret:
   SELKIES_PASSWORD
```

برای کیبورد موبایل نیز چند لایه اصلاح در Container فعال شده است.

---

# 🔒 نکات امنیتی مهم

این پروژه را به‌عنوان یک سرویس Production دائمی در نظر نگیرید.

### رمز قوی استفاده کنید

مثال بد:

```text
123456
```

مثال بهتر:

```text
KaliDesktop_2026!Strong
```

### Secret را چاپ نکنید

از این کار خودداری کنید:

```bash
echo "$DESKTOP_PASSWORD"
```

### Session را باز رها نکنید

لینک Quick Tunnel عمومی است و تا زمانی که Job فعال باشد قابل استفاده است.

### فایل حساس داخل Desktop ذخیره نکنید

مانند:

```text
SSH private keys
API keys
Wallet files
Passwords
Personal documents
```

---

# ☁️ Cloudflare Quick Tunnel چیست؟

این پروژه از Tunnel دائمی با Domain شخصی استفاده نمی‌کند.

به‌جای آن:

```text
cloudflared tunnel --url http://127.0.0.1:3000
```

یک Quick Tunnel می‌سازد.

نتیجه یک دامنه موقت مانند:

```text
https://xxxxx.trycloudflare.com
```

است.

این روش برای Sessionهای موقت مناسب است، اما برای سرویس Production و دائمی بهتر است از معماری مناسب‌تر و احراز هویت قوی استفاده شود.

---

# 🐳 دستورات مفید Docker

نمایش Container:

```bash
docker ps
```

نمایش همه Containerها:

```bash
docker ps -a
```

دیدن Log:

```bash
docker logs selkies-desktop
```

دنبال‌کردن Log:

```bash
docker logs -f selkies-desktop
```

ورود به Container:

```bash
docker exec -it selkies-desktop bash
```

بررسی Image:

```bash
docker images
```

---

# 🧪 تست دستی Selkies

بعد از اجرای Container:

```bash
curl \
  --user "desktop:${SELKIES_PASSWORD}" \
  http://127.0.0.1:3000/
```

اگر HTML برگردد یعنی سرویس HTTP قابل دسترسی است.

---

# 🔍 تست Keymap داخل Container

```bash
setxkbmap -query
```

و:

```bash
xset q
```

و:

```bash
echo "$DISPLAY"
```

---

# 📝 شخصی‌سازی Username

مقدار پیش‌فرض:

```text
DESKTOP_USER=desktop
```

را می‌توان تغییر داد.

مثلاً:

```text
DESKTOP_USER=kaliuser
```

ولی باید مطمئن شوید Selkies / LinuxServer image با آن مقدار در نسخه مورد استفاده شما سازگار است.

---

# 🎨 تغییر عنوان Desktop

این متغیرها عنوان را کنترل می‌کنند:

```text
TITLE="Kali Web Desktop"
SELKIES_UI_TITLE="Kali Desktop"
```

مثلاً:

```text
TITLE="My Kali"
SELKIES_UI_TITLE="My Kali Web"
```

---

# 📋 خلاصه اجرای کامل

```text
GitHub Actions
      ↓
Validate Secret
      ↓
Create Dockerfile + customization
      ↓
docker build
      ↓
Run Kali + Selkies
      ↓
Local HTTP test
      ↓
Install cloudflared
      ↓
Create Quick Tunnel
      ↓
Extract trycloudflare.com URL
      ↓
Public HTTP test
      ↓
Publish URL in Actions Summary
      ↓
Keep container alive
```

---

# 📌 فایل Workflow

فایل اصلی را مثلاً با این نام ذخیره کنید:

```text
.github/workflows/kali-desktop.yml
```

بعد کد Workflow را داخل آن قرار دهید.

---

# ✅ Checklist قبل از اجرا

```text
[ ] Repository ساخته شده
[ ] Actions فعال است
[ ] فایل Workflow وجود دارد
[ ] Secret با نام SELKIES_PASSWORD ساخته شده
[ ] Secret خالی نیست
[ ] Workflow با workflow_dispatch قابل اجرا است
[ ] Docker روی ubuntu-latest در دسترس است
[ ] Runner به اینترنت دسترسی دارد
```

---

# 🧠 نتیجه

این پروژه یک Pipeline کامل برای ساخت یک **Kali Web Desktop موقت** است:

- Kali داخل Docker
- Selkies برای Desktop Streaming
- Cloudflare برای Public Access
- Firefox برای استفاده داخل Browser
- تنظیمات Low-Bandwidth
- 60 FPS هدف‌گذاری‌شده
- رزولوشن 1280×720
- Audio
- Clipboard
- Keyboard repair
- Wallpaper
- Health checks
- Keep-alive

لینک عمومی فقط تا زمانی در دسترس است که GitHub Actions Job فعال باشد.

---

## ⚠️ توجه نهایی

این README رفتار Workflow فعلی را مستند می‌کند. برخی قابلیت‌های Selkies و image پایه ممکن است با تغییر نسخه‌ها متفاوت شوند؛ بنابراین در صورت تغییر image یا نسخه Selkies، متغیرهای محیطی را دوباره بررسی کنید.
