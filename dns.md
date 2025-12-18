# Mini-guide BIND9 (Linux Debian/Ubuntu)

## Prérequis
- Droits sudo
- Nom de domaine : remplace `TONNOM` (ex: `infra.lan`).
- IP du serveur DNS : remplace `<DNS_IP>` (ex: `192.168.155.21`).

## Installation
```bash
sudo apt update
sudo apt install bind9 bind9utils bind9-doc -y
```

## Déclarer la zone
Édite le fichier local :
```bash
sudo nano /etc/bind/named.conf.local
```

Ajoute ou adapte :
```conf
zone "TONNOM" {
    type master;
    file "/etc/bind/db.TONNOM";
};
```

## Créer le fichier de zone (liaison noms <-> IPs)
Duplique le modèle puis édite :
```bash
sudo cp /etc/bind/db.local /etc/bind/db.TONNOM
sudo nano /etc/bind/db.TONNOM
```

Exemple de contenu (adapte `TONNOM`, `<DNS_IP>`, hôtes et IPs) :
```conf
; Fichier de zone pour TONNOM
$TTL    604800
@       IN      SOA     ns.TONNOM. root.TONNOM. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
@       IN      NS      ns.TONNOM.      ; Nom du serveur DNS
@       IN      A       <DNS_IP>        ; IP du domaine racine

; Definitions des machines
ns      IN      A       <DNS_IP>        ; Serveur DNS
www     IN      A       192.0.2.10      ; Exemple Web
files   IN      A       192.0.2.11      ; Exemple stockage
admin   IN      A       192.0.2.12      ; Exemple admin
```

## Redémarrer et vérifier
```bash
sudo systemctl restart bind9
sudo systemctl status bind9
sudo named-checkconf
sudo named-checkzone TONNOM /etc/bind/db.TONNOM
```

## Nettoyage (optionnel)
```bash
sudo systemctl stop bind9
sudo apt-get purge bind9 bind9utils bind9-doc
sudo rm -rf /etc/bind /var/cache/bind /var/lib/bind
sudo apt-get autoremove --purge
```