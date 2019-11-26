# Monitoring with Prometheus, Alertmanager and Grafana
---

## Description

 - This monitoring solution will deploy prometheus, alertmanager and grafana by using docker-compose.
 - All the data is stored in docker volumes for data persistence
 
## Installation

 - To install the monitoring solution run the command below in the repository root folder:
 
 ```
 docker-compose up -d
 ```
 
## Web Access
 
 - Prometheus: http://localhost:9090
 - Alertmanager: http://localhost:9093
 - Grafana: http://localhost:3000

## Configuration

 - Note: after each configuration change a restart is required. 
 
```
docker-compose up --force-recreate
```

## Manage Targets

 - Create target groups in the "prometheus.yml" file under "scrape_configs":
 
```
scrape_configs:

  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']

  - job_name: 'Targets 1'
    file_sd_configs:
      - files:
         - targets-1.yml

  - job_name: 'Targets 2'
    metrics_path: '/metrics'
    file_sd_configs:
      - files:
         - targets-2.yml
```

 - Each target group is being configured in a separated file (for example "target-1.yml"):
 
```
- targets: ['172.17.62.193:9182']
  labels:
    name: 'Server 1'

- targets: ['192.168.1.9:9182']
  labels:
    name: 'Server 2'

- targets: ['10.0.32.52:9091']
  labels:
    name: 'Server 3'
```

## Manage Rules

 - Add the alerts file to the "prometheus.yml" file under "rule_files":
 
```
rule_files:
  - 'alerts.yml'
```

 - Add the alerts specification to the "alerts.yaml" file:
 
```
groups:
  - name: critical-alerts
    rules:
    - alert: 'Instance Down'
      expr: up == 0
      for: 1m
      labels:
        severity: critical
      annotations:
        summary: "[ {{ $labels.name }} ] is down"
        description: "[ {{ $labels.name }} ] has been down for more than 5 minutes"
  - name: usage-alerts
    rules:
    - alert: 'RAM Usage'
      expr: 100 - ((node_memory_MemAvailable_bytes * 100) / node_memory_MemTotal_bytes) > 15
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: "Server {{ $labels.instance }} reach the RAM memory limit"
        description: "Server {{ $labels.instance }} has been using more than 85% of his memory RAM in the last 5 minutes"    
```

## Exporters:

 - node_exporter (linux)
   - https://github.com/prometheus/node_exporter/releases

 - wmi_exporter (windows)
   - https://github.com/martinlindhe/wmi_exporter/releases

