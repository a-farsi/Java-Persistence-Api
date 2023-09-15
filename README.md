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

```java
@Entity
public class Musicien {
   // ...
}
```

La deuxième annotation obligatoire est @Id, que nous plaçons sur le champ de la clé primaire de notre classe. Ce champ de la clé primaire est nécessaire, car il indique à JPA quel champ représente la clé primaire, en l'occurrence le champ "id". En réalité, en toute rigueur, ces deux éléments suffisent à faire fonctionner notre classe en tant qu'entité JPA.
```java
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
Une autre annotation que nous pouvons aposer sur notre entité est @Table qui nous permet de renommer le nom de la table si nous le souhaitons car par default la table prend le nom de la classe.

```java
@Table(name="Musicien")
@Entity
public class Musicien {
    // attributes
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
    @GeneratedValue
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
    @Id
    @GeneratedValue
    private Long id;
    @Column(name="name", length=80)
    private String name;
    @Temporal
    private Date dateOfBirth;
    private MusicType musicType;
    // getters
    // setters
}
```

Ces champs colonnes, qui permettent notamment de préciser les noms des colonnes, nous permettent de nous adapter à une table Musicien qui existe déjà. Grâce à ces métadonnées, nous pouvons en fait adapter un nouveau modèle objet à un schéma de base de données legacy (qui existait déjà avant l'application que nous sommes en train de construire).

Enfin, il reste le cas du champ "MusicType", qui est particulier. Il s'agit d'une énumération (une classe Java particulière appelée "énumération" et dont nous fixons les instances lors de la déclaration de la classe). En l'occurrence, notre énumération ne peut avoir que quatre instances : JAZZ, CLASSICAL, ROCK et FOLK.

```java
public class MusicType {
    JAZZ, CLASSIC, ROCK, FOLK
}
```
Ce champ MusicType, qui ne peut prendre que ces quatre valeurs, peut être mappé de deux manières différentes en base de données :

* La première consiste à enregistrer le numéro d'ordre de l'instance telle qu'elle a été déclarée dans l'énumération MusicType, donc 0 pour JAZZ, 1 pour CLASSICAL, 2 pour ROCK et 3 pour FOLK.

* La deuxième manière est de le mapper sous forme de chaînes de caractères, c'est-à-dire d'écrire explicitement en base de données le nom de l'instance (JAZZ, CLASSICAL, …). C'est d'ailleurs la méthode que nous allons utiliser ici, pour deux raisons. Premièrement, c'est une question de lisibilité : il sera plus simple de comprendre quelles valeurs d'énumération ont été enregistrées pour les lignes de notre table lorsque nous examinerons uniquement le contenu de la base. De plus, pour des raisons d'évolution de notre modèle, si jamais quelqu'un intervertit CLASSICAL et JAZZ, éventuellement sans le savoir, cela modifiera également les numéros d'ordre des instances dans cette énumération et donc changera les valeurs enregistrées dans notre base.

Nous avons configuré notre classe en tant qu'entité JPA. Nous devons maintenant tester ce modèle, et pour cela, nous avons besoin d'intégrer une base de données dans notre environnement de développement. La solution locale que nous allons choisir est une base de données MySQL. Notre objectif est également de montrer que nous pouvons parfaitement utiliser JPA avec un serveur de base de données spécifique, en l'occurrence MySQL dans un environnement de développement, et le déployer en production avec un autre serveur de base de données, en l'occurrence Oracle.

# Configuration JPA

Comment indiquons-nous à JPA qu'elle doit se connecter à une base de données particulière plutôt qu'à une autre ? Comment lui précisons-nous que nos classes sont stockées dans tel module de notre application plutôt que dans tel autre ? Comment lui expliquons-nous comment elle doit, par exemple, journaliser ses requêtes SQL qu'elle envoie à la base de données, etc. ? Toutes ces questions trouvent une réponse unique dans un fichier de configuration appelé le "persistence.xml". Ce fichier regroupe toutes les informations techniques nécessaires à JPA car il rassemble des métadonnées qui ne peuvent pas être exprimées au niveau des différentes classes de nos entités JPA

```xml
<?xml version="1.0"  encoding="UTF-8"?>
<persistence  xmlns="http://java.sun.com/xml/ns/persistence"  
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
              version="1.0"  xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
              http://java.sun.com/xml/ns/persistence/persistence_1_0.xsd">
    <!-- transaction type : JTA ou RESOURCE_LOCAL -->
    <persistence-unit  name="Dataservice" transaction-type="RESOURCE_LOCAL">
         <provider>oracle.toplink.essentials.PersistenceProvider</provider>
         <class>fr.afa.model.Musicien</class>
    </persistence-unit>
    
    <properties>
          <!-- set of properties, some can be implementation specific -->
    </properties>
