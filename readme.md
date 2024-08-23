# Getting Started with Metrics and Using Prometheus and Grafana

## Introduction

This document will cover the basics of how to:

* Setup/Install/Configure:
    * Prometheus
    * Grafana
    * Exporters
        * [node_exporter](https://github.com/prometheus/node_exporter)
        * [Graylog](https://go2docs.graylog.org/current/interacting_with_your_log_data/metrics.html?Highlight=metrics#ExportviaPrometheusMetricExporting)
        * [elasticsearch_exporter](https://github.com/prometheus-community/elasticsearch_exporter)
* Setup Grafana Dashboards:
    * Node Exporter
    * Elasticsearch Exporter
    * Graylog

## Basic Install of Prometheus and Grafana, using Docker

> [!NOTE]  
> The following assumes you already have Docker Engine installed. For documentation on installing Docker Engine, see [Install Docker Engine](https://docs.docker.com/engine/install/)

We will be using the docker compose file from [awesome-compose](https://github.com/docker/awesome-compose).

Because there is not a native way to download ONLY the `prometheus-grafana` folder, we will download the entire repo.

```sh
# Download repo
git clone https://github.com/docker/awesome-compose.git
# Execute docker compose file and start Prometheus and Grafana containers
sudo docker compose -f ./awesome-compose/prometheus-grafana/compose.yaml up -d
```

Optionally, you can copy the services and volumes from `compose.yaml` and place in your existing Docker compose file(s).

* Grafana is accessible via a web browser on port `:3000`
* Prometheus UI is accessible via a web browser on port `:9090`

## Install/Configure Exporters

Special applications, called [exporters](https://prometheus.io/docs/instrumenting/exporters/), run on your server, and server prometheus compatible output via HTTP. Different types of exporters bind to different ports.

### Graylog

Graylog has a built in prometheus exporter, but we need to enable it:

```yml
# Add the environment and port items to your existing docker compose file
services:
  graylog:
    environment:
      GRAYLOG_PROMETHEUS_EXPORTER_ENABLED: "true"
      GRAYLOG_PROMETHEUS_EXPORTER_BIND_ADDRESS: "0.0.0.0:9833"
    ports:
    - "9833:9833/tcp" # Graylog Prometheus exporter
```

For documentation, see https://go2docs.graylog.org/current/interacting_with_your_log_data/metrics.html

### OpenSearch

Add to docker compose:

> [!NOTE]  
> Be sure to update the `--es.uri` property to use the appropriate value for your environment.

```yml
services:
  # add to services
  elasticsearch_exporter:
      image: quay.io/prometheuscommunity/elasticsearch-exporter:latest
      command:
      - '--es.uri=http://opensearch:9200'
      restart: on-failure
      ports:
      - "9114:9114/tcp"
```

This exporter binds to `:9114`

### Host metrics

Using https://hub.docker.com/r/prom/node-exporter

```yml
services:
  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    network_mode: host
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'
```

This exporter binds to `:9100`

