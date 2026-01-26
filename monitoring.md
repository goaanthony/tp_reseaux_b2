# Monitoring avec LGTM Stack (Docker)

Mise en place d'une stack d'observabilité complète : **Prometheus** (Métriques), **Loki** (Logs), **Grafana** (Visualisation) et **Promtail** (Agent de logs).

## Prérequis
Installation de Docker sur la machine de monitoring (VM dédiée recommandée).
```bash
sudo apt update && sudo apt install docker.io docker-compose-plugin -y
# Ajout de l'utilisateur au groupe docker (éviter sudo)
sudo usermod -aG docker $USER
newgrp docker
```

## Installation & Configuration

### 1. Structure et Docker Compose
Création du dossier et du fichier de définition des services.

```bash
mkdir -p ~/monitoring && cd ~/monitoring

cat > docker-compose.yml <<'EOF'
version: '3.8'

services:
  # Visualisation
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    restart: unless-stopped

  # Métriques
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    restart: unless-stopped

  # Logs (Stockage)
  loki:
    image: grafana/loki:latest
    container_name: loki
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    restart: unless-stopped

  # Logs (Collecteur)
  promtail:
    image: grafana/promtail:latest
    container_name: promtail
    volumes:
      - /var/log:/var/log
    command: -config.file=/etc/promtail/config.yml
    restart: unless-stopped
EOF
```

### 2. Configuration Prometheus (Métriques)
Féfinir les cibles à surveiller.

```bash
cat > prometheus.yml <<'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
EOF
```

### 3. Configuration Promtail (Logs)
Définir quels logs envoyer à Loki.

```bash
cat > promtail-config.yml <<'EOF'
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
    - targets:
        - localhost
      labels:
        job: varlogs
        __path__: /var/log/*log
EOF
```

## Démarrage
```bash
docker compose up -d
docker compose ps
```

## Configuration Interface Web
Accès: `http://<IP_VM>:3000`
Login par défaut: `admin` / `admin` (changement requis à la première connexion).

### Connexion des sources de données
1. Aller dans **Connections** -> **Data Sources**
2. Ajouter **Prometheus**:
   - URL: `http://prometheus:9090`
   - Cliquer sur *Save & Test*
3. Ajouter **Loki**:
   - URL: `http://loki:3100`
   - Cliquer sur *Save & Test*
