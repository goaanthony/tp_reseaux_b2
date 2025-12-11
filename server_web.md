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
