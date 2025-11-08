
- **IP**
```
10.10.156.137
```


## Ènumeration
Résultat du **nmap**
![[Lian_Yu/files/nmap.png]]

Exploration du port 80
![[Lian_Yu/files/web.png]]

Énumeration des répertoires cachés avec **gobuster**
```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://10.10.14.238/
```

Après ça on découvre un répertoire `/island`! Allons visiter ça
![[island.png]]

Là on découvre un code qui est : **vigilante** (peut être utilisé plus tard).

On continu le **gobuster**
```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://10.10.14.238/island/
```
Là on a trouver `/2100`

![[gobstIsland.png]]
Allons visiter ça
![[2100.png]]

Dans le code source de cette page, il y a un message qui dit qu'on mettre notre ticket là mais comment?
![[sourceCode.png]]

On va faire un autre **gobuster** avec le paramètre `-x` pour trouver un fichier selon son extension:
```
gobuster dir -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://10.10.14.238/island/2100 -x ticket
```

![[ticketfil.png]]

On a trouvé un token:
```
RTy8yhBQdscX
```

C'est quelque chose de crypter donc on va le décoder avec [cyberchef](https://gchq.github.io/CyberChef/)
![[cyberchef.png]]

on a eu un truc:
```
!#th3h00d
```

On va essayer de faire une connexion **ftp** avec `vigilante:!#th3h00d`
![[ftp.png]]

On va maintenant télécharger les fichiers avec **get**
Pour analyser le fichier **Leave_me_alone.png** de plus près, nous allons utiliser **hexeditor**
![[hexeditor.png]]

Comme on peut le voir à la première ligne, le header du fichier n'est pas encodé en PNG. On doit donc corriger ça:
![[correctionHeader.png]]

On maintenant:
![[Leave_me_alone.png]]

On peut maintenant extraire les infos cachées dans "aa.jpg" grâce au mdp trouver dans "Leave_me_alone.png"

![[inspec_aa.png]]

Maintenant on va unziper le fichier **ss.zip**
```
unzip ss.zip
```

On a deux sortie: `passwd.txt` et `shado`
Quand on fait `cat shado`, on a
```
M3tahuman
```

Maintenant on a le mdp ssh et on peut trouver l'username dans le **ftp** avec les commandes
```
cd ..
ls
```

![[ssh.png]]

Avec `cat user.txt` on a l'user flag.

## Privilège escalation
Voyons d'abord ce qu'on peut exécuter avec les droits sudo
![[droitsudo.png]]

Là on voit qu'on peut exécuter `/user/bin/pkexec` avec sudo

![[root.png]]








