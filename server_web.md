### Ce script sert a faire un server web avec nginx

- install nginx
```bash
sudo apt install nginx -y
```

- Permet de faire un fichier html de base si tu veux pas faire echo tu use nano et tu code ton site web sinon guette la doc 
```bash
echo "<h1>Site du Serveur 2</h1>" > /var/www/html/index.html
```

- On start

```bash
systemctl enable nginx
systemctl start nginx
```

- certificat https
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/TONNOM.key -out /etc/ssl/certs/TONNOM.crt

-configurations nginx

sudo nano /etc/nginx/sites-available/default

server {
    listen 80;
    listen [::]:80;
    server_name TONNOM.lan www.TONNOM.lan;
    # Rediriger tout le trafic HTTP vers HTTPS (Optionnel mais recommandé)
    return 301 https://$host$request_uri;
}

server {
    # Port HTTPS
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name TONNOM.lan www.TONNOM.lan;

    # Chemin vers tes certificats créés à l'étape 1
    ssl_certificate /etc/ssl/certs/TONNOM.crt;
    ssl_certificate_key /etc/ssl/private/TONNOM.key;

    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

sudo nginx -t

sudo systemctl restart nginx




sudo systemctl stop nginx

sudo apt-get purge nginx nginx-common nginx-full nginx-core

sudo rm -rf /etc/nginx
sudo rm -rf /var/log/nginx

sudo rm -rf /var/www/html

sudo apt-get autoremove --purge