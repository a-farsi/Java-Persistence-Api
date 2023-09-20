# Module JPA: Operations CRUD

Une fois que nous avons créé notre modèle objet, c'est-à-dire cet ensemble de classes avec leurs annotations, et une fois que nous avons configuré notre fichier persistance.xml et l'avons rangé dans le répertoire approprié, de manière à ce que le module JPA puisse le trouver et l'analyser, nous sommes prêts à mettre en œuvre notre modèle JPA.

JPA expose un ensemble de fonctionnalités, nous ne les examinerons pas toutes en détail. À présent, concentrons-nous sur les fonctionnalités CRUD (Create, Read, Update, Delete). La création d'un objet en base de données, c'est-à-dire l'exécution d'une requête 'insert' avec les valeurs d'un objet Java que nous avons préalablement créé, suit le modèle suivant : tout d'abord, nous créons notre bean Musicien qui doit être persisté en base de données, puis nous devons écrire un peu de code technique JPA. Le premier objet que nous devons créer est l'objet EntityManagerFactory, une interface propre à la spécification JPA.

Le patron de création de cet objet est la suivante : 
```java
EntityManagerFactory emf =Persistence.createEntityManagerFactory(...);
```
Où nous passons en paramètre une chaîne de caractères représentant le nom de l'unité de persistance que nous souhaitons charger. L'objet _EntityManagerFactory emf_ modélise en réalité à la fois l'ensemble de notre unité de persistance d'une part, et d'autre part, la connexion à la base de données. Par conséquent, sa création est assez coûteuse, car l'implémentation JPA doit à la fois établir une connexion à la base de données et analyser l'ensemble des classes. Il est donc recommandé de créer une unique instance de cet objet pour l'intégralité de notre application.

À partir de cette instance d'EntityManagerFactory, nous créons un deuxième objet EntityManager en invoquant la méthode createEntityManager(). 

```java
EntityManager em = emf.createEntityManager();
```

La création de cet EntityManager par rapport à la Factory est très légère, ce qui signifie que nous pouvons en créer autant que nécessaire. Une bonne pratique consiste à en créer un par transaction. À chaque nouvelle transaction, nous pouvons donc instancier un nouvel EntityManager. C'est via l'EntityManager que nous accédons aux opérations de persistance.

1. Pour persister un objet Java en base de données, c'est-à-dire un bean Java, il suffit de le passer en argument à la méthode persistence de l'EntityManager. Bien entendu, toutes les opérations en base de données doivent se dérouler dans le contexte d'une transaction, une règle qui s'applique autant dans le domaine SQL que dans le domaine JPA. Au travers de l'EntityManager, nous devons donc créer une transaction et la démarrer avec : 
 em.getTransaction().begin(); 
 Ensuite, après avoir effectué nos opérations de persistance, nous pouvons soit soumettre la transaction en appelant la méthode 'commit', soit l'annuler en utilisant la méthode 'rollback': 

```java 
   Musicien musicien =  new Musicien() ;
   em.getTransaction().begin() ;
   em.persist(musicien) ;
   em.getTransaction().commit() ;
   //em.getTransaction().rollback() ;
```

2. l'opération suivante concerne la lecture d'un objet de la base de données. Tout comme notre opération d'écriture en base, la lecture d'un objet passe également par l'objet EntityManager, via la méthode _find(...)_. Pour cette opération, nous devons spécifier la classe de l'entité que nous souhaitons lire ainsi que la valeur de la clé primaire.

```java 
Musicien musicien = entityManager.find(Musicien.class,  1L);
```

Lors de l'invocation de la méthode _find(...)_, JPA génère une requête SELECT et crée automatiquement l'objet Musicien pour nous. Dans ce cas, nous l'avons fait dans un contexte non transactionnel parce qu'une lecture n'est pas nécessairement transactionnelle, cela dépend de l'utilisation ultérieure que nous prévoyons pour l'objet. 
Nous pouvons également créer cet objet Musicien régulièrement et le lire dans un contexte transactionnel, en particulier si nous prévoyons de le modifier par la suite. Cependant, il est important de noter que l'attachement de cette instance de Musicien à une transaction entraîne un coût supplémentaire, ce qui signifie que nous devons payer un petit surcoût si nous exécutons cette requête _find(...)_ dans le contexte d'une transaction.