# Les Patterns de Création : L'Art de Fabriquer des Objets

## Objectifs Pédagogiques

À la fin de ce module, vous serez en mesure de :

* **Comprendre** l'intention et la structure des principaux patterns de création.
* **Implémenter** en Java les patterns Singleton, Factory Method et Builder.
* **Choisir** le pattern de création le plus adapté à un problème de conception donné.
* **Identifier** les avantages et les inconvénients de chaque pattern, notamment en termes de testabilité et de
  flexibilité.
* **Appliquer** ces patterns pour améliorer l'architecture du projet fil rouge.

## Introduction : L'Importance de Bien "Construire"

Quand vous construisez un objet en Java, vous utilisez le plus souvent le mot-clé `new`. C'est simple, direct, et ça
fonctionne. Mais que se passe-t-il quand la création devient plus complexe ?

* Et si vous ne deviez avoir **qu'une seule instance** d'une classe dans toute votre application (comme un gestionnaire
  de configuration) ?
* Et si le **type d'objet à créer dépend d'une condition** qui n'est connue qu'à l'exécution (comme choisir un
  exportateur CSV ou JSON) ?
* Et si votre objet a **tellement de paramètres** dans son constructeur que vous vous y perdez et que le rendre immuable
  devient un cauchemar ?

C'est là que les **Patterns de Création** entrent en jeu. Ils ne se contentent pas de créer des objets ; ils vous
donnent le **contrôle** sur le processus de création. Ils permettent de masquer la complexité, d'améliorer la
flexibilité et de rendre votre code beaucoup plus propre et intentionnel.

## L'essentiel : Les Incontournables de la Création

### Le Pattern Singleton

#### L'intention

**Assurer qu'une classe n'a qu'une seule et unique instance et fournir un point d'accès global à cette instance.**

C'est probablement le pattern le plus connu (et parfois le plus mal utilisé !). Pensez au président d'un pays : il n'y
en a qu'un seul à un instant T. Si quelqu'un a besoin de s'adresser au président, il ne crée pas un "nouveau" président,
il s'adresse à l'unique instance existante.

#### Structure et Implémentation

Pour garantir l'unicité, le pattern Singleton combine deux choses :

1. Un **constructeur privé** pour empêcher la création d'instances depuis l'extérieur (`new MySingleton()` sera
   impossible).
2. Une **méthode statique** (`getInstance()`) qui crée l'instance si elle n'existe pas, ou retourne l'instance existante
   sinon.

##### Diagramme UML

<code-block lang="plantuml">
@startuml
!theme vibrant
title Diagramme du Pattern Singleton

class Singleton {

- {static} instance: Singleton
- Singleton()

+ {static} getInstance(): Singleton
  }

note right of Singleton::instance
L'unique instance de la classe,
stockée dans un champ statique.
end note

note right of Singleton::Singleton()
Le constructeur est privé,
personne ne peut instancier
la classe depuis l'extérieur.
end note

note right of Singleton::getInstance
La méthode statique qui fournit
l'accès à l'instance unique.
end note
@enduml
</code-block>

##### Exemple : Un gestionnaire de configuration

Imaginons une classe qui charge un fichier de configuration une seule fois et le met à disposition de toute
l'application.

```java
package fr.formation.spring.app.singleton;

import java.util.Properties;

// Un gestionnaire de configuration implémentant le pattern Singleton.
public class AppConfig {

    // 1. L'unique instance, déclarée private et static.
    // L'attribut 'volatile' garantit que les changements sur cette variable
    // sont visibles par tous les threads.
    private static volatile AppConfig instance;

    private Properties properties;

    // 2. Le constructeur est privé.
    private AppConfig() {
        // Simule le chargement d'un fichier de configuration.
        // Dans une vraie app, on lirait un fichier .properties ici.
        this.properties = new Properties();
        this.properties.setProperty("app.version", "1.0.0");
        this.properties.setProperty("database.url", "jdbc:h2:mem:librarydb");
        System.out.println("Instance de AppConfig créée et configurée.");
    }

    // 3. La méthode statique pour obtenir l'instance.
    public static AppConfig getInstance() {
        // Double-Checked Locking pour la performance en environnement multi-threadé.
        if (instance == null) {
            synchronized (AppConfig.class) {
                if (instance == null) {
                    instance = new AppConfig();
                }
            }
        }
        return instance;
    }

    public String getProperty(String key) {
        return this.properties.getProperty(key);
    }
}
```

