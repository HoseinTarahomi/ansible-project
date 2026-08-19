# 🏗️ Infrastructure Architecture Diagram

این سند شامل دیاگرام و معماری تفصیلی زیرساخت پروداکشن (Target Node) و نحوه اتصال Control Node و کاربران است.

---

## 📐 دیاگرام کلی معماری (System Overview)

```mermaid
graph TD
    Users[👥 External Users / Traffic] -->|HTTPS 443 / HTTP 80| Firewall[🛡️ UFW Firewall]

    subgraph ToolsServer["🖥️ Target Server (Tools-Server)"]
        Firewall --> Traefik[🚦 Traefik v3 Reverse Proxy]

        subgraph DockerNetwork["🔒 Docker Isolated Network (proxy)"]
            Traefik -->|nexus.ht22.ir| Nexus[📦 Sonatype Nexus 3]
            Traefik -->|grafana.ht22.ir| Grafana[📊 Grafana Dashboard]
            
            Prometheus[📈 Prometheus] -->|Internal Scrape| NodeExporter[🖥️ Node Exporter]
            Promtail[📜 Promtail] -->|Push Logs| Loki[🗄️ Grafana Loki]
            Prometheus -->|Internal Alerting| Alertmanager[🔔 Alertmanager]
            Grafana -->|Query Metrics| Prometheus
            Grafana -->|Query Logs| Loki
        end

        Cron[⏰ Daily Cron Job 02:00] -->|Backup Volumes| Borg[💾 BorgBackup Script]
        Borg --> Archive[(🔒 Encrypted Repositories)]
    end

    subgraph ControlNode["💻 Control Node (ubuntu1)"]
        Ansible[⚙️ Ansible Playbooks] -->|SSH Key Executions| ToolsServer
    end
```

---

## 🔍 شرح جریان ترافیک و اجزا

1. **لایه ورودی (Ingress):** تنها درخواست‌های مربوط به دامنه‌های عمومی ( و ) از طریق پورت‌های 80/443 توسط Traefik هدایت می‌شوند.
2. **لایه سرویس‌های داخلی:** سایر سرویس‌ها (Prometheus، Alertmanager، Loki) دامنه‌ی عمومی نداشته و فقط به عنوان سرویس‌های داخلی در شبکه ایزوله داکر به یکدیگر و به Grafana پاسخ می‌دهند.
3. **لایه مدیریت و اتوماسیون:** سرور  از طریق SSH و کلید امنیتی، پیکربندی‌ها را روی سرور مقصد اعمال می‌کند.
