# 🏗️ Infrastructure Architecture Diagram

این سند شامل دیاگرام و معماری تفصیلی زیرساخت پروداکشن (Target Node) و نحوه اتصال Control Node و کاربران است.

---

## 📐 دیاگرام کلی معماری (System Overview)

```mermaid
graph TD
    Users[👥 Users / Internet] -->|HTTPS 443 / HTTP 80| Firewall[🛡️ UFW Firewall]

    subgraph ToolsServer["🖥️ Target Server (Tools-Server)"]
        Firewall --> Traefik[🚦 Traefik v3 Reverse Proxy]

        subgraph DockerNetwork["🔒 Docker Isolated Network (proxy)"]
            Traefik -->|nexus.ht22.ir| Nexus[📦 Sonatype Nexus 3]
            Traefik -->|registry.ht22.ir| Registry[🐳 Docker Private Registry]
            Traefik -->|grafana.ht22.ir| Grafana[📊 Grafana]
            Traefik -->|prometheus.ht22.ir| Prometheus[📈 Prometheus]
            Traefik -->|alertmanager.ht22.ir| Alertmanager[🔔 Alertmanager]
            Traefik -->|traefik.ht22.ir| Dash[🎛️ Traefik Dashboard]

            Promtail[📜 Promtail] -->|Push Logs| Loki[🗄️ Grafana Loki]
            Prometheus -->|Scrape Metrics| NodeExporter[🖥️ Node Exporter]
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

1. **لایه ورودی (Ingress):** تمام درخواست‌های کاربران ابتدا به UFW Firewall رسیده و تنها پورت‌های 80 و 443 به سمت Traefik هدایت می‌شوند.
2. **لایه Reverse Proxy:** سرویس Traefik به صورت هوشمند و بر اساس SNI/Host Header، ترافیک را به کانتینر مربوطه در شبکه ایزوله proxy هدایت می‌کند.
3. **لایه مانیتورینگ و لاگینگ:**
   - Prometheus: اطلاعات متریك سیستم و کانتینرها را جمع‌آوری می‌کند.
   - Promtail & Loki: لاگ‌های متنی سیستم‌عامل و تمام کانتینرها را جمع‌آوری و برای نمایش به Grafana می‌فرستد.
4. **لایه مدیریت و اتوماسیون:** سرور ubuntu1 از طریق SSH و کلید امنیتی، تغییرات و رول‌های انسیبل را روی Target اجرا می‌کند.
