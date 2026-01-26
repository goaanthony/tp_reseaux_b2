```
docker network create monitoring

docker run -d \
  --name prometheus \
  --network monitoring \
  -p 9090:9090 \
  prom/prometheus

docker run -d \
  --name grafana \
  --network monitoring \
  -p 3000:3000 \
  grafana/grafana
```

```
docker run -d \
  --name node-exporter \
  --network monitoring \
  -p 9100:9100 \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  quay.io/prometheus/node-exporter \
  --path.rootfs=/host
```

```
global:
  scrape_interval: 20s
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'nodes'
    static_configs:
      - targets: 
        - 'node-exporter:9100'
        # Ajouter d'autres serveurs ici
```

```
docker run -d \
  --name prometheus \
  --network monitoring \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus
```