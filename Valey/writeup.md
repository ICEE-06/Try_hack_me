- **IP**
```
10.10.54.49
```

On commence toujours par un scan nmap

![[Valey/file/nmap.png]]

Voilà sur quoi on tombe lorsqu'on explore le port 80

![[Valey/file/web.png]]

Il y a deux liens dans la page mais il n'y a rien de bien intéressant donc on va faire un fuzzing:

```
ffuf -w /usr/share/wordlists/dirb/common.txt -u http://10.10.54.49/FUZZ
```

![[fuzz.png]]

On a découvert un nouveau répertoire:

![[web_static_00.png]]

La page nous revele qu'il y a un autre répertoire `/dev1243224123123`

![[web_dev1243224123123.png]]

Là on tombe sur une page de connexion. Dans le code source de la page, il n'y a rien d'intéressant à part qu'il existe un fichier **dev.js**. On clique dessus et on trouve ça:

![[devJs.png]]

On est tombé sur des logins donc on va se connecter vec ces credentials:

![[web_dev_success_login.png]]
Cette note nous dit qu'il y a un port `ftp` ouvert que nous avons pas trouvé lors du premier scan. On va refaire un autre scan nmap et on va mettre comme option le scan de tout les ports tcp:

![[nmap_ftp_port.png]]

On va donc se connecter via ftp et on met les logins qu'on a trouvé:

```
ftp 10.10.54.49 37370
```

Une fois connecté, on trouve 3 fichiers **pcapng**. On va les télécharger avec `get` et analyser avec **wireshark**. Lorsqu'on exporte le fichier **siemHTTP2.pcapng** en fchier html, on trouve:

![[indexForSsh.png]]

On va se connecter via ssh avec ce qu'on a trouvé:

```
ssh valleyDev@10.10.54.49
```

![[ssh_ValeyDev.png]]

![[Valey/file/userFlag.png]]

Pour l'élévation de privilège on ne peut pas exécuter **sudo -l** même avec le mot de passe. Nous allons enumérer **crontabs** avec `cat /etc/crontab`

![[crontab.png]]





