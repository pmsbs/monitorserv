# AGENTS.md - PMSBS Monitor Server

This repository contains a Docker Compose-based monitoring stack with Prometheus and Grafana.

## Project Structure

```
.
├── docker-compose.yml      # Main stack definition
├── prometheus.yml          # Prometheus scrape configuration
├── grafana.ini             # Grafana configuration (SMTP)
├── blackbox.yml            # Blackbox exporter configuration
├── provisioning/
│   └── datasources/
│       └── datasource.yaml # Grafana auto-provisioned datasources
├── .env.example            # Environment variables template
└── README.md
```

## Build/Lint/Test Commands

### Running the Stack

```bash
# Start all services
docker compose up -d

# Stop all services
docker compose down

# View logs
docker compose logs -f [service-name]

# Restart a specific service
docker compose restart [service-name]
```

### Service Commands

```bash
# Prometheus (port 9091)
docker compose up -d prometheus

# Grafana (port 3000)
docker compose up -d grafana

# Node Exporter (port 9100)
docker compose up -d node-exporter

# Blackbox Exporter (port 9115)
docker compose up -d blackbox
```

### Configuration Validation

```bash
# Validate docker-compose syntax
docker compose config

# Check Prometheus config syntax
docker run --rm -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus promtool check config /etc/prometheus/prometheus.yml

# Check Blackbox exporter config
docker run --rm -v $(pwd)/blackbox.yml:/config.yml prom/blackbox-exporter --config.file=/config.yml --check.config
```

### Accessing Services

- **Grafana**: http://localhost:3000 (default admin/admin)
- **Prometheus**: http://localhost:9091
- **Node Exporter**: http://localhost:9100/metrics
- **Blackbox Exporter**: http://localhost:9115/probe?target=https://example.com

## Code Style Guidelines

### YAML Configuration

- **Indentation**: Use 2 spaces (no tabs)
- **Alignment**: Align values under keys at the same level
- **Quoting**: Quote strings that could be interpreted as booleans or numbers
- **Comments**: Use `#` for comments; document non-obvious configurations

```yaml
# Good
global:
  scrape_interval: 60s  # Collect metrics every minute

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

# Avoid
global:
scrape_interval: 60s
```

### Naming Conventions

- **Job names**: Use lowercase with underscores (snake_case)
- **Service names**: Use lowercase with hyphens (kebab-case)
- **Labels**: Use lowercase with underscores
- **File names**: Use lowercase with hyphens (e.g., `datasource.yaml`)

### Docker Compose Best Practices

- Always specify image versions or use `:latest` explicitly
- Use named volumes for persistent data
- Define networks explicitly
- Set resource constraints for production deployments
- Use `restart: unless-stopped` for long-running services

### Prometheus Configuration

- Always define `scrape_interval` in `global` (default: 60s)
- Use descriptive job names
- Add `relabel_configs` for custom labeling
- Include both `__address__` and `instance` labels
- Group related targets in the same job

```yaml
scrape_configs:
  - job_name: 'production'
    static_configs:
      - targets: ['server1:9100', 'server2:9100']
        labels:
          environment: production
```

### Grafana Provisioning

- Use the `apiVersion: 1` format for datasource provisioning
- Specify `access: proxy` for server-side data sources
- Set `isDefault: true` for the primary datasource
- Use `url:` with the Docker service name, not localhost

### Error Handling

- Always validate YAML syntax before deploying
- Test configuration changes in dry-run mode when possible
- Use descriptive error messages in relabel configs
- Verify network connectivity between services

### Security

- Never commit `.env` files with real credentials
- Use `${VAR}` syntax for environment variable interpolation
- Enable `no-new-privileges` security option for exporters
- Use read-only filesystem mounts when possible

## Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
cp .env.example .env
```

Required variables for SMTP (Grafana alerting):
- `GF_EMAIL_HOST` - SMTP server address
- `GF_EMAIL_USER` - SMTP username
- `GF_EMAIL_PASSWORD` - SMTP password
- `GF_EMAIL_FROM_ADDRESS` - From email address
- `GF_EMAIL_FROM_NAME` - From display name

## Adding New Targets

1. Edit `prometheus.yml` to add new targets to existing jobs or create new jobs
2. Validate the configuration: `docker compose config`
3. Reload Prometheus: `docker compose exec prometheus kill -HUP 1`
4. Verify targets in Prometheus UI at http://localhost:9091/targets

## Adding New Dashboards

1. Create dashboards in Grafana UI
2. Export JSON and save to `provisioning/dashboards/`
3. Add dashboard provider config to `provisioning/dashboards/dashboards.yaml`

## Troubleshooting

```bash
# Check service status
docker compose ps

# View service logs
docker compose logs [service-name]

# Restart with fresh volumes
docker compose down -v
docker compose up -d

# Access container shell
docker compose exec [service-name] sh
```
