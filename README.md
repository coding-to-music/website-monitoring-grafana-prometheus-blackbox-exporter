# website-monitoring-grafana-prometheus-blackbox-exporter

# 🚀 Monitor your websites availability, http status code (current and history), certificate, redirects and more with Grafana and Prometheus blackbox exporter. 🚀

https://github.com/coding-to-music/website-monitoring-grafana-prometheus-blackbox-exporter

From / By https://github.com/mbelloiseau/website-monitoring

https://github.com/mbelloiseau/website-monitoring

## Environment variables:

```java

```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/website-monitoring-grafana-prometheus-blackbox-exporter.git
git push -u origin main
```

## Results

Blackbox Exporter http://localhost:9115 (all the websites being probed)

Grafana http://localhost:3000

Prometheus http://localhost:9090 (change the port to whatever is assigned rather than 9090)

# website-monitoring

![web-1](screenshots/website-monitoring_1.png)

Monitore your websites availability, http status code (current and history), certificate, redirects and more with

- [Prometheus](https://github.com/prometheus/prometheus)
- [Prometheus blackbox exporter](https://github.com/prometheus/blackbox_exporter)
- [Grafana](https://github.com/grafana/grafana)

## Dependencies

- [docker](https://docs.docker.com/install/)
- [docker-composer](https://docs.docker.com/compose/install/)

## Usage

- `git clone git@github.com:mbelloiseau/website-monitoring.git && cd website-monitoring`
- Edit `config/prometheus/targets.yml` (see targets.yml.example) or use `./gen_target.sh website-1.tld website-2.tld ...`
- Create and start containers `docker-compose up -d`
- [Visualize dashboards](http://localhost:3000/)

If you already have Prometheus and Prometheus blackbox exporter up and running just import the dashboards ([website-monitoring](dashboards/website-monitoring.json) or [overview](dashboards/overview.json)) and use the right [datasource](screenshots/import.png) and [jobs](screenshots/import.png) (http_job and icmp_job)

## Dashboards

### Website monitoring

- HTTP status code
- HTTP redirects
- HTTP version
- TLS version
- Certificate validity
- ICMP
- DNS lookup time
- Availability over the last 24 hours, 3 days and 7 days
- Probe duration and status code history

![web-2](screenshots/website-monitoring_2.png)
![web-3](screenshots/website-monitoring_3.png)

### Overview

- Total number of targets
- Percentage of HTTP 200 status code
- Percentage of targets using SSL
- Global invalid status code history

![overview](screenshots/overview_1.png)

## Tips and tricks

### PromQL

Some useful PromQL queries

- Number of days till certificate expiration
  - `(probe_ssl_earliest_cert_expiry{instance=~"$target",job="$http_job"} - time()) / (60*60*24)`
- Display bad HTTP status code
  - `probe_http_status_code{job="$http_job",instance=~"$target"} != 200`
- Count the number of each status code
  - `count_values("code", probe_http_status_code)`
- Percentage of HTTP 200
  - `((count(count by (instance) (probe_http_status_code == 200))) / (count(count by (instance) (probe_http_status_code)))) * 100`

### Misc

- Request blackbox exporter
  - `curl -s "localhost:9115/probe?module=http_2xx&target=target.tld"`

use example.com

```
curl -s "localhost:9115/probe?module=http_2xx&target=example.com"
```

Output

```
# HELP probe_dns_lookup_time_seconds Returns the time taken for probe dns lookup in seconds
# TYPE probe_dns_lookup_time_seconds gauge
probe_dns_lookup_time_seconds 0.002240472
# HELP probe_duration_seconds Returns how long the probe took to complete in seconds
# TYPE probe_duration_seconds gauge
probe_duration_seconds 0.006341017
# HELP probe_failed_due_to_regex Indicates if probe failed due to regex
# TYPE probe_failed_due_to_regex gauge
probe_failed_due_to_regex 0
# HELP probe_http_content_length Length of http content response
# TYPE probe_http_content_length gauge
probe_http_content_length 1256
# HELP probe_http_duration_seconds Duration of http request by phase, summed over all redirects
# TYPE probe_http_duration_seconds gauge
probe_http_duration_seconds{phase="connect"} 0.001819131
probe_http_duration_seconds{phase="processing"} 0.001684495
probe_http_duration_seconds{phase="resolve"} 0.002240472
probe_http_duration_seconds{phase="tls"} 0
probe_http_duration_seconds{phase="transfer"} 9.9657e-05
# HELP probe_http_last_modified_timestamp_seconds Returns the Last-Modified HTTP response header in unixtime
# TYPE probe_http_last_modified_timestamp_seconds gauge
probe_http_last_modified_timestamp_seconds 1.571296706e+09
# HELP probe_http_redirects The number of redirects
# TYPE probe_http_redirects gauge
probe_http_redirects 0
# HELP probe_http_ssl Indicates if SSL was used for the final redirect
# TYPE probe_http_ssl gauge
probe_http_ssl 0
# HELP probe_http_status_code Response HTTP status code
# TYPE probe_http_status_code gauge
probe_http_status_code 200
# HELP probe_http_uncompressed_body_length Length of uncompressed response body
# TYPE probe_http_uncompressed_body_length gauge
probe_http_uncompressed_body_length 1256
# HELP probe_http_version Returns the version of HTTP of the probe response
# TYPE probe_http_version gauge
probe_http_version 1.1
# HELP probe_ip_addr_hash Specifies the hash of IP address. It's useful to detect if the IP address changes.
# TYPE probe_ip_addr_hash gauge
probe_ip_addr_hash 3.947174862e+09
# HELP probe_ip_protocol Specifies whether probe ip protocol is IP4 or IP6
# TYPE probe_ip_protocol gauge
probe_ip_protocol 4
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
```

## To monitor basic auth logging into a website, see issue:

https://github.com/mbelloiseau/website-monitoring/issues/6

Question: have you ever managed to monitor/validate username and password of a page/api via blackbox-exporter?

Response: Yes, I had to monitore an API with blackbox and prometheus

### to monitore an API with blackbox and prometheus

prometheus.yml

```
  - job_name: "web-monitoring-http_restricted"
    metrics_path: /probe
    params:
      module: [http_2xx_restricted]
    file_sd_configs:
      - files:
        - /etc/prometheus/targets/http_restricted.yml
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: localhost:9115
```

blackbox.yml

```
  http_2xx_restricted:
    prober: http
    timeout: 8s
    http:
      valid_http_versions: ["HTTP/1.1", "HTTP/2.0"]
      valid_status_codes: []
      method: GET
      preferred_ip_protocol: "ip4"
      ip_protocol_fallback: false
      basic_auth:
        username: "user"
        password: "password"
```

http_restricted.yml is just a classic target file

But I'm not sure how to manage the configuration if you've more than one host and credentials for each host
