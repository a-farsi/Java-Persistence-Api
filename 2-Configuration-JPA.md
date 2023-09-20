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

Qu'est-ce que ces paires clé-valeur nous indiquent ici, et qu'est-ce que ces propriétés nous disent ? Elles nous informent simplement de notre intention de rédiger les scripts de création et de suppression de nos schémas de table dans deux fichiers distincts : 'create.sql' et 'drop.sql', stockés dans un répertoire nommé 'sql'. Cette information se révèle précieuse lors du développement, car elle précise l'emplacement du répertoire 'sql', qui doit être situé dans notre projet Eclipse. Cependant, en production, ces deux propriétés doivent être supprimées.

Les deux dernières propriétés indiquent à EclipseLink comment traiter ces deux scripts de création et de suppression de notre base de données. Faut-il les diriger uniquement vers les fichiers, auquel cas nous serions responsables de leur exécution pour créer le schéma de la base de données ? Ou bien faut-il les acheminer exclusivement vers la base de données pour une création automatique du schéma au démarrage de notre application ? Ou encore, faut-il choisir les deux options : créer automatiquement le schéma en base de données tout en conservant une trace des scripts de création et de suppression dans des fichiers ?

En mode développement, le dernier mode se révèle pratique, car il combine la création automatique du schéma avec la sauvegarde des commandes SQL dans un fichier, que nous pouvons ensuite versionner, par exemple.

Enfin, ces dernières propriétés nous permettent de spécifier ce qu'EclipseLink journalisera. EclipseLink, tout comme OpenJPA et Hibernate, a la capacité d'enregistrer une grande quantité d'informations sur la façon dont il traite nos données. Cela va de la charge des classes à l'analyse de ces classes. Il peut journaliser chaque classe, chaque champ, ainsi que la manière dont il interprète l'ensemble des annotations que nous avons ajoutées à nos classes et à nos champs, jusqu'aux requêtes SQL envoyées à la base de données. Bien entendu, la configuration de cette journalisation doit être ajustée en fonction du mode, qu'il s'agisse du mode développement ou du mode production, principalement pour des raisons de performances. Le fait de journaliser l'intégralité d'une requête SQL a un impact notable sur l'exécution de notre application.