# Ansible Project Context

## Project Purpose

این پروژه برای پیکربندی و مدیریت سرورهای Ubuntu با Ansible و راه‌اندازی سرویس‌های مبتنی بر Docker استفاده می‌شود.

هدف این است که تمام تنظیمات زیرساخت به‌صورت خودکار، تکرارپذیر و Idempotent توسط Ansible اعمال شوند و وابستگی به تنظیمات دستی وجود نداشته باشد.

## Current Project Structure

ansible-project/
├── ansible.cfg
├── docs/
│   └── PROJECT_CONTEXT.md
├── inventory/
│   ├── group_vars/
│   │   └── all/
│   │       ├── main.yml
│   │       └── vault.yml
│   ├── hosts
│   └── host_vars/
├── playbooks/
│   └── site.yml
└── roles/
├── common/
├── docker/
├── nexus/
└── traefik/

## Secret Management

Status: Completed

Ansible Vault برای مدیریت اطلاعات حساس استفاده می‌شود.

Current encrypted secrets:

* Nexus admin password

Vault structure:

inventory/
└── group_vars/
└── all/
├── main.yml
└── vault.yml

Public variables:

inventory/group_vars/all/main.yml

Encrypted secrets:

inventory/group_vars/all/vault.yml

Variable reference:

nexus_admin_password: "{{ vault_nexus_admin_password }}"

Execution requires Vault password:

ansible-playbook 
-i inventory/hosts 
playbooks/site.yml 
--ask-vault-pass

Validation completed:

* Vault file encrypted successfully
* Public variables loaded successfully
* Nexus Role executed successfully
* Idempotency verified
* Second execution resulted in changed=0

## Docker Daemon Management

Status: Completed

Docker daemon configuration is managed using an Ansible Template.

Configuration file:

/etc/docker/daemon.json

Template:

roles/docker/templates/daemon.json.j2

Validation:

Before replacing the active Docker daemon configuration, Ansible validates the rendered JSON using:

python3 -m json.tool

Handler behavior:

* Docker is restarted only when daemon.json changes.
* Docker is not restarted during an unchanged Playbook execution.
* Docker configuration validation occurs before the active configuration is replaced.

Validation completed:

* Ansible Syntax Check passed
* Docker Role executed successfully
* Docker service status: active
* Nexus Registry Mirror remained active
* Nexus container remained running
* Traefik container remained running
* Idempotency verified
* Second execution resulted in changed=0

## Existing Roles

### common

تنظیمات پایه سیستم‌عامل:

* به‌روزرسانی APT cache
* نصب پکیج‌های عمومی
* تنظیم hostname
* تنظیم timezone
* ایجاد Directoryهای موردنیاز سرویس‌ها
* مدیریت کاربران
* تنظیم sudo
* تنظیم SSH
* اضافه کردن SSH Public Key
* بارگذاری Kernel Moduleها
* تنظیم Sysctl
* تنظیم System Limits
* غیرفعال کردن Swap

### docker

مدیریت Docker:

* نصب Docker Engine
* نصب Docker CLI
* نصب Containerd
* نصب Docker Buildx
* نصب Docker Compose Plugin
* نصب Docker Python SDK
* تنظیم Docker daemon با Template
* اعتبارسنجی JSON قبل از اعمال تنظیمات
* فعال‌سازی Docker service
* Restart Docker فقط در صورت تغییر Config
* ایجاد Docker Networkها
* اضافه کردن Admin User به گروه Docker
* تنظیم Nexus به‌عنوان Docker Registry Mirror

### traefik

راه‌اندازی Traefik با Docker Compose:

* ایجاد Directoryهای Traefik
* ایجاد Directory مربوط به Dynamic Configuration
* ایجاد فایل ACME
* ایجاد فایل تنظیمات Traefik
* ایجاد Docker Compose
* ایجاد Middlewareهای Dynamic
* ایجاد تنظیمات TLS
* Deploy کردن Traefik

### nexus

راه‌اندازی Nexus Repository Manager با Docker Compose:

* ایجاد Nexus Data Directory
* ایجاد Docker Compose
* Deploy کردن Nexus
* انتظار برای آماده شدن Nexus
* مدیریت Repositoryهای Docker از طریق Nexus REST API
* ایجاد یا به‌روزرسانی Repositoryها به‌صورت Idempotent

Repositoryهای مدیریت‌شده:

* docker-proxy
* docker-hosted
* docker-group

## Current Infrastructure

### Server

* سرور هدف: ubuntu2
* آدرس IP: 192.168.208.159
* سیستم‌عامل: Ubuntu 26.04 LTS
* Admin User: ht22
* Timezone: Asia/Tehran

### Docker

Docker نصب و فعال است.

Docker Registry Mirror:

http://192.168.208.159:8084

تنظیم مربوطه در Ansible:

docker_registry_mirror: "http://192.168.208.159:8084"

Docker daemon برای استفاده از Nexus Docker Group به‌عنوان Registry Mirror تنظیم شده است.

خروجی docker info تأیید کرده است:

Registry Mirrors:
http://192.168.208.159:8084/

### Docker Network

Network اصلی:

proxy

Traefik و Nexus روی این Network قرار دارند.

### Traefik

* Traefik در حال اجرا است.
* Traefik با Docker Compose مدیریت می‌شود.
* ارتباط Traefik با Nexus تست شده و سالم است.

### Nexus

Nexus با Ansible و Docker Compose مدیریت می‌شود.

Repositoryهای Docker:

* docker-proxy

  * Type: Proxy
  * Port: 8082

* docker-hosted

  * Type: Hosted
  * Port: 8083

* docker-group

  * Type: Group
  * Port: 8084

ترتیب اعضای Docker Group:

* docker-hosted
* docker-proxy

Docker Group به‌عنوان Registry Mirror استفاده می‌شود.

## Nexus Docker Registry Mirror Status

وضعیت: Completed

تاریخ تکمیل: 2026-07-31

موارد انجام‌شده:

* Nexus با موفقیت Deploy شده است.
* Repository مربوط به docker-proxy ایجاد و تنظیم شده است.
* Repository مربوط به docker-hosted ایجاد و تنظیم شده است.
* Repository مربوط به docker-group ایجاد و تنظیم شده است.
* پورت‌های Repositoryها اصلاح و با تنظیمات Ansible هماهنگ شده‌اند.
* Docker daemon برای استفاده از Nexus Group تنظیم شده است.
* تنظیمات Repositoryها به‌صورت Idempotent توسط Ansible مدیریت می‌شوند.
* اجرای Role مربوط به Nexus بدون خطا انجام شده است.

آخرین اجرای موفق:

PLAY RECAP

ubuntu2:
ok=11
changed=0
unreachable=0
failed=0
skipped=0
rescued=0
ignored=0

این خروجی نشان می‌دهد Role مربوط به Nexus بدون خطا اجرا شده و اجرای مجدد آن تغییری در زیرساخت ایجاد نکرده است.

## Nexus Registry Mirror Verification

دستور زیر اجرا شد:

docker pull nginx:latest

Image با موفقیت Pull شد.

پس از حذف Image محلی:

docker image rm nginx:latest

دوباره Pull انجام شد.

نتیجه:

* Docker درخواست را از طریق Nexus Docker Group ارسال کرده است.
* Nexus Image را Cache کرده است.
* Pull دوم از Cache Nexus انجام شده است.
* Nexus Docker Registry Mirror به‌درستی کار می‌کند.

## Removed From Scope

### MinIO

MinIO از Scope فعلی پروژه حذف شده است.

نباید موارد زیر برای MinIO وجود داشته باشند:

* Role
* Task
* Directory
* Variable
* Docker Compose
* مستندات مربوطه

متغیر minio_dir نباید در پروژه استفاده شود.

### Backup

قابلیت Backup فعلاً از Scope پروژه حذف شده است.

نباید موارد زیر برای Backup وجود داشته باشند:

* Role
* Task
* Directory
* Variable
* Job
* مستندات مربوطه

متغیر backup_dir نباید در پروژه استفاده شود.

## Project Rules

1. قبل از هر تغییر، فایل فعلی بررسی شود.
2. ساختار فعلی پروژه بدون دلیل تغییر نکند.
3. Role یا فایل جدید فقط در صورت نیاز واقعی ایجاد شود.
4. تنظیمات دستی باید به Ansible منتقل شوند.
5. Taskها باید Idempotent باشند.
6. اجرای مجدد Playbook نباید باعث تغییر غیرضروری شود.
7. Variableهای عمومی باید از inventory/group_vars/all/main.yml مدیریت شوند.
8. Secretها باید با Ansible Vault مدیریت شوند.
9. قبل از اجرای Playbook، Syntax Check انجام شود.
10. بعد از هر تغییر، تست و Verification انجام شود.
11. وضعیت جدید پروژه بعد از هر مرحله در همین فایل ثبت شود.
12. MinIO و Backup خارج از Scope فعلی پروژه هستند و نباید دوباره اضافه شوند.

## Standard Validation Commands

بررسی Syntax:

ansible-playbook 
-i inventory/hosts 
playbooks/site.yml 
--syntax-check 
--ask-vault-pass

اجرای کامل Playbook:

ansible-playbook 
-i inventory/hosts 
playbooks/site.yml 
--ask-vault-pass

اجرای فقط Role Docker:

ansible-playbook 
-i inventory/hosts 
playbooks/site.yml 
--tags docker 
--ask-vault-pass

اجرای فقط Role Nexus:

ansible-playbook 
-i inventory/hosts 
playbooks/site.yml 
--tags nexus 
--ask-vault-pass

بررسی وضعیت Docker:

systemctl is-active docker

بررسی Docker Registry Mirror:

docker info | grep -A 3 "Registry Mirrors"

بررسی Containerها:

docker ps

تست Pull از طریق Nexus:

docker pull nginx:latest

## Current Status

* Common Role: فعال و سالم
* Docker Role: فعال و سالم
* Docker daemon validation: فعال
* Docker restart handler: فعال
* Docker Role Idempotency: تأییدشده
* Traefik Role: فعال و در حال اجرا
* Nexus Role: فعال و سالم
* Nexus Docker Proxy: فعال
* Nexus Docker Hosted: فعال
* Nexus Docker Group: فعال
* Docker Registry Mirror: فعال و تست‌شده
* Nexus Image Cache: تست‌شده و سالم
* Ansible Vault: فعال
* Nexus Credentials: رمزنگاری‌شده
* Secret Management: Completed
* MinIO: خارج از Scope
* Backup: خارج از Scope

آخرین Task تکمیل‌شده:

Docker Daemon Validation and Conditional Restart Handler

وضعیت:

COMPLETED AND VERIFIED

