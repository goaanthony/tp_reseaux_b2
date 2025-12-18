# Sauvegarde avec Syncthing (Docker)

Synchronisation automatique entre Server2 (source) et Server3 (backup).

## Prérequis
```bash
sudo apt update && sudo apt install docker.io docker-compose -y
```

## Server2 (Source)
```bash
mkdir -p ~/syncthing-backup/data && cd ~/syncthing-backup
cat > docker-compose.yml <<'EOF'
version: "3"
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing_source
    hostname: server2-source
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Paris
    volumes:
      - ./config:/config
      - ./data:/data1
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: unless-stopped
EOF
sudo docker-compose up -d
sudo chown -R 1000:1000 data
sudo docker-compose restart
```

## Server3 (Backup)
```bash
mkdir -p ~/backup_system/syncthing/data_backup && cd ~/backup_system/syncthing
cat > docker-compose.yml <<'EOF'
version: "3"
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing_backup
    hostname: server3-backup
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Paris
    volumes:
      - ./config:/config
      - ./data_backup:/data1
    ports:
      - 8384:8384
      - 22000:22000/tcp
      - 22000:22000/udp
      - 21027:21027/udp
    restart: unless-stopped
EOF
sudo docker-compose up -d
sudo chown -R 1000:1000 data_backup
sudo docker-compose restart
```

## Configuration (Interface Web)
Accès: `http://<IP_SERVER>:8384`

### Appairage
1. Server2: *Actions > Afficher l'ID* → copier
2. Server3: *Ajouter un appareil* → coller l'ID → nommer "Server2-Source"
3. Server2: Accepter la connexion → nommer "Server3-Backup"

### Dossier partagé
**Server2:**
- *Ajouter un dossier* → Nom: `Projet` → Chemin: `/data1/projet`
- Onglet *Partage*: cocher "Server3-Backup"
- Onglet *Avancé*: Type → **Envoi uniquement**

**Server3:**
- Accepter le dossier "Projet"
- Onglet *Avancé*: Type → **Réception uniquement**
- Onglet *Versionnage*: **Versionnage échelonné**

## Test
```bash
# Server2
cd ~/syncthing-backup/data/projet
echo "Test réussi" > validation.txt

# Server3
ls -l ~/backup_system/syncthing/data_backup/projet/
# validation.txt doit apparaître
```