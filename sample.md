# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093
          - 'localhost:9097'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
    - "rules/alerts.yaml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    scheme: https
    tls_config:
      ca_file: 'public.crt'
      server_name: 'localhost' # For self signed certificates we need to add our domain/server name, If you have a valid certificate, you can skip this line.
      insecure_skip_verify: true
    basic_auth:
      username: "userprom"
      password: "grafana"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: 'application-server'
    static_configs:
      - targets: ['3.90.108.255:9100']
