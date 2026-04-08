# monitoring-stack

A self-hosted monitoring setup built with Prometheus, Grafana, Alertmanager, Node Exporter, and cAdvisor — all running via Docker Compose on a single machine.

Prometheus scrapes metrics from Node Exporter (host-level) and cAdvisor (container-level). Grafana visualizes everything with two pre-provisioned dashboards. Alertmanager handles alert routing and sends emails via Gmail SMTP when thresholds are breached.

---

## Requirements

- Docker Engine 24+
- Docker Compose 2+
- `envsubst` installed (`apt install gettext-base` on Ubuntu)

---

## Getting started

Clone the repo and go into the directory.

```bash
git clone https://github.com/Nour-Ben-Hadid/monitoring-stack.git
cd monitoring-stack
```

Copy the example env file and fill in your values.

```bash
cp .env.example .env
nano .env
```

The variables you need to set are your Grafana admin credentials and your Gmail SMTP details. For Gmail, use an App Password (not your real password) — generate one at myaccount.google.com under Security → App Passwords.

Before starting the stack, render the Alertmanager config with your real values:

```bash
set -a && source .env && set +a && envsubst < alertmanager/alertmanager.yml > alertmanager/alertmanager.rendered.yml
```

Then bring everything up:

```bash
docker compose up -d
docker compose ps
```

---

## Accessing the services

Grafana runs on port 3000, Prometheus on 9090, Alertmanager on 9093, cAdvisor on 8080, and Node Exporter metrics are at port 9100. All accessible at localhost.

Grafana login uses whatever credentials you set in `.env`.

---

## Dashboards

Two dashboards load automatically when Grafana starts — no manual import needed.

Node Exporter Full (Grafana ID 1860) gives you a complete view of host system metrics. Docker cAdvisor (Grafana ID 14282) shows per-container resource usage.

---

## Alerts

Alert rules are defined in `prometheus/alerts/` and cover high CPU usage (warning at 80%, critical at 95%), high memory usage, high disk usage, and containers or services that go down. When a rule fires, Prometheus sends the alert to Alertmanager which routes it to your email.

To test the email path manually:

```bash
curl -XPOST http://localhost:9093/api/v2/alerts \
  -H "Content-Type: application/json" \
  -d '[{"labels":{"alertname":"TestAlert","severity":"critical"},"annotations":{"description":"Manual test"},"startsAt":"2026-01-01T00:00:00Z","endsAt":"2026-01-01T23:59:00Z"}]'
```

Then check `docker compose logs alertmanager` to confirm delivery.

---

## Important notes

`alertmanager.rendered.yml` is gitignored because it contains your real credentials. Re-run the `envsubst` command above any time you change `.env`.

The Alertmanager image has no shell utilities, so config substitution happens on the host before the container starts — that's what the render step is for.


## CI

A GitHub Actions pipeline validates the Docker Compose configuration, starts the stack, checks service health, and tears everything down on each push.