<tip>
**Qu'est-ce que le "Double-Checked Locking" ?**
C'est une optimisation pour les singletons en environnement multi-threadé.
1.  Le premier `if (instance == null)` est rapide et évite d'entrer dans le bloc `synchronized` (qui est coûteux) si l'instance existe déjà.
2.  Le bloc `synchronized` garantit qu'un seul thread peut créer l'instance.
3.  Le deuxième `if (instance == null)` est crucial. Si deux threads passent le premier `if` en même temps, le premier entrera dans le bloc `synchronized` et créera l'instance. Quand le second thread entrera à son tour, ce deuxième `if` l'empêchera de créer une nouvelle instance.
</tip>

#### Cas d'utilisation et Bonnes Pratiques

* **À utiliser pour :** Des services transversaux qui doivent être uniques par nature : gestionnaires de logs, pools de
  connexions, caches, gestionnaires de configuration.
* **Attention :** Le Singleton est souvent critiqué car il introduit un **état global** dans l'application, ce qui peut
  rendre les tests unitaires difficiles. Il crée une dépendance forte (couplage) entre le client et la classe du
  singleton.

<warning title="Singleton et Spring">
Dans un écosystème comme Spring, le framework gère lui-même le cycle de vie des beans. Par défaut, **tous les beans Spring sont des Singletons** (au sein du conteneur Spring). Vous n'avez donc généralement pas besoin d'implémenter ce pattern manuellement. Il suffit de déclarer votre classe comme un `@Component` (ou `@Service`, `@Repository`) et de l'injecter là où vous en avez besoin. C'est le principe de l'**Inversion de Contrôle (IoC)**, qui est souvent une meilleure alternative au Singleton manuel.
</warning>

### Exercice 2 : Votre premier Singleton

Créez une classe `LoggerService` qui suit le pattern Singleton. Cette classe doit permettre d'écrire des messages de log
dans la console, en préfixant chaque message par un timestamp.

**Cahier des charges :**

1. La classe `LoggerService` doit être un Singleton.
2. Elle doit avoir une méthode `log(String message)` qui affiche `[YYYY-MM-DDTHH:mm:ss.sss] message` dans la console.
3. Créez une classe `Main` pour tester que vous obtenez bien la même instance à chaque appel de `getInstance()`.

### Correction exercice 2 {collapsible='true''}

Voici une solution possible pour l'exercice.

##### LoggerService.java

```java
package fr.formation.spring.app.singleton;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class LoggerService {

    private static volatile LoggerService instance;
    private final DateTimeFormatter formatter =
            DateTimeFormatter.ISO_LOCAL_DATE_TIME;

    private LoggerService() {
        // Constructeur privé pour empêcher l'instanciation directe.
    }

    public static LoggerService getInstance() {
        if (instance == null) {
            synchronized (LoggerService.class) {
                if (instance == null) {
                    instance = new LoggerService();
                }
            }
        }
        return instance;
    }

    public void log(String message) {
        String timestamp = LocalDateTime.now().format(formatter);
        System.out.println("[" + timestamp + "] " + message);
    }
}
```

##### TestLogger.java

```java
package fr.formation.spring.app.singleton;

public class TestLogger {
    public static void main(String[] args) {
        System.out.println("Démarrage du test du LoggerService...");

        // On récupère deux "instances" du logger
        LoggerService logger1 = LoggerService.getInstance();
        LoggerService logger2 = LoggerService.getInstance();

        // On vérifie si les deux variables pointent vers le même objet en mémoire
        if (logger1 == logger2) {
            System.out.println("logger1 et logger2 sont la même instance. Singleton OK !");
        } else {
            System.out.println("ERREUR : logger1 et logger2 sont des instances différentes.");
        }

        // On utilise le logger
        logger1.log("Ceci est le premier message.");
        logger2.log("Ceci est le deuxième message, via la seconde variable.");
    }
}
```