</persistence>
```
Ce fichier suit un standard, et ne comporte qu'un seul élément racine qui a pour nom <persistence>. Le premier élément fils de cet élément racine est 'persistence-unit' qui modélise une unité de persistance JPA.

Que signifie une unité de persistance techniquement ? C'est un ensemble de classes, de classes persistantes, d'entités JPA, et d'autres classes, qui sont rassemblées à l'intérieur de cette unité de persistance. une unité de persistance possède un nom, spécifié sous forme d'un attribut de l'élément 'persistence-unit', ici noté 'name = "Dataservice"'.

En général, une application JPA se limite à une seule unité de persistance, qui englobe l'ensemble des classes du modèle. Cependant, il est possible, d'un point de vue technique, d'en créer autant que nécessaire.

Il est important de ne pas confondre différentes unités de persistance qui peuvent coexister au sein d'une même application. Une unité de persistance pourrait regrouper un ensemble de classes utilisées en mode développement et test avec une autre unité de persistance contenant les mêmes classes, mais destinées cette fois à la production. Nous allons exploiter cette fonctionnalité de JPA qui nous permet d'utiliser un même jeu de classes dans deux unités de persistance distinctes. Pour ce faire, nous les déclarerons dans deux fichiers persistence.xml différents. L'un sera déployé dans notre environnement de test et de développement, tandis que l'autre sera déployé dans un environnement de production distinct.

Il serait important que nous prenions un moment pour expliquer certaines choses :

En réalité, nous n'aurons pas dans notre application deux unités de persistance différentes en fonctionnement simultané au sein de la même application déployée en PROD.

Le premier aspect à préciser concerne l'implémentation de la spécification JPA que nous pouvons utiliser. Plusieurs implémentations sont disponibles : Glassfish propose une implémentation standard appelée EclipseLink, WildFly offre la très connue et populaire implémentation Hibernate, disponible dans plusieurs langages, pas seulement en Java. Il existe également d'autres implémentations, telles qu'OpenJPA, soutenue par la Fondation Apache, etc...

Le deuxième aspect concerne le mode dans lequel nous allons opérer. En fait, il existe deux modes de fonctionnement pour JPA. Le premier mode est Java SE, bien que JPA soit une API Java EE, elle peut être extraite de Java EE et associée à des applications Java SE pour fonctionner dans un environnement SE. Il est évident que cet environnement est moins riche en fonctionnalités que l'environnement Java EE, notamment en ce qui concerne la gestion automatique des transactions en environnement Java SE. Cela est précisé dans le fichier "persistence.xml" par l'indication 'transaction-type="RESOURCE_LOCAL"', ce qui signifie que nous allons gérer manuellement les transactions au sein de notre application.

Enfin, le fichier persistence.xml nous permet de spécifier un jeu de propriétés. Ces propriétés se présentent sous forme de paires clé-valeur, et elles nous permettent de personnaliser notre module de mappage objet-relationnel.


```xml    
    <properties>
          <!-- set of properties, some can be implementation specific -->
    </properties>
```

Nous allons voir un certain nombre de ces propriétés. En réalité, il existe deux types de propriétés que nous pouvons configurer. Il y a tout d'abord les propriétés standards définies par la spécification, dont les clés sont précisément spécifiées dans cette dernière. C'est le cas des propriétés que nous présentons ci-dessous, et elles indiquent comment se connecter à la base de données requise par JPA pour fonctionner.

```xml   
    <properties>
        <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/jpatestdb" />
        <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />
        <property name="javax.persistence.jdbc.user" value="USER_NAME" />
        <property name="javax.persistence.jdbc.password" value="PASSWORD" />
    </properties>
```    

Dans ce contexte, nous utilisons une base de données MySQL, et nous devons spécifier quatre propriétés. Les trois premières sont les éléments nécessaires pour établir la connexion : l'URL de connexion, le nom d'utilisateur, et le mot de passe. En plus de ces éléments, nous devons préciser l'implémentation du pilote JDBC qui sera utilisée par le module JPA pour interagir avec la base de données.

Il est évident que ces quatre éléments, à savoir le pilote, l'URL de connexion, le nom d'utilisateur, et le mot de passe, sont spécifiques à chaque base de données. La syntaxe de ces éléments est généralement propre à chaque édition de base de données.

Cependant, nous avons également d'autres paramètres qui peuvent être spécifiques à l'implémentation de JPA que nous utilisons. Dans notre cas, nous utilisons EclipseLink, donc nous pouvons fournir des paramètres qui seront interprétés et pris en compte exclusivement par EclipseLink. Si nous avions utilisé Hibernate comme implémentation, nous aurions pu laisser ces paires clé-valeur ici, mais elles n'auraient pas été prises en compte, et nous aurions dû ajouter d'autres paramètres spécifiques à Hibernate.

```xml
    <persistence-unit  name="Dataservice" transaction-type="RESOURCE_LOCAL">
         <provider>org.eclipse.persistence.jpa.PersistenceProvider</provider>
         <class>fr.afa.model.Musicien</class>
    </persistence-unit>
    
    <properties>
          <!-- filres where DDL is written -->
          <property name="eclipselink.create-ddl-jdbc-file-name" value="sql/create.sql" />
          <property name="eclipselink.drop-ddl-jdbc-file-name" value="sql/drop.sql" />
          <!-- Possible values : sql -script | database | both -->
          <property name="eclipselink.ddl-generation.output-mode" value="both" />
          <property name="eclipselink.logging.timestamp" value="true" />
    </properties>
```    






