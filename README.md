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
Une autre annotation que nous pouvons aposer sur notre entité est @Table qui nous permet de renommer le nom de la table si nous le souhaitons.

```java
@Table(name="Musicien")
@Entity
public class Musicien {
   // ...
}
```

La deuxième annotation obligatoire est @Id, que nous plaçons sur le champ de la clé primaire de notre classe. Ce champ de la clé primaire est nécessaire, car il indique à JPA quel champ représente la clé primaire, en l'occurrence le champ "id". En réalité, en toute rigueur, ces deux éléments suffisent à faire fonctionner notre classe en tant qu'entité JPA.
```java
@Table(name="Musicien")
@Entity
public class Musicien {
    @Id
    private Long id;
    private String name;
    private Date dateOfBirth;
    private MusicType musicType;
    // getters
    // setters
}
```
Cependant, les choses ne sont pas tout à fait terminées, car nous avons besoin d'affiner nos métadonnées, notamment les éléments optionnels. Le premier de ces éléments optionnels consiste à indiquer à JPA comment notre clé primaire est générée.

Si nous ne souhaitons pas déléguer à JPA la génération de la clé primaire et préférons que notre application génère cette valeur (ce sera à nous de fixer le champ "id" de la clé primaire correctement, et ce sera à notre code d'application de s'assurer que la valeur de la clé primaire insérée dans un objet n'est pas déjà utilisée en base de données). Ce problème est assez complexe, il semble plus simple de déléguer cette génération à l'implémentation JPA elle-même.

Pour cela nous aposons l'annotation @GeneratedValue sur l'identidiant de notre classe. Cette annotation dispose de 4 stratégies qui sont les suivantes : 

--AUTO : Le fournisseur JPA (Hibernate) détermine la meilleure méthode de génération de clé primaire à utiliser en fonction de la base de données spécifique. Il choisira alors la stratégie la plus appropriée parmi les autres stratégies possibles (IDENTITY, SEQUENCE, ou TABLE).

--IDENTITY: la clé primaire devrait être générée en utilisant une colonne d'identité dans la base de données.
Elle est souvent utilisée avec des bases de données telles que MySQL et SQL Server, où des colonnes à incrémentation automatique sont utilisées pour la génération de clés primaires.


--SEQUENCE - la clé primaire devrait être générée en utilisant une séquence de base de données. Les séquences sont généralement utilisées dans des bases de données telles qu'Oracle et PostgreSQL pour générer des valeurs numériques uniques. L'utilisation de cette stratégie nécessite l'utilisation de l'annotation @SequenceGenerator.

--TABLE - utilise une table dédiée qui stocke les clés des tables générées. L'utilisation de cette stratégie nécessite l'utilisation de l'annotation @TableGenerator
 

```java
@Table(name="Musicien")
@Entity
public class Musicien {
    @Id
    private Long id;
    private String name;
    private Date dateOfBirth;
    private MusicType musicType;
    // getters
    // setters
}
```

Le deuxième élément à prendre en compte est que certains champs de notre bean java seront automatiquement pris en charge par JPA pour être stockés dans des colonnes de la table Musicien, tandis que d'autres ne le seront pas.

Les champs qui sont mappés par défaut (c'est-à-dire pris en charge par JPA) sont les types primitifs java, les champs associés au type primitif et les champs de chaînes de caractères. En revanche, les champs d'un autre type doivent être annotés avec des annotations JPA pour pouvoir être enregistrés en base de données. C'est le cas du champ de type Date qui n'est pas mappé par défaut par JPA. Il doit être annoté avec l'annotation @Temporal, précédée du type de date à préserver dans ce champ : s'agit-il d'une date au format jj/mm/aaaa ou d'une date comprenant également l'heure, les minutes et les secondes, jj/mm/aaa/hh/min/seconde, par exemple. Ces trois formats sont des types SQL bien connus, donc avec l'annotation @Temporal, nous précisons le type de date que nous souhaitons enregistrer en base.


Nous pouvons également ajouter des métadonnées optionnelles sur les différents champs de notre classe. En l'occurrence, pour enregistrer les noms de nos Musiciens, nous souhaitons préciser les noms des colonnes dans lesquelles les champs 'nom' seront enregistrés. Ce type de métadonnée optionnelle peut être ajouté grâce à l'annotation @Column(…). Cette annotation peut également nous permettre de définir la longueur des champs 'nom'. Par défaut, JPA enregistre les chaînes de caractères dans des colonnes de type varchar avec une longueur de 256 caractères, ce qui peut être surdimensionné pour notre application.

```java
@Table(name="Musicien")
@Entity
public class Musicien {
    
    @Column(name="name", length=80)
    private String name;

    // getters
    // setters
}
```