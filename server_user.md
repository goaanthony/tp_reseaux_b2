### Ce script regroupe les commandes pour creer, supprimer et gestionner des users

- Ajouter un user 
```bash
sudo adduser  
```

- Permet de changer le hostname
```bash
sudo hostnamectl set-hostname -- Nom --
sudo nano /etc/hosts
```

- del un user
```bash
sudo deluser --remove-home 
```

- kill un process ave le max de puissace (9)
```bash
sudo kill -9 -- Nom du process ou id--
```

- Add un user dans un groupe
sudo usermod -aG groupName userName



 