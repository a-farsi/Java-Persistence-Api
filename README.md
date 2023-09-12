# Java-Persistence-Api
Le module JPA :
JPA signifie Java Persistence API. Il s’agit d’une techno très mature, issue des travaux débutés au milieu des années 90, conduisit par la boite Toplink qui commercialisaient un produit de mapping Objet Relationnel programmé en C++.
JPA est issue de ces travaux et l’implémentation de la société Toplink. C’est une technologie mature et sophistiquée utilisée dans des projets d’entreprise. Il s’agit d’automatiser au maximum les opérations CRUD (Create, Read, Update and Delete) avec la base de données (BD). 

Quand on manipule des BD complexes, on peut se retrouver avec une quantité astronomique de requêtes SQL à écrire pour manipuler notre modèle objet et implémenter les différents opérations CRUD vers la BD. 
JPA apporte cette solution de l’automatisation, et la génération de l’ensemble de ces requêtes SQL.

On peut également, au travers d'un modèle JPA, passer nos propres requêtes SQL, et ce point est
extrêmement important si le modèle objet que l'on gère est un modèle objet qui existe déjà depuis
longtemps et pour lequel de nombreuses requêtes SQL ont été écrites. Ces requêtes SQL peuvent
parfaitement être récupérées, réutilisées et incorporées dans un modèle JPA afin d'être toujours
exécutées.
Il existe un langage alternatif à SQL, qui est JPQL, permettant d'écrire des requêtes sur le modèle objet,
qui seront traduites par l'implémentation JPA en langage SQL puis envoyées à la BD.