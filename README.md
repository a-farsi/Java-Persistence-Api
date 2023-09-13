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

L'intérêt de JPQL réside dans le fait que les requêtes rédigées sous forme d'objets sont indépendantes de la façon dont les objets sont stockés en base de données et de la structure de table choisie pour représenter notre modèle objet. Si la structure de la base de données évolue au fil de la vie de notre modèle objet, les requêtes peuvent être adaptées en fonction de ces changements de structure relationnelle. Les requêtes JPQL ne sont pas réécrites, mais leur traduction par l'implémentation sera modifiée pour s'adapter à la nouvelle structure relationnelle choisie.


Écrivons maintenant notre modèle objet. Nous commençons par écrire une classe que nous nommons "Musicien". Cette classe comporte 3 champs : le nom en chaîne de caractères, le deuxième est un champ de type date portant la date de naissance, et enfin un troisième de type MusicType.

```java
public class Musicien {
    private String name;
    private Date dateOfBirth;
    private MusicType musicType;
    // getters
    // setters
}
```

De quoi avons-nous besoin de façon indispensable pour écrire cet élément, qui est en fait un bean java très classique dans une BD ? Eh bien, ce que nous devons d'abord constater, c'est que l'ensemble des enregistrements dans une base doit posséder une clé primaire, et cette clé primaire doit impérativement figurer dans notre bean java. Par conséquent, le premier champ technique que nous devons ajouter de manière impérative à tout objet JPA est un champ de clé primaire.

Nous avons, a priori, plusieurs choix pour le type de cette clé primaire. Les choix principaux sont de type String, de type entier int ou de type long. Dans ce contexte, nous optons pour le type entier long en termes de performance, car il sera meilleur par rapport au type String et probablement comparable au type int. Une fois que le choix est fait, nous avons encore la possibilité de choisir entre un long de type primitif java et un long de classe wrapper, c'est-à-dire un objet. Il est toujours préférable de choisir l'objet plutôt que le type primitif. En effet, le modèle relationnel place "null" dans un champ de table lorsque la valeur d'un attribut d'un objet n'est pas spécifiée. Or, si notre long est de type primitif, sa valeur par défaut sera 0 et ne sera jamais "null" si on ne spécifie pas sa valeur. Par conséquent, le fait de choisir un long de type objet permettra de mieux correspondre au comportement en base de données, en particulier en ce qui concerne nos données.


Donc, notre classe Musicien, avec son champ technique, peut maintenant être sauvegardée en base de données et être gérée par notre module JPA. À présent, nous devons ajouter un certain nombre d'éléments techniques, qui sont en fait des annotations. Ces annotations seront utilisées par JPA comme des métadonnées appliquées à différents éléments de la classe Musicien.

La première de ces annotations est @Entity, qui doit être posée sur la classe Musicien. Cette annotation permet à JPA de reconnaître que cette classe est une entité JPA et de la gérer en tant que telle.

