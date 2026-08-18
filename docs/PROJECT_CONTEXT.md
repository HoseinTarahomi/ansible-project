# Bare Metal Infrastructure Context & Overview

این سند شامل جزئیات کامل معماری، کانفیگ سرویس‌ها، نحوه مدیریت زیرساخت با Ansible و اطلاعات متغیرهای پروژه است.

---

## 🏗️ معماری کلی زیرساخت (Architecture Overview)

این پروژه یک زیرساخت پروداکشن مبتنی بر Bare Metal Ubuntu است که به صورت ۱۰۰٪ خودکار توسط Ansible مدیریت می‌شود. تمام سرویس‌ها در قالب کانتینرهای Docker روی یک شبکه ایزوله (proxy) اجرا شده و ترافیک آن‌ها از طریق Traefik هدایت می‌شود.

* Control Node: ubuntu1 (192.168.208.158) - سرور مدیریت انسیبل
* Target Node: ubuntu2 (192.168.208.159) - سرور اصلی اجراکننده سرویس‌ها

---

## 🛠️ جدول کامل سرویس‌ها و دامنه‌ها

| سرویس | آدرس وب / دامنه | پورت داخلی | شبکه داکر | توضیحات |
| :--- | :--- | :--- | :--- | :--- |
| Traefik v3 | [https://traefik.ht22.ir](https://traefik.ht22.ir) | 8080 | proxy | Reverse Proxy + SSL Let's Encrypt |
| Sonatype Nexus 3 | [https://nexus.ht22.ir](https://nexus.ht22.ir) | 8081 | proxy | مدیریت Artifactها و Pypi/Npm/Maven |
| Docker Registry | [https://registry.ht22.ir](https://registry.ht22.ir) | 8084 | proxy | Private Docker Registry (Hosted + Proxy) |
| Grafana | [https://grafana.ht22.ir](https://grafana.ht22.ir) | 3000 | proxy | داشبورد متمرکز مانیتورینگ و لاگ‌ها |
| Prometheus | [https://prometheus.ht22.ir](https://prometheus.ht22.ir) | 9090 | proxy | جمع‌آوری Metricهای سیستم و کانتینرها |
| Alertmanager | [https://alertmanager.ht22.ir](https://alertmanager.ht22.ir) | 9093 | proxy | ارسال هشدارهای مانیتورینگ |
| Grafana Loki | داخلی | 3100 | proxy | پایگاه‌داده متمرکز ذخیره لاگ‌ها |
| Promtail | داخلی | Agent | Host | جمع‌آوری لاگ‌های سیستم‌عامل و کانتینرها |

---

## 📂 شرح تفصیلی رول‌های Ansible

### 1. common (پیکربندی سیستم‌عامل و امنیت)
* تنظیم Swap space (در صورت عدم وجود).
* تنظیم Kernel parameters (br_netfilter, overlay, sysctl).
* پیکربندی فایروال UFW: مسدودسازی تمام پورت‌های ورودی و باز گذاشتن اختصاصی پورت‌های 22 (SSH)، 80 (HTTP) و 443 (HTTPS).
* مدیریت کاربران، SSH Keyها و دسترسی‌های Sudo.

### 2. traefik (مسیریابی و SSL)
* مدیریت ترافیک ورودی وب.
* دریافت و تمدید خودکار گواهی‌های SSL/TLS با Let's Encrypt.
* مدیریت شبکه سراسری proxy برای کانتینرها.

### 3. nexus (مدیریت ریپازیتوری‌ها)
* نصب Nexus 3 به همراه پیکربندی اتوماتیک ریپازیتوری‌های داکر:
  - Docker Proxy: کش کردن ایمیج‌های داکر هاب.
  - Docker Hosted: ذخیره ایمیج‌های اختصاصی پروژه.
  - Docker Group: ارائه یک endpoint یکپارچه روی پورت 8084.

### 4. monitoring (مشاهده‌پذیری و لاگینگ)
* Prometheus & Node Exporter: پایش منابع سخت‌افزاری و کانتینرها.
* Alertmanager: مدیریت هشدارهای مانیتورینگ.
* Grafana: نمایش بصری متریک‌ها.
* Loki & Promtail: جمع‌آوری زنده لاگ‌های فایل‌های /var/log/* و کانتینرهای /var/lib/docker/containers/*.

### 5. backup (پشتیبان‌گیری و بازگردانی)
* نصب و تنظیم ابزار BorgBackup.
* اسکریپت خودکار /usr/local/bin/borg-backup.sh جهت پشتیبان‌گیری انکریپت‌شده از /opt/monitoring، /etc/traefik و /var/lib/docker/volumes.
* زمان‌بندی Cron در ساعت ۰۲:۰۰ بامداد به همراه سیاست Retention (۷ روزه، ۴ هفتگی، ۶ ماهه).

---

## 🚀 دستورات کلیدی مدیریت و عیب‌یابی

* اجرای کامل Ansible Playbook:
  ansible-playbook -i inventory/hosts playbooks/site.yml --ask-vault-pass

* اجرای یک رول خاص (مثلاً مانیتورینگ یا بک‌آپ):
  ansible-playbook -i inventory/hosts playbooks/site.yml --tags "monitoring" --ask-vault-pass

* اجرای دستی بک‌آپ Borg (روی ubuntu2):
  sudo /usr/local/bin/borg-backup.sh

* مشاهده لیست بک‌آپ‌های ذخیره‌شده Borg:
  sudo BORG_PASSPHRASE='BorgSecurePass123!' borg list /var/backups/borg

* بررسی وضعیت فایروال UFW:
  sudo ufw status verbose