**Résultat attendu dans la console :**

```
Démarrage du test du LoggerService...
logger1 et logger2 sont la même instance. Singleton OK !
[2023-11-27T10:30:00.123] Ceci est le premier message.
[2023-11-27T10:30:00.124] Ceci est le deuxième message, via la seconde variable.
```

---

### Le Pattern Factory Method (Fabrique)

#### L'intention {id="l-intention_2"}

**Définir une interface pour créer un objet, mais laisser les sous-classes décider de la classe à instancier.** La
Factory Method permet à une classe de déléguer l'instanciation à ses sous-classes.

Imaginez une usine de logistique. Le manager de l'usine (`Creator`) sait qu'il doit "créer un moyen de transport", mais
il ne sait pas lequel. C'est la commande du client qui va déterminer s'il faut créer un camion (`ConcreteProduct1`) ou
un bateau (`ConcreteProduct2`). Le choix est délégué à des départements spécialisés (`ConcreteCreator`s).

#### Structure et Implémentation {id="structure-et-impl-mentation_2"}

##### Diagramme UML {id="diagramme-uml_2"}

<code-block lang="plantuml">
@startuml
!theme vibrant
title "Diagramme du Pattern Factory Method"

interface Product {

+ operation(): void
  }

class ConcreteProductA implements Product {

+ operation(): void
  }

class ConcreteProductB implements Product {

+ operation(): void
  }

abstract class Creator {

+ {abstract} factoryMethod(): Product
+ someOperation(): void
  }

note right of Creator::factoryMethod
La méthode de fabrique.
Elle est abstraite pour forcer
les sous-classes à l'implémenter.
end note

class ConcreteCreatorA extends Creator {

+ factoryMethod(): Product
  }

class ConcreteCreatorB extends Creator {

+ factoryMethod(): Product
  }

ConcreteCreatorA ..> ConcreteProductA : crée
ConcreteCreatorB ..> ConcreteProductB : crée

Client -> Creator : utilise

@enduml
</code-block>

##### Exemple : Un système d'export de données

Dans notre application de bibliothèque, nous voulons pouvoir exporter la liste des livres en différents formats (CSV,
JSON).

1. **Le Produit (Product)** : Une interface commune pour tous les exportateurs.

```java
package fr.formation.spring.app.factory;

// Interface "Product"
public interface DataExporter {
    String export(Object data);
}
```

2. **Les Produits Concrets (ConcreteProduct)** : Les implémentations pour chaque format.

```java
package fr.formation.spring.app.factory;

// ConcreteProduct A
public class CsvExporter implements DataExporter {
    @Override
    public String export(Object data) {
        System.out.println("Exportation des données au format CSV...");
        // Logique de conversion de l'objet data en chaîne CSV.
        return "id,title\n1,Dune\n2,LOTR";
    }
}
```

```java
package fr.formation.spring.app.factory;

// ConcreteProduct B
public class JsonExporter implements DataExporter {
    @Override
    public String export(Object data) {
        System.out.println("Exportation des données au format JSON...");
        // Logique de conversion (ex: avec Jackson/Gson).
        return "[{\"id\":1, \"title\":\"Dune\"}, {\"id\":2, \"title\":\"LOTR\"}]";
    }
}
```

3. **La Fabrique (Creator)** : Une classe qui fournit l'exportateur demandé. Ici, nous utilisons une version simplifiée
   et très courante où la fabrique n'est pas abstraite, mais a une méthode qui prend un paramètre pour décider quoi
   créer.

```java
package fr.formation.spring.app.factory;

public class ExporterFactory {

    // La "Factory Method"
    public static DataExporter createExporter(String format) {
        if ("CSV".equalsIgnoreCase(format)) {
            return new CsvExporter();
        } else if ("JSON".equalsIgnoreCase(format)) {
            return new JsonExporter();
        } else {
            throw new IllegalArgumentException("Format non supporté : " + format);
        }
    }
}
```

<tip>
L'avantage principal est le **découplage**. Le code qui a besoin d'un exportateur (le client) n'a plus besoin de connaître les classes concrètes `CsvExporter` ou `JsonExporter`. Il ne connaît que l'interface `DataExporter` et la fabrique `ExporterFactory`. Si demain on ajoute un `XmlExporter`, le code client n'a pas besoin d'être modifié ! C'est une parfaite illustration du **Principe Ouvert/Fermé**.
</tip>

### Exercice 3 : Mettre en place une fabrique de notifications

Reprenons le mauvais exemple du premier chapitre ! Votre mission est de le refactoriser en utilisant le pattern Factory
Method.

**Cahier des charges :**

1. Créez une interface `Notifier` avec une méthode `send(String userId, String message)`.
2. Créez trois classes qui implémentent cette interface : `EmailNotifier`, `SmsNotifier`, et `PushNotifier`.
3. Créez une classe `NotifierFactory` avec une méthode statique `getNotifier(String type)` qui retourne l'instance
   appropriée du notificateur.
4. Modifiez le service `NotificationService` pour qu'il utilise cette fabrique au lieu de sa structure `if-else`.

### Correction exercice 3 {collapsible='true''}

Excellent ! Voici comment le code devient propre et extensible.

##### 1. L'interface `Notifier`

```java
package fr.formation.spring.app.notifications.factory;

public interface Notifier {
    void send(String userId, String message);
}
```

##### 2. Les implémentations concrètes

```java
package fr.formation.spring.app.notifications.factory;

public class EmailNotifier implements Notifier {
    @Override
    public void send(String userId, String message) {
        System.out.println("--- ENVOI EMAIL ---");
        System.out.println("À: " + userId + "@email.com");
        System.out.println("Message: " + message);
        System.out.println("--------------------");
    }
}
```

```java
package fr.formation.spring.app.notifications.factory;

public class SmsNotifier implements Notifier {
    @Override
    public void send(String userId, String message) {
        System.out.println("--- ENVOI SMS ---");
        System.out.println("Numéro pour: " + userId);
        System.out.println("Message: " + message);
        System.out.println("-----------------");
    }
}
```

```java
package fr.formation.spring.app.notifications.factory;

public class PushNotifier implements Notifier {
    @Override
    public void send(String userId, String message) {
        System.out.println("--- NOTIFICATION PUSH ---");
        System.out.println("Device Token pour: " + userId);
        System.out.println("Message: " + message);
        System.out.println("-------------------------");
    }
}
```

##### 3. La `NotifierFactory`

```java
package fr.formation.spring.app.notifications.factory;

public class NotifierFactory {

    public static Notifier getNotifier(String type) {
        if (type == null || type.isEmpty()) {
            return null;
        }
        switch (type.toUpperCase()) {
            case "EMAIL":
                return new EmailNotifier();
            case "SMS":
                return new SmsNotifier();
            case "PUSH":
                return new PushNotifier();
            default:
                throw new IllegalArgumentException("Type de notificateur inconnu: " + type);
        }
    }
}
```

<tip>
Utiliser un `switch` est souvent plus propre et performant qu'une longue chaîne de `if-else if`.</tip>

##### 4. Le nouveau `NotificationService`

```java
package fr.formation.spring.app.notifications.factory;

public class NotificationService {

    public void sendNotification(String user, String message, String type) {
        Notifier notifier = NotifierFactory.getNotifier(type);
        if (notifier != null) {
            notifier.send(user, message);
        }
    }
}
```

Regardez comme cette classe est devenue simple ! Sa seule responsabilité est d'orchestrer l'envoi. Pour ajouter un
notificateur `Slack`, il suffit de créer la classe `SlackNotifier` et d'ajouter un `case "SLACK"` dans la fabrique. La
classe `NotificationService` **n'a pas besoin d'être modifiée**. Le Principe Ouvert/Fermé est respecté.

---

### Le Pattern Builder (Monteur)

#### L'intention {id="l-intention_1"}

**Séparer la construction d'un objet complexe de sa représentation**, de sorte que le même processus de construction
puisse créer différentes représentations.

C'est la solution élégante au problème des constructeurs à rallonge (les "telescoping constructors") et un excellent
moyen de créer des objets **immuables**.

Pensez à la préparation d'un sandwich chez Subway. Vous (`Director`) ne vous souciez pas de comment on coupe le pain ou
on dispose les légumes. Vous donnez des instructions à l'employé (`Builder`) : "Je veux un pain italien (`buildPain()`),
du poulet teriyaki (`buildGarniture()`), pas de fromage (`buildFromage()`), et une sauce sweet onion (`buildSauce()`)".
À la fin, vous demandez le résultat final (`getResult()`). Le processus de construction est le même, mais le résultat (
le sandwich, ou `Product`) peut être très différent.

#### Structure et Implémentation {id="structure-et-impl-mentation_1"}

##### Diagramme UML {id="diagramme-uml_1"}

<code-block lang="plantuml">
@startuml
!theme vibrant
title "Diagramme du Pattern Builder"

class Product {

- partA: String
- partB: String
- partC: String

+ Product(builder: ConcreteBuilder)
  }

interface Builder {

+ buildPartA(value: String): Builder
+ buildPartB(value: String): Builder
+ buildPartC(value: String): Builder
+ build(): Product
  }

class ConcreteBuilder implements Builder {

- partA: String
- partB: String
- partC: String

+ buildPartA(value: String): Builder
+ buildPartB(value: String): Builder
+ buildPartC(value: String): Builder
+ build(): Product
  }

class Director {

- builder: Builder

+ constructSportsCar(): Product
+ constructSUV(): Product
  }

Director -> Builder : utilise
ConcreteBuilder ..> Product : crée

note on link: Le Director connaît le\nprocessus de construction.

note right of ConcreteBuilder::build
La méthode `build()` retourne
le produit final, souvent en
passant `this` (le builder lui-même)
au constructeur du produit.
end note

@enduml
</code-block>

##### Exemple : Construire un livre

Notre entité `Book` peut devenir complexe. Utilisons un Builder pour la créer proprement, surtout si nous voulons la
rendre immuable (avec des champs `final`).

1. **Le Produit (Product)** : L'objet `Book`. Pour le rendre immuable, ses champs sont `final` et il n'y a pas de
   setters.

```java
// Version immuable de notre entité Book
public class ImmutableBook {
    private final Long id;
    private final String title;
    private final String isbn;
    private final Author author;
    private final Set<Category> categories;

    // Le constructeur est privé et n'est appelé que par le Builder
    private ImmutableBook(BookBuilder builder) {
        this.id = builder.id;
        this.title = builder.title;
        this.isbn = builder.isbn;
        this.author = builder.author;
        this.categories = builder.categories;
    }

    // Uniquement des getters...
}
```

2. **Le Monteur (Builder)** : Une classe, souvent statique et interne au produit, qui accumule les données de
   configuration.

```java
public static class BookBuilder {
    // Les champs du builder reflètent ceux du produit
    private Long id;
    private String title;
    private String isbn;
    private Author author;
    private Set<Category> categories;

    // Le constructeur du builder prend les paramètres obligatoires
    public BookBuilder(String title, String isbn) {
        this.title = title;
        this.isbn = isbn;
    }

    // Méthodes "fluent" pour les paramètres optionnels
    public BookBuilder withId(Long id) {
        this.id = id;
        return this; // Renvoyer 'this' permet le chaînage
    }

    public BookBuilder byAuthor(Author author) {
        this.author = author;
        return this;
    }

    public BookBuilder inCategories(Set<Category> categories) {
        this.categories = categories;
        return this;
    }

    // La méthode finale qui construit l'objet
    public ImmutableBook build() {
        // On peut ajouter des validations ici
        if (title == null || title.isEmpty()) {
            throw new IllegalStateException("Le titre ne peut être vide");
        }
        return new ImmutableBook(this);
    }
}
```

##### Utilisation

Le code client devient incroyablement lisible :

```java
Author herbert = new Author();
herbert.

setName("Frank Herbert");

ImmutableBook dune = new ImmutableBook.BookBuilder("Dune", "978-2266286260")
        .byAuthor(herbert)
        .inCategories(Set.of(new Category("SF")))
        .build();
```

### Exercice 4 : Créer un livre via une API REST avec un Builder

C'est l'heure de mettre les mains dans le projet fil rouge ! Nous allons créer un endpoint `POST /api/books` pour
ajouter un nouveau livre.

**Cahier des charges :**

1. Créez un DTO (Data Transfer Object) `BookCreationDTO` qui recevra les données de la requête HTTP. Il contiendra
   `title`, `isbn`, `authorId` et une `Set<Long>` de `categoryIds`.
2. Dans la classe `Book`, ajoutez une classe interne statique `BookBuilder`. Ce builder devra pouvoir construire une
   entité `Book` complète.
3. Créez un `BookService` avec une méthode `createBook(BookCreationDTO dto)`. Cette méthode utilisera le `BookBuilder`.
   Elle devra récupérer l'auteur et les catégories à partir de leurs IDs en utilisant les repositories.
4. Créez un `BookController` avec une méthode `POST` mappée sur `/api/books` qui utilise le `BookService`.
5. Fournissez la requête HTTP (format `HTTP Client` d'IntelliJ) pour tester votre endpoint.

### Correction exercice 4 {collapsible='true''}

Absolument ! Voici la solution complète pour cet exercice.

##### 1. Le DTO `BookCreationDTO.java`

Créez un package `fr.formation.spring.app.dto`.

```java
package fr.formation.spring.app.dto;

import java.util.Set;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class BookCreationDTO {
    private String title;
    private String isbn;
    private Long authorId;
    private Set<Long> categoryIds;
}
```

##### 2. L'entité `Book` avec son `BookBuilder`

Modifiez l'entité `Book.java` pour y inclure le builder. Pour cet exercice, nous n'allons pas la rendre immuable pour
rester compatible avec JPA, mais le builder reste très utile.

```java
package fr.formation.spring.app.entities;

// ... imports et annotations de l'entité ...

@Entity
@Getter
@Setter
public class Book {
    // ... champs existants ...

    // Builder statique interne
    public static class BookBuilder {
        private String title;
        private String isbn;
        private Author author;
        private Set<Category> categories;

        public BookBuilder(String title, String isbn) {
            this.title = title;
            this.isbn = isbn;
        }

        public BookBuilder withAuthor(Author author) {
            this.author = author;
            return this;
        }

        public BookBuilder withCategories(Set<Category> categories) {
            this.categories = categories;
            return this;
        }

        public Book build() {
            Book book = new Book();
            book.setTitle(this.title);
            book.setIsbn(this.isbn);
            book.setAuthor(this.author);
            book.setCategories(this.categories);
            return book;
        }
    }
}
```

##### 3. Le `BookService.java`

Créez un package `fr.formation.spring.app.services`.

```java
package fr.formation.spring.app.services;

import fr.formation.spring.app.dto.BookCreationDTO;
import fr.formation.spring.app.entities.Author;
import fr.formation.spring.app.entities.Book;
import fr.formation.spring.app.entities.Category;
import fr.formation.spring.app.repositories.AuthorRepository;
import fr.formation.spring.app.repositories.BookRepository;
import fr.formation.spring.app.repositories.CategoryRepository;
import jakarta.persistence.EntityNotFoundException;
import org.springframework.stereotype.Service;

import java.util.HashSet;
import java.util.Set;

@Service
public class BookService {

    private final BookRepository bookRepository;
    private final AuthorRepository authorRepository;
    private final CategoryRepository categoryRepository;

    public BookService(BookRepository br, AuthorRepository ar, CategoryRepository cr) {
        this.bookRepository = br;
        this.authorRepository = ar;
        this.categoryRepository = cr;
    }

    public Book createBook(BookCreationDTO dto) {
        // 1. Récupérer les entités dépendantes
        Author author = authorRepository.findById(dto.getAuthorId())
                .orElseThrow(() -> new EntityNotFoundException("Auteur non trouvé"));

        Set<Category> categories = new HashSet<>(
                categoryRepository.findAllById(dto.getCategoryIds())
        );

        // 2. Utiliser le Builder
        Book newBook = new Book.BookBuilder(dto.getTitle(), dto.getIsbn())
                .withAuthor(author)
                .withCategories(categories)
                .build();

        // 3. Sauvegarder et retourner le livre créé
        return bookRepository.save(newBook);
    }
}
```

##### 4. Le `BookController.java`

Créez un package `fr.formation.spring.app.controllers`.

```java
package fr.formation.spring.app.controllers;

import fr.formation.spring.app.dto.BookCreationDTO;
import fr.formation.spring.app.entities.Book;
import fr.formation.spring.app.services.BookService;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/books")
public class BookController {

    private final BookService bookService;

    public BookController(BookService bookService) {
        this.bookService = bookService;
    }

    @PostMapping
    public ResponseEntity<Book> createBook(@RequestBody BookCreationDTO bookDTO) {
        Book createdBook = bookService.createBook(bookDTO);
        return new ResponseEntity<>(createdBook, HttpStatus.CREATED);
    }
}
```

##### 5. La requête HTTP pour tester

Créez un fichier `requests.http` à la racine de votre projet.

```http
### Créer un nouveau livre

# Pour que cette requête fonctionne, assurez-vous que l'auteur avec l'ID 1
# et la catégorie avec l'ID 1 existent bien dans la base de données.
# Notre DataInitializer s'en charge au démarrage. Frank Herbert a l'ID 1
# et Science-Fiction a l'ID 1.

POST http://localhost:8080/api/books
Content-Type: application/json

{
  "title": "Le Messie de Dune",
  "isbn": "978-2266155450",
  "authorId": 1,
  "categoryIds": [1]
}
```

En exécutant cette requête, vous devriez recevoir une réponse `201 Created` avec le JSON du livre nouvellement créé, y
compris son ID généré par la base de données.

## Pour aller plus loin

### Le Pattern Abstract Factory (Fabrique Abstraite)

#### L'intention {id="l-intention_4"}

**Fournir une interface pour créer des familles d'objets liés ou dépendants sans spécifier leurs classes concrètes.**

C'est une "usine d'usines". Imaginez que vous devez créer une interface utilisateur. Vous avez besoin d'un bouton ET d'
une fenêtre. Si le thème est "Windows", il vous faut un `WindowsButton` et une `WindowsWindow`. Si le thème est "macOS",
il vous faut un `MacButton` et une `MacWindow`.

La Fabrique Abstraite (`UIFactory`) fournit des méthodes comme `createButton()` et `createWindow()`. Vous aurez ensuite
des implémentations concrètes comme `WindowsFactory` et `MacFactory` qui s'assureront de créer une famille cohérente de
produits.

#### Diagramme UML {id="diagramme-uml_3"}

<code-block lang="plantuml">
@startuml
!theme vibrant
title "Diagramme du Pattern Abstract Factory"

interface AbstractProductA { }
class ConcreteProductA1 implements AbstractProductA
class ConcreteProductA2 implements AbstractProductA

interface AbstractProductB { }
class ConcreteProductB1 implements AbstractProductB
class ConcreteProductB2 implements AbstractProductB

interface AbstractFactory {

+ createProductA(): AbstractProductA
+ createProductB(): AbstractProductB
  }

class ConcreteFactory1 implements AbstractFactory {

+ createProductA(): AbstractProductA
+ createProductB(): AbstractProductB
  }
  class ConcreteFactory2 implements AbstractFactory {
+ createProductA(): AbstractProductA
+ createProductB(): AbstractProductB
  }

ConcreteFactory1 ..> ConcreteProductA1 : crée
ConcreteFactory1 ..> ConcreteProductB1 : crée
ConcreteFactory2 ..> ConcreteProductA2 : crée
ConcreteFactory2 ..> ConcreteProductB2 : crée

Client -> AbstractFactory
Client -> AbstractProductA
Client -> AbstractProductB

@enduml
</code-block>

![abstract-factory-Diagramme_du_Pattern_Abstract_Factory.svg](abstract-factory-Diagramme_du_Pattern_Abstract_Factory.svg)


### Le Pattern Prototype

#### L'intention {id="l-intention_3"}

**Spécifier les types d'objets à créer en utilisant une instance prototype, et créer de nouveaux objets en copiant ce
prototype.**

C'est le pattern du clonage. Plutôt que de construire un objet complexe de zéro, vous prenez un objet existant et vous
le copiez. C'est très efficace si la création d'un objet est coûteuse (ex: nécessite un appel à la base de données).

En Java, cela se fait via l'interface `Cloneable` et la méthode `clone()`.

<warning>
Attention au **Shallow Copy vs. Deep Copy**.
*   **Shallow Copy (copie superficielle) :** Copie les champs de types primitifs et les références. Les objets référencés ne sont PAS copiés. Le clone et l'original partagent les mêmes objets dépendants.
*   **Deep Copy (copie profonde) :** Copie tout, y compris les objets référencés, en créant de nouvelles instances pour eux. C'est souvent ce que l'on veut, mais c'est plus complexe à implémenter.
</warning>

## Auto-évaluation

Testez vos nouvelles compétences sur les patterns de création !

<warning>
Les corrections de cette auto-évaluation se trouvent à la fin de l'ensemble du support de cours.
</warning>

**Questions à Choix Multiple (QCM)**

1. Vous devez créer un objet qui a 2 paramètres obligatoires et 8 paramètres optionnels. Quel pattern est le plus adapté
   pour rendre le code de création lisible et sûr ?
    * a) Singleton
    * b) Factory Method
    * c) Builder
    * d) Prototype

2. Le principal avantage du pattern Factory Method est qu'il respecte quel principe S.O.L.I.D. ?
    * a) Single Responsibility Principle
    * b) Open/Closed Principle
    * c) Liskov Substitution Principle
    * d) Interface Segregation Principle

3. Quel est le principal inconvénient de l'utilisation intensive du pattern Singleton (manuel) ?
    * a) Il est très lent à l'exécution.
    * b) Il rend les tests unitaires difficiles et crée un couplage fort.
    * c) Il ne peut pas être utilisé dans un environnement multi-threadé.
    * d) Il consomme beaucoup de mémoire.

4. Une "Fabrique Abstraite" (Abstract Factory) est conçue pour...
    * a) Créer une seule instance d'un objet.
    * b) Créer des familles d'objets liés et cohérents.
    * c) Cloner un objet existant.
    * d) Rendre un objet immuable.

**Questions Ouvertes**

5. Expliquez la différence fondamentale entre le pattern Factory Method et le pattern Abstract Factory.
6. Vous utilisez le pattern Prototype pour cloner un objet `Order` qui contient une `List<OrderItem>` et une référence à
   un objet `Customer`. Décrivez ce qui se passe lors d'une copie superficielle (shallow copy) et pourquoi cela peut
   être problématique.

## Conclusion

Félicitations ! Vous venez de maîtriser les outils les plus puissants pour contrôler la création d'objets.

Vous avez appris à :

* Garantir l'unicité d'une instance avec le **Singleton**.
* Déléguer la logique d'instanciation à des sous-classes avec la **Factory Method**, découplant ainsi votre code.
* Construire des objets complexes pas à pas et de manière lisible avec le **Builder**.
* Et vous avez eu un aperçu de la création de familles d'objets avec l'**Abstract Factory** et du clonage avec le *
  *Prototype**.

Ces patterns sont la fondation d'un code flexible et maintenable. Vous les retrouverez partout, que ce soit dans le JDK,
dans Spring, ou dans les librairies que vous utilisez tous les jours.

Maintenant que nous savons comment *créer* nos objets proprement, le prochain chapitre nous apprendra comment les
*assembler* pour former des structures plus grandes et plus robustes. Place aux **Patterns de Structure** 