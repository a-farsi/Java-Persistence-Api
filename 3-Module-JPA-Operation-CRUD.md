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

3. L'opération _update_ consiste en la mise à jour d'une entité. Nous avons ici deux façons de réaliser cette mise à jour, dépendant du contexte dans lequel nous travaillons. 

3.a. La première méthode, la plus simple, consiste à récupérer notre entité par son nom de classe et sa clé primaire, que ce soit dans un contexte transactionnel ou non, puis à la modifier dans un contexte transactionnel. La modification de cette entité déclenchera automatiquement une mise à jour vers la base de données par le biais de JPA lors de l'appel de la méthode commit de notre EntityManager, plus précisément lors de la gestion de la transaction par cet EntityManager. À ce stade, on peut avoir l'impression qu'une forme de magie JPA opère, car une simple invocation de la méthode setAge avec une valeur d'âge différente générera une mise à jour dans la base de données. En réalité, notre objet Musicien est une entité JPA, liée à un EntityManager, et dès qu'il est associé à un EntityManager, toute modification de cet objet entraînera automatiquement une mise à jour de la base de données. C'est pourquoi on parle de mapping Objet-Relationnel. L'objectif du mapping ORM est de refléter en base de données automatiquement les modifications apportées à notre modèle objet. Il convient également de noter que l'EntityManager gère automatiquement un cache utilisé pour optimiser les accès à la base de données. Par conséquent, les requêtes effectuées par l'EntityManager ne génèrent pas nécessairement des requêtes SQL vers la base si l'objet demandé par notre application est déjà présent dans le cache de l'EntityManager. JPA évitera ainsi l'envoi d'une requête SQL à la base de données.

En réalité, ce cache fonctionne à deux niveaux, car l'objet EntityManagerFactory gère également son propre cache. Ainsi, JPA gère un cache à deux niveaux : un premier niveau au niveau de l'EntityManager, qui est valable pour l'ensemble de la base de données, et un deuxième niveau de cache associé à chaque EntityManager créé à partir de cet EntityManagerFactory et partageant le cycle de vie de cet EntityManager. Cette double stratégie de cache permet d'optimiser les performances et offre des résultats tout à fait satisfaisants pour les outils de mapping objet-relationnel.

Il convient de noter que le fonctionnement de ce cache repose sur un principe fondamental, à savoir que nous nous interdisons de modifier la base de données en contournant notre module JPA. Si nous ouvrons directement une connexion à la base de données et apportons des modifications aux données ou à la structure des tables, le cache de l'EntityManagerFactory, notamment, sera désynchronisé par rapport à la base de données et n'aura aucun moyen de détecter les modifications que nous aurions apportées de manière indépendante. Ainsi, ce cache présente l'avantage de l'optimisation, mais aussi l'inconvénient d'entraîner une utilisation exclusive de la base de données.

3.b. La deuxième solution que nous utilisons fréquemment pour la modification d'une instance JPA consiste à prendre un objet Musicien que nous avons créé de toutes pièces et qui n'a jamais été enregistré en base de données. Nous lui attribuons une valeur de clé primaire correspondant à celle d'un autre objet Musicien existant, puis nous remplissons ses champs avec les valeurs de notre application.

Au lieu de tenter de copier manuellement ses champs dans une instance de Musicien déjà connue de l'EntityManager et que nous aurions récupérée à partir de cette clé primaire, nous utilisons la méthode merge pour effectuer la fusion dans notre EntityManager. En substance, cela revient à réinsérer cet objet Musicien en base de données. Comment fonctionne notre EntityManager dans ce cas ? Il utilise la clé primaire que nous avons définie manuellement, mais qui correspond à une valeur de clé primaire déjà connue, pour copier les valeurs des champs de cet objet Musicien en base de données, mettant ainsi à jour l'entité Musicien associée à cette clé primaire. Cette méthode est particulièrement utile lorsque l'objet Musicien que nous avons extrait de la base de données a été envoyé à un module d'application qui n'a pas connaissance du module JPA, le détachant ainsi de l'EntityManager. Ensuite, cet objet est utilisé dans un formulaire utilisateur, modifié, et les modifications sont ensuite reportées en base de données.

Ainsi, ces modifications peuvent être appliquées à l'aide de la méthode _merge(...)_, qui utilise cet objet comme modèle pour mettre à jour les champs correspondants en base de données.

4. La dernière opér tion est celle de _delete(...)_