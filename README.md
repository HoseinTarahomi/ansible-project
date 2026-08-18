<div align="center">

# Bare Metal Automated Infrastructure with Ansible

[![Ansible](https://img.shields.io/badge/Ansible-2.15+-EE0000?style=for-the-badge&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)](https://ubuntu.com/)
[![Docker](https://img.shields.io/badge/Docker-24.0+-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Traefik](https://img.shields.io/badge/Traefik-v3.0-24A1DE?style=for-the-badge&logo=traefik&logoColor=white)](https://traefik.io/)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-success?style=for-the-badge)]()

**یک زیرساخت عملیاتی، خودکار و امن مبتنی بر Ansible برای راه‌اندازی و مدیریت سرورهای Bare Metal Ubuntu.**

[معماری و جزئیات فنی](docs/PROJECT_CONTEXT.md) • [نقشه راه پروژه](docs/ROADMAP.md)

</div>

---

## 📌 نگاه کلی (Overview)

این پروژه تمام مراحل کانفیگ سیستم‌عامل، ایمن‌سازی شبکه، راه‌اندازی Reverse Proxy، مدیریت ریپازیتوری‌ها، سیستم مانیتورینگ متمرکز و پشتیبان‌گیری خودکار را به صورت ۱۰۰٪ کد (Infrastructure as Code) مدیریت می‌کند.

### 🏗️ معماری زیرساخت

* Control Node (ubuntu1): سرور مدیریت و اجرای Playbookهای انسیبل.
* Target Node (ubuntu2 / ubuntu3): سرور اصلی پروداکشن میزبان کانتینرها روی شبکه ایزوله proxy.

---

## 🛠️ سرویس‌ها و پشته فناوری

| سرویس | آدرس وب / دامنه | پورت داخلی | کاربرد |
| :--- | :--- | :--- | :--- |
| Traefik v3 | https://traefik.ht22.ir | 8080 | Reverse Proxy با دریافت خودکار SSL (Let's Encrypt) |
| Sonatype Nexus 3 | https://nexus.ht22.ir | 8081 | مدیریت Artifactها و کش پکیج‌ها |
| Docker Registry | https://registry.ht22.ir | 8084 | Private Docker Registry (Hosted + Proxy Group) |
| Grafana | https://grafana.ht22.ir | 3000 | داشبورد یکپارچه مانیتورینگ و مشاهده لاگ‌ها |
| Prometheus | https://prometheus.ht22.ir | 9090 | جمع‌آوری و ذخیره‌سازی Metricها |
| Alertmanager | https://alertmanager.ht22.ir | 9093 | مدیریت و ارسال هشدارهای سیستم |
| Loki & Promtail | سرویس داخلی | 3100 | مدیریت و متمرکزسازی لاگ‌های لینوکس و داکر |

---

## 📂 ساختار رول‌های Ansible

* common: تنظیمات لینوکس، Swap، پارامترهای کرنل و UFW Firewall
* traefik: راه‌اندازی Traefik v3 و شبکه سراسری proxy
* nexus: نصب Nexus 3 و پیکربندی خودکار Docker Repositories
* monitoring: پشته مانیتورینگ (Prometheus, Grafana, Alertmanager, Loki, Promtail)
* backup: ابزار BorgBackup، اسکریپت‌های خودکار و Cron Job روزانه

---

## 🚀 راهنمای سریع راه‌اندازی (Quick Start)

### ۱. پیش‌نیازهای اجرای پروژه روی سرور جدید

قبل از اجرای Playbook روی یک سرور جدید، موارد زیر باید مهیا باشند:

* **نصب Ansible:** نسخه 2.15 یا بالاتر روی Control Node.
* **پیش‌نیاز سرور مقصد (Target):** نصب بودن Python 3 روی سرور جدید.
* **دسترسی SSH بدون پسورد:** کپی کردن SSH Key از Control Node به سرور مقصد (ssh-copy-id).
* **دسترسی Sudo:** داشتن دسترسی Root یا کاربر با قابلیت sudo بدون پسورد در سرور مقصد.
* **تنظیمات Inventory:** به‌روزرسانی IP سرور مقصد در فایل inventory/hosts.

تست اتصال SSH و Ansible پیش از اجرا:
ansible all -i inventory/hosts -m ping

### ۲. اجرا و استقرار زیرساخت

کلون کردن پروژه:
git clone https://github.com/HoseinTarahomi/ansible-project.git
cd ansible-project

اجرای کامل Playbook:
ansible-playbook -i inventory/hosts playbooks/site.yml --ask-vault-pass

### ۳. اجرای رول‌های خاص (Tag-based Execution)

اجرای فقط رول مانیتورینگ:
ansible-playbook -i inventory/hosts playbooks/site.yml --tags "monitoring" --ask-vault-pass

اجرای رول فایروال و امنیت:
ansible-playbook -i inventory/hosts playbooks/site.yml --tags "common" --ask-vault-pass

---

## ⚙️ امنیت و پشتیبان‌گیری

* UFW Firewall: تمام پورت‌های ورودی مسدود شده و تنها پورت‌های 22 (SSH)، 80 (HTTP) و 443 (HTTPS) مجاز هستند.
* BorgBackup: فرآیند پشتیبان‌گیری خودکار هر روز ساعت ۰۲:۰۰ بامداد به صورت رمزنگاری‌شده اجرا شده و دارای سیاست نگهداری (Retention) ۷ روزه، ۴ هفتگی و ۶ ماهه است.

---

## 📚 مستندات تکمیلی

برای اطلاعات دقیق‌تر درباره متغیرها، معماری شبکه و نقشه راه، مستندات زیر را مطالعه کنید:
* PROJECT_CONTEXT.md (در دایرکتوری docs): شرح تفصیلی معماری و دستورات عیب‌یابی.
* ROADMAP.md (در دایرکتوری docs): وضعیت فازهای طی‌شده و برنامه‌های آینده.
