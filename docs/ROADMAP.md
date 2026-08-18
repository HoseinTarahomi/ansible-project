# Project Roadmap & Infrastructure Status

این سند شامل وضعیت نهایی زیرساخت، فازهای تکمیل‌شده و برنامه‌های اختیاری آینده پروژه است.

---

## ✅ Phase 1: Core Infrastructure & Reverse Proxy (COMPLETED)
* **Traefik v3:** به عنوان Reverse Proxy با مسیریابی هوشمند و گواهی SSL خودکار (Let's Encrypt).
* **Sonatype Nexus 3:** راه‌اندازی مخزن مرکزی جهت مدیریت Artifactها و **Docker Private Registry**.
* **UFW Firewall:** ایمن‌سازی لایه شبکه و بستن تمام پورت‌های ورودی به جز پورت‌های ضروری (`22`, `80`, `443`).

---

## ✅ Phase 2: Observability & Centralized Logging (COMPLETED)
* **Prometheus & Node Exporter:** پایش وضعیت منابع سیستم‌عامل و کانتینرها.
* **Alertmanager:** مدیریت و ارسال هشدارهای سیستم.
* **Grafana:** پنل یکپارچه برای مدیریت داشبوردها و تحلیل داده‌ها.
* **Grafana Loki & Promtail:** جمع‌آوری، فیلتر و ذخیره‌سازی متمرکز لاگ‌های سیستم‌عامل و کانتینرهای داکر.

---

## ✅ Phase 3: Automated Backups & Disaster Recovery (COMPLETED)
* **BorgBackup:** پشتیبان‌گیری خودکار، فشرده و رمزنگاری‌شده از فایل‌های کانفیگ و ولوم‌های داکر.
* **Cron & Retention:** زمان‌بندی روزانه (ساعت ۰۲:۰۰ بامداد) با سیاست نگهداری (۷ روزه، ۴ هفتگی، ۶ ماهه).
* **Disaster Recovery Verified:** صحت‌سنجی کامل فرآیند بازگردانی (Restore Test) با موفقیت انجام شد.

---

## ⏸️ Future Considerations (ON HOLD / OPTIONAL)
* **CI/CD Pipeline:** راه‌اندازی GitLab/GitHub Runner جهت استقرار خودکار پروژه‌ها (در صورت نیاز در آینده).

---

## 📌 وضعیت فعلی سیستم (Production Baseline)
* **Status:** Fully Deployed, Secure, Monitored, and Backup-Protected.
