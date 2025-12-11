### Ce script permet de setup rapidement une vm debian minimal avec les tools de bases et permet de configurer automatiquement une carte reseaux avec une ip static

```bash
#!/bin/bash

# Verif root avec arret
if [ "$EUID" -ne 0 ]; then
  echo "ERREUR : Ce script doit être exécuté avec sudo (ou en root)."
  exit 1
fi

#echo "--- 1. installation des paquets, mise a jour ---"
#apt update
#apt install -y nano htop curl wget zip unzip openssh-server net-tools iproute2 dnsutils isc-dhcp-client

echo "--- 2. Configuration reseau ---"

INTERFACE="enp0s8"

echo "Configuration de l'interface $INTERFACE en 192.168.155.22..."

cp /etc/network/interfaces /etc/network/interfaces.bak

# Ajout de la configuration
cat << EOL_NET >> /etc/network/interfaces

# Configuration Host-Only
allow-hotplug $INTERFACE
iface $INTERFACE inet static
    address 192.168.155.22
    netmask 255.255.255.0
EOL_NET

echo "Redémarrage du service réseau..."

# On force l'interface a  monter
ip link set $INTERFACE up
systemctl restart networking

echo "--- Terminé ! ---"
ip a show $INTERFACE

# 1. On supprime toutes les adresses IP de la carte enp0s8
sudo ip addr flush dev enp0s8

# 2. On redm le service reseau pour qu'il relise ton fichier config
sudo systemctl restart networking

# 3. On verif
ip a show enp0s8

```