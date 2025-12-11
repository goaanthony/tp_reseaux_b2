# ÉTAPE 1 : INFRASTRUCTURE & RÉSEAU
## 1. Configuration IP
```bash
cat << EOF > /etc/network/interfaces
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

# WAN (NAT VirtualBox)
allow-hotplug enp0s3
iface enp0s3 inet dhcp

# LAN (Réseau Interne)
allow-hotplug enp0s8
iface enp0s8 inet static
    address 192.168.10.1/24
EOF
```
## 2. Activer le Routage (IP Forwarding)
```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
```
## 3. Configurer le NAT (Masquerade)
```bash
apt update && apt install -y iptables iptables-persistent
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
iptables-save > /etc/iptables/rules.v4
```

## 4. Appliquer
```bash
systemctl restart networking
```

# ÉTAPE 2 : SAUVEGARDE & RESTAURATION

## 1. Configuration IP et Passerelle
```bash
cat << EOF > /etc/network/interfaces
source /etc/network/interfaces.d/*

auto lo
iface lo inet loopback

# LAN
allow-hotplug enp0s3
iface enp0s3 inet static
    address 192.168.10.10/24
    gateway 192.168.10.1
EOF
```

## 2. DNS Temporaire (pour installer les paquets)
```bash
echo "nameserver 1.1.1.1" > /etc/resolv.conf
```

## 3. Appliquer
```bash
systemctl restart networking
```

### Création du script
```bash
cat << 'EOF' > /usr/local/bin/backup_system.sh
#!/bin/bash
BACKUP_DIR="/var/backups/infra_snapshots"
DATE=$(date +%Y-%m-%d_%H%M)
HOSTNAME=$(hostname)
ARCHIVE_NAME="$BACKUP_DIR/$HOSTNAME-$DATE.tar.gz"

mkdir -p $BACKUP_DIR

# Liste des paquets
dpkg --get-selections > $BACKUP_DIR/installed_packages.txt

# Archive (Config + Home + Web + DNS + DB)
tar -czf $ARCHIVE_NAME \
    /etc \
    /home \
    /root \
    /var/www \
    /var/lib/bind \
    /var/lib/mysql \
    $BACKUP_DIR/installed_packages.txt \
    --exclude=$BACKUP_DIR 2>/dev/null

# Suppression vieux backups (+7 jours)
find $BACKUP_DIR -type f -name "*.tar.gz" -mtime +7 -delete
echo "Backup OK: $ARCHIVE_NAME"
EOF
```
### Rendre exécutable
```bash
chmod +x /usr/local/bin/backup_system.sh
```

### Ajout au crontab (3h du matin)
```bash
(crontab -l 2>/dev/null; echo "0 3 * * * /usr/local/bin/backup_system.sh") | crontab -
```

### Restauration
```bash
#!/bin/bash

if [ -z "$1" ]; then
    echo "Précisez le fichier de backup."
    exit 1
fi

echo "Restauration en cours..."
tar -xzf $1 -C /
echo "Fichiers restaurés. Pensez à réinstaller les paquets si nécessaire :"
echo "dpkg --set-selections < /var/backups/infra_snapshots/installed_packages.txt && apt-get dselect-upgrade"
```

# ÉTAPE 3 : SERVICES (Sur VM-Services) PAS UTILISE

### Installation
```bash
apt install -y bind9 bind9utils bind9-doc
```

### Déclaration de la zone
```bash
cat << EOF > /etc/bind/named.conf.local
zone "projet.lan" {
    type master;
    file "/etc/bind/db.projet.lan";
};
EOF
```

### Fichier de zone
```bash
cat << EOF > /etc/bind/db.projet.lan
;
; BIND data file for projet.lan
;
\$TTL    604800
@       IN      SOA     projet.lan. root.projet.lan. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns.projet.lan.
@       IN      A       192.168.10.10
ns      IN      A       192.168.10.10
www     IN      CNAME   projet.lan.
EOF
```
### Redémarrage
```bash
systemctl restart bind9
```

### Configuration locale pour utiliser ce DNS
```bash
echo -e "domain projet.lan\nsearch projet.lan\nnameserver 192.168.10.10" > /etc/resolv.conf
```

### Installation
```bash
apt install -y apache2 openssl
```

### Certificat SSL auto-signé
```bash
mkdir -p /etc/apache2/ssl
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/apache2/ssl/apache.key \
    -out /etc/apache2/ssl/apache.crt \
    -subj "/C=FR/ST=France/L=Paris/O=Projet/CN=projet.lan"
```

### Configuration VirtualHost
```bash
cat << EOF > /etc/apache2/sites-available/projet-ssl.conf
<VirtualHost *:443>
    ServerName www.projet.lan
    DocumentRoot /var/www/html
    SSLEngine on
    SSLCertificateFile /etc/apache2/ssl/apache.crt
    SSLCertificateKeyFile /etc/apache2/ssl/apache.key
</VirtualHost>
EOF
```

### Activation
```bash
a2enmod ssl
a2ensite projet-ssl.conf
systemctl restart apache2
```
### Installation
```bash
apt install -y mariadb-server
```

### Création Base et Utilisateur
```bash
mariadb -u root <<EOF
CREATE DATABASE filrouge_db;
CREATE USER 'admin_projet'@'localhost' IDENTIFIED BY 'SuperMotDePasse!';
GRANT ALL PRIVILEGES ON filrouge_db.* TO 'admin_projet'@'localhost';
FLUSH PRIVILEGES;
EOF
```