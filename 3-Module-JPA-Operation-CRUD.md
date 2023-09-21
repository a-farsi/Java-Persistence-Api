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

   -  3.a. La première méthode, la plus simple, consiste à récupérer notre entité par son nom de classe et sa clé primaire, que ce soit dans un contexte transactionnel ou non, puis à la modifier dans un contexte transactionnel. La modification de cette entité déclenchera automatiquement une mise à jour vers la base de données par le biais de JPA lors de l'appel de la méthode commit de notre EntityManager, plus précisément lors de la gestion de la transaction par cet EntityManager. À ce stade, on peut avoir l'impression qu'une forme de magie JPA opère, car une simple invocation de la méthode _setAge()_ avec une valeur d'âge différente générera une mise à jour dans la base de données.
   
   En réalité, notre objet Musicien est une entité JPA, liée à un EntityManager, et toute modification de cet objet entraînera automatiquement une mise à jour de la base de données. C'est pourquoi on parle de mapping Objet-Relationnel. L'objectif du mapping ORM est de refléter en base de données automatiquement les modifications apportées à notre modèle objet. Il convient également de noter que l'EntityManager gère automatiquement un cache utilisé pour optimiser les accès à la base de données. Par conséquent, les requêtes effectuées par l'EntityManager ne génèrent pas nécessairement des requêtes SQL vers la base si l'objet demandé par notre application est déjà présent dans le cache de l'EntityManager. JPA évitera ainsi l'envoi d'une requête SQL à la base de données.

```java
        // Création de l'EntityManagerFactory (à configurer selon votre environnement)
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa-test");
        // Création de l'EntityManager
        EntityManager em = emf.createEntityManager();
        // Id du Musicien que nous voulons mettre à jour
        Long musicienId = 1L;
        // Début d'une transaction
        EntityTransaction transaction = entityManager.getTransaction();
        transaction.begin();
        try {
            // Chargement de l'entité Musicien par son id (clé primaire)
            Musicien musicien = em.find(Musicien.class, musicienId);
            if (musicien != null) {
                // Modification de l'entité Musicien
                musicien.setAge(30); // Exemple de modification de l'âge
                // La mise à jour sera automatiquement propagée à la base de données
            }
            // Validation de la transaction
            transaction.commit();
        } catch (Exception e) {
            // En cas d'erreur, annulation de la transaction
            if (transaction != null && transaction.isActive()) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            // Fermeture de l'EntityManager et de l'EntityManagerFactory
            em.close();
            emf.close();
        }
```
En réalité, la gestion de cache en JPA fonctionne à deux niveaux, car l'objet EntityManagerFactory gère également son propre cache. Ainsi, JPA met en œuvre un système de cache à deux niveaux :

1. **Premier niveau de cache (au niveau de l'EntityManager) :** Ce cache est valable pour l'ensemble de la base de données. Il est associé à l'EntityManager en cours et stocke temporairement les entités chargées depuis la base de données au cours d'une transaction spécifique. Cela permet d'optimiser les performances en évitant de répéter les requêtes pour les mêmes entités au sein de la même transaction.

2. **Deuxième niveau de cache (associé à l'EntityManagerFactory) :** Ce niveau de cache est partagé entre tous les EntityManagers créés à partir de la même EntityManagerFactory. Il conserve des données en cache qui peuvent être partagées entre plusieurs transactions et même entre plusieurs threads de l'application. Cela permet d'optimiser davantage les performances en réduisant la nécessité de requêtes répétées pour les mêmes données, même lorsque différents EntityManagers sont utilisés dans différentes parties de l'application.

Cette double stratégie de cache contribue à améliorer significativement les performances des outils de mapping objet-relationnel, en minimisant les requêtes inutiles à la base de données et en réduisant la charge sur le système de gestion de base de données.

Il convient de noter que le fonctionnement de ce cache repose sur un principe fondamental, à savoir que nous nous interdisons de modifier la base de données en contournant notre module JPA. Si nous ouvrons directement une connexion à la base de données et apportons des modifications aux données ou à la structure des tables, le cache de l'_EntityManagerFactory_, notamment, sera désynchronisé par rapport à la base de données et n'aura aucun moyen de détecter les modifications que nous aurions apportées de manière indépendante. Ainsi, ce cache présente l'avantage de l'optimisation, mais aussi l'inconvénient d'entraîner une utilisation exclusive de la base de données.

   -  3.b. La deuxième solution que nous utilisons fréquemment pour la modification d'une instance JPA consiste à prendre un objet Musicien que nous avons créé de toutes pièces et qui n'a jamais été enregistré en base de données. Nous lui attribuons une valeur de clé primaire correspondant à celle d'un autre objet Musicien existant, puis nous remplissons ses champs avec les valeurs de notre application.
   Au lieu de tenter de copier manuellement ses champs dans une instance de Musicien déjà connue de l'EntityManager et que nous aurions récupérée à partir de cette clé primaire, nous utilisons la méthode _merge(...)_ pour effectuer la fusion dans notre EntityManager. En substance, cela revient à réinsérer cet objet Musicien en base de données. Comment fonctionne notre EntityManager dans ce cas ? Il utilise la clé primaire que nous avons définie manuellement, mais qui correspond à une valeur de clé primaire déjà connue, pour copier les valeurs des champs de cet objet Musicien en base de données, mettant ainsi à jour l'entité Musicien associée à cette clé primaire. Cette méthode est particulièrement utile lorsque l'objet Musicien que nous avons extrait de la base de données a été envoyé à un module d'application qui n'a pas connaissance du module JPA, le détachant ainsi de l'EntityManager. Ensuite, cet objet est utilisé dans un formulaire utilisateur, modifié, et les modifications sont ensuite reportées en base de données. Ainsi, ces modifications peuvent être appliquées à l'aide de la méthode _merge(...)_, qui utilise cet objet comme modèle pour mettre à jour les champs correspondants en base de données.

```java
        // Création de l'EntityManagerFactory (à configurer selon votre environnement)
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("NomDeVotreUniteDePersistence");
        // Création de l'EntityManager
        EntityManager em = emf.createEntityManager();
        // Début d'une transaction
        EntityTransaction transaction = em.getTransaction();
        transaction.begin();
        try {
            // Création d'un nouvel objet Musicien (qui n'a jamais été enregistré en base de données)
            Musicien musicien = new Musicien();
            musicien.setId(1); // Attribuer une valeur de clé primaire existante
            // Remplir les champs de l'objet Musicien avec les valeurs de l'application
            musicien.setNom("John Doe");
            musicien.setAge(35);
            // ...
            // Utilisation de la méthode merge() pour fusionner l'objet avec l'EntityManager
            Musicien musicienMerged = em.merge(musicien);
            // À ce stade, l'objet musicienMerged est réinséré en base de données avec les modifications
            // Validation de la transaction
            transaction.commit();
        } catch (Exception e) {
            // En cas d'erreur, annulation de la transaction
            if (transaction != null && transaction.isActive()) {
                transaction.rollback();
            }
            e.printStackTrace();
        } finally {
            // Fermeture de l'EntityManager et de l'EntityManagerFactory
            em.close();
            emf.close();
        }
```
4. La dernière opération kde notre tétralogie CRUD est celle de la suppression, et elle concerne l'effacement d'une entité en JPA. Elle suit un schéma un peu particulier, car il est necéssaire de connaître cette entité pour pouvoir la supprimer. Nous devons charger cette entité depuis la base de données, puis la passer en tant que paramètre à la méthode _remove(...)_ de l'EntityManager.

Automatiquement, JPA envoie une requête 'delete' qui efface la ligne correspondante dans la table Musicien. Cette modification doit être effectuée dans le contexte d'une transaction.

```java
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa-test") ;
        EntityManager em = emf.createEntityManager() ;
        Long id = 1L;
        Musicien musicien =  em.find(Musicien.class, id) ;
        em.getTransaction().begin() ;
        em.remove(musicien) ;
        em.getTransaction().commit() ;
        em.close();
```

Un dernier point à noter concernant l'EntityManager est que, normalement, lorsque nous n'avons plus besoin d'un EntityManager, nous devrions appeler sa méthode 'close' pour indiquer que notre application peut libérer les ressources gérées par cet EntityManager. Cela inclut la destruction de son cache, ce qui nous permet d'économiser la mémoire de notre application.

La dernière fonctionnalité liée à l'EntityManager de JPA que nous avons abordée concerne la notion de requête. nous pouvons écrire des requêtes directement en SQL et les gérer via notre EntityManager. Nous pouvons également rédiger des requêtes dans un langage spécifique à JPA, appelé JPQL (JPA Query Language). Les requêtes peuvent être déclarées dans notre code de deux manières principales : 
  -  soit en les écrivant sous forme de chaînes de caractères et en les transmettant à notre EntityManager, 
  -  soit en les intégrant dans des annotations. Dans ce dernier cas, nos requêtes deviennent des métadonnées de notre modèle objet, et leur syntaxe est analysée lors du chargement de notre modèle JPA, c'est-à-dire lors du chargement de notre entité de persistance. 

En revanche, les requêtes écrites directement dans le code sont analysées à l'exécution, ce qui signifie qu'elles peuvent générer des erreurs lors de l'exécution de notre application plutôt qu'à son chargement.

Dans l'exemple ci-dessous, nous avons deux requêtes nommées qui s'appliquent à la classe Musicien.

```java
@Entity
@NamedQueries({
    @NamedQuery(
     name = "MUSUCIEN_FIND_ALL",
     query = "SELECT m FROM Musicien m"
   )
   @NamedQuery(
     name = "MUSUCIEN_FIND_BY_AGE",
     query = "SELECT m FROM Musicien m WHERE m.age > :age"
   )
})

public class Musicien {
    //....
}
```

Comment allons-nous procéder à l'exécution de ces requêtes ? Elles sont représentées sous forme d'objets Query, qui sont des entités de JPA. Pour créer une instance de l'objet Query, nous invoquons la méthode 'createNamedQuery' en lui transmettant le nom de la requête exprimée à l'aide d'annotations JPA.

Une fois que cet objet est créé, nous avons la possibilité de définir les valeurs des paramètres à l'aide de la méthode 'setParameter', puis d'exécuter la requête en utilisant la méthode 'getResultList'. Notre requête JPQL, qui est formulée en fonction du modèle objet, est convertie en une requête SQL par l'implémentation JPA, puis exécutée au niveau de la base de données.

Il est à noter qu'en plus des requêtes JPQL ou SQL, nous avons la possibilité d'utiliser des requêtes SQL nommées, et de les exécuter de la même manière en suivant le même schéma. Il convient de noter que ces deux types de requêtes traversent le cache et s'exécutent systématiquement au niveau de la base de données. Ensuite, lors de la lecture des objets résultants, il est possible qu'ils bénéficient du fait que le cache détient déjà ces objets, évitant ainsi la nécessité d'accéder à la base de données pour les récupérer.