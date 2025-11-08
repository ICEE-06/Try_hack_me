## Infos
- ip : `10.10.13.134`
- os: linux

## Énumeration

Le scan nmap revele 2 ports ouverts:
![[nmap.png]]

On a un port http ouvert
![[web.png]]

On peut trouver une chose intérressante dans le seul poste éxistant:
![[post.png]]

C'est une incitation au **port knocking**
```
knock 10.10.128.104 1111 2222 3333 4444
```

Après le **port knocking**, on a trouvé un nouveau port ouvert: le port **ftp**.
![[knock.png]]

On peut ftp l'IP en utilisant l'identifiant `anonymous`





