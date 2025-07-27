# Les Patterns de Structure : L'Art d'Assembler les Objets

## Objectifs Pédagogiques

À la fin de ce module, vous serez capable de :

* **Décrire** comment les patterns de structure simplifient les relations entre les entités.
* **Implémenter** en Java les patterns Adapter, Decorator et Facade.
* **Utiliser** un Adapter pour faire collaborer des classes aux interfaces incompatibles.
* **Ajouter** des fonctionnalités à un objet dynamiquement avec un Decorator.
* **Simplifier** l'interface d'un sous-système complexe avec une Facade.
* **Choisir** le pattern de structure adéquat pour résoudre des problèmes de couplage et de complexité.

## Introduction : Des Briques aux Murs

Dans le module précédent, nous avons appris à fabriquer nos briques (nos objets) de la meilleure façon possible. Mais
une pile de briques ne fait pas une maison. Il faut maintenant les assembler, créer des murs, des portes, des fenêtres,
pour former une structure cohérente et fonctionnelle.

C'est précisément le rôle des **Patterns de Structure**. Ils s'intéressent à la **composition des classes et des objets
**. Comment les faire travailler ensemble ? Comment simplifier leurs relations ? Comment ajouter des fonctionnalités
sans tout casser ?

Ces patterns sont les principes d'architecture qui transforment un ensemble de classes disjointes en un système robuste
et élégant. Vous allez apprendre à créer des "traducteurs", à "décorer" vos objets à la volée, et à cacher la complexité
derrière des "façades" simples.

## L'essentiel : Les Piliers de l'Architecture

### Le Pattern Adapter (Adaptateur)

#### L'intention

**Convertir l'interface d'une classe en une autre interface attendue par le client.** L'Adapter permet à des classes de
collaborer alors qu'elles ont des interfaces incompatibles.

C'est votre adaptateur de prise électrique universel. Vous êtes en Angleterre avec votre chargeur français (interface
incompatible). Vous n'allez pas acheter un nouveau chargeur (modifier le code existant). Vous utilisez un adaptateur qui
fait le pont entre la prise murale anglaise et votre prise française.

#### Structure et Implémentation

##### Diagramme UML

<code-block lang="plantuml">
@startuml
!theme vibrant
title "Diagramme du Pattern Adapter"

class Client {

- target: Target
  }
  note left: Le client ne connaît que l'interface Target.

interface Target {

+ request(): void
  }

class Adapter implements Target {

- adaptee: Adaptee

+ request(): void
  }
  note on link: L'Adapter implémente l'interface\nattendue par le client.

class Adaptee {

+ specificRequest(): void
  }
  note right: C'est la classe existante\navec l'interface incompatible.

Client -> Target
Adapter ..|> Target
Adapter -> Adaptee : délègue l'appel

note top of Adapter
Dans sa méthode `request()`,
l'Adapter appelle `adaptee.specificRequest()`,
faisant ainsi la "traduction".
end note

@enduml
</code-block>

##### Exemple : Adapter un ancien service de données

Imaginons que notre application de bibliothèque doive s'interfacer avec un ancien système qui fournit les données d'un
livre sous un format étrange : une `Map<String, String>`. Notre application, elle, s'attend à manipuler un `BookDTO`.

1. **L'interface cible (Target)** : Ce que notre système attend.

```java
package fr.formation.spring.app.adapter;

// Notre DTO moderne
public class BookDTO {
    private String title;
    private String authorName;
    // Getters et Setters...
}
```

Nous n'avons pas d'interface `Target` formelle, mais on peut dire que notre client attend un `BookDTO`.

2. **La classe à adapter (Adaptee)** : Le service hérité.

```java
package fr.formation.spring.app.adapter;

import java.util.Map;
import java.util.HashMap;

// Le service existant avec son interface "incompatible"
public class LegacyBookProvider {
    public Map<String, String> fetchBookData(long id) {
        // Simule un appel à un vieux système
        Map<String, String> bookData = new HashMap<>();
        bookData.put("bookTitle", "1984");
        bookData.put("writer", "George Orwell");
        return bookData;
    }
}
```

3. **L'adaptateur (Adapter)**

```java
package fr.formation.spring.app.adapter;

import java.util.Map;

public class BookServiceAdapter {
    private final LegacyBookProvider legacyProvider = new LegacyBookProvider();

    // La méthode de l'adapter qui correspond à ce que le client veut
    public BookDTO getBook(long id) {
        // 1. Appeler l'ancien système
        Map<String, String> legacyData = legacyProvider.fetchBookData(id);

        // 2. "Traduire" les données
        BookDTO dto = new BookDTO();
        dto.setTitle(legacyData.get("bookTitle"));
        dto.setAuthorName(legacyData.get("writer"));

        // 3. Retourner l'objet attendu par le client
        return dto;
    }
}
```

Le client n'a plus besoin de connaître l'existence de `LegacyBookProvider` ni de son format de données étrange. Il
appelle simplement `bookServiceAdapter.getBook(id)` et obtient ce qu'il attend.

#### Cas d'utilisation et Bonnes Pratiques

* **Quand l'utiliser ?** Quand vous devez intégrer une librairie ou un composant externe dont l'interface ne correspond
  pas à celle requise par votre application.
* **Exemples connus :** `Arrays.asList()` adapte un tableau en une `List`. `InputStreamReader` adapte un `InputStream` (
  binaire) en `Reader` (caractères).
* L'Adapter est un pattern de **refactoring** très puissant pour intégrer du code legacy sans le modifier.

### Exercice 5 : Adapter un service de taux de change

Dans notre bibliothèque, nous voulons afficher le prix des livres en euros. Nous avons un service externe qui nous donne
les taux de change, mais il retourne le résultat en XML. Notre application, elle, préfère manipuler du JSON ou des
objets simples.

**Cahier des charges :**

1. **L'Adaptee :** Créez une classe `XmlCurrencyRateProvider` avec une méthode `getRateAsXml(String from, String to)`
   qui retourne un `String` représentant un XML simple comme `<rate currency="EUR">0.92</rate>`.
2. **L'interface "implicite" du client :** Notre client a besoin d'une méthode `getRate(String from, String to)` qui
   retourne un `double`.
3. **L'Adapter :** Créez une classe `CurrencyAdapter` qui s'occupe de :
    * Appeler le service XML.
    * Parser la chaîne XML pour en extraire la valeur `double`.
    * Retourner cette valeur.
4. Écrivez une petite classe de test pour prouver que votre adaptateur fonctionne.

### Correction exercice 5 {collapsible='true''}

Excellent exercice de "traduction" ! Voici la solution.

##### 1. L'Adaptee : `XmlCurrencyRateProvider.java`

```java
package fr.formation.spring.app.adapter.exercise;

// La classe "legacy" ou externe que nous ne pouvons/voulons pas modifier.
public class XmlCurrencyRateProvider {

    /**
     * Simule un service qui retourne un taux de change au format XML.
     * @param from Devise d'origine (ex: "USD")
     * @param to Devise de destination (ex: "EUR")
     * @return Une chaîne XML.
     */
    public String getRateAsXml(String from, String to) {
        System.out.println("Appel au service XML pour " + from + "->" + to);
        // Dans un cas réel, ce serait un appel HTTP.
        // Ici on simule la réponse pour USD -> EUR.
        if ("USD".equals(from) && "EUR".equals(to)) {
            return "<rate currency=\"EUR\">0.92</rate>";
        }
        return "<rate currency=\"EUR\">0.0</rate>";
    }
}
```

##### 2 & 3. L'Adapter : `CurrencyAdapter.java`

```java
package fr.formation.spring.app.adapter.exercise;

// Notre adaptateur qui fait le pont.
public class CurrencyAdapter {

    private final XmlCurrencyRateProvider adaptee = new XmlCurrencyRateProvider();

    /**
     * La méthode que notre système "moderne" utilisera.
     * @return un double, facile à manipuler.
     */
    public double getRate(String from, String to) {
        // 1. Appel à la méthode de l'adaptee
        String xmlData = adaptee.getRateAsXml(from, to);

        // 2. "Traduction" du format XML vers double.
        // C'est le coeur du travail de l'adaptateur.
        // ATTENTION : ce parsing est très basique et fragile.
        // Dans un vrai projet, on utiliserait une librairie comme JAXB ou Jackson-dataformat-xml.
        try {
            String valueStr = xmlData.substring(
                    xmlData.indexOf('>') + 1,
                    xmlData.lastIndexOf('<')
            );
            return Double.parseDouble(valueStr);
        } catch (Exception e) {
            System.err.println("Erreur de parsing XML: " + e.getMessage());
            return 0.0;
        }
    }
}
```

##### 4. Le test

```java
package fr.formation.spring.app.adapter.exercise;

public class TestAdapter {
    public static void main(String[] args) {
        // Le client ne connaît que l'adaptateur.
        CurrencyAdapter adapter = new CurrencyAdapter();

        // Le client demande un double, et il obtient un double !
        double rate = adapter.getRate("USD", "EUR");

        System.out.println("Le client a reçu le taux de change : " + rate);

        // Simuler un calcul de prix
        double priceInUsd = 25.0;
        double priceInEur = priceInUsd * rate;

        System.out.printf(
                "Un livre à %.2f USD coûte %.2f EUR.%n",
                priceInUsd,
                priceInEur
        );
    }
}
```

---

### Le Pattern Decorator (Décorateur)

#### L'intention {id="l-intention_2"}

**Attacher dynamiquement des responsabilités supplémentaires à un objet.** Les décorateurs offrent une alternative
flexible à l'héritage pour étendre les fonctionnalités.

C'est comme commander un café. Vous commencez avec un café simple (`Component`). Ensuite, vous pouvez le "décorer" avec
du lait (`MilkDecorator`), puis du sucre (`SugarDecorator`), puis de la chantilly (`WhippedCreamDecorator`). Chaque
décorateur "enveloppe" le précédent et ajoute son propre coût et sa propre description, tout en restant un "café".

#### Structure et Implémentation {id="structure-et-impl-mentation_2"}

##### Diagramme UML {id="diagramme-uml_2"}

<code-block lang="plantuml">
@startuml
!theme vibrant
title "Diagramme du Pattern Decorator"

interface Component {

operation(): void
}
class ConcreteComponent implements Component {

operation(): void
}
abstract class Decorator implements Component {

wrapped: Component
Decorator(c: Component)
operation(): void
}
note right: Le décorateur implémente la même\ninterface que l'objet qu'il décore.
class ConcreteDecoratorA extends Decorator {

operation(): void
addedBehavior(): void
}
class ConcreteDecoratorB extends Decorator {

operation(): void
}
Client -d-> Component

Decorator o-u- Component
note on link
Composition : le décorateur
"enveloppe" un autre Component.
end note

ConcreteDecoratorA -u-|> Decorator
ConcreteDecoratorB -u-|> Decorator
ConcreteComponent .u.|> Component
@enduml
</code-block>

![decorator-pattern-Diagramme_du_Pattern_Decorator.svg](decorator-pattern-Diagramme_du_Pattern_Decorator.svg)


##### Exemple : Ajouter des options à une boisson

1. **Le composant (Component)** : L'interface de base pour tous les objets (décorés ou non).

```java
package fr.formation.spring.app.decorator;

public interface Beverage {
    String getDescription();

    double getCost();
}
```

2. **Le composant concret (ConcreteComponent)** : L'objet de base.

```java
package fr.formation.spring.app.decorator;

public class SimpleCoffee implements Beverage {
    @Override
    public String getDescription() {
        return "Café simple";
    }

    @Override
    public double getCost() {
        return 1.50;
    }
}
```

3. **Le décorateur abstrait (Decorator)**

```java
package fr.formation.spring.app.decorator;

public abstract class BeverageDecorator implements Beverage {
    protected Beverage wrappedBeverage; // L'objet enveloppé

    public BeverageDecorator(Beverage beverage) {
        this.wrappedBeverage = beverage;
    }

    @Override
    public String getDescription() {
        // Délègue à l'objet enveloppé
        return wrappedBeverage.getDescription();
    }

    @Override
    public double getCost() {
        // Délègue à l'objet enveloppé
        return wrappedBeverage.getCost();
    }
}
```

4. **Les décorateurs concrets (ConcreteDecorator)**

```java
package fr.formation.spring.app.decorator;

public class MilkDecorator extends BeverageDecorator {
    public MilkDecorator(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return super.getDescription() + ", Lait";
    }

    @Override
    public double getCost() {
        return super.getCost() + 0.50;
    }
}
```

```java
package fr.formation.spring.app.decorator;

public class SugarDecorator extends BeverageDecorator {
    public SugarDecorator(Beverage beverage) {
        super(beverage);
    }

    @Override
    public String getDescription() {
        return super.getDescription() + ", Sucre";
    }

    @Override
    public double getCost() {
        return super.getCost() + 0.10;
    }
}
```

##### Utilisation

Le code client est très flexible :

```java
Beverage myCoffee = new SimpleCoffee();
System.out.

println(myCoffee.getDescription() +" : "+myCoffee.

getCost() +"€");

// On décore le café avec du lait
myCoffee =new

MilkDecorator(myCoffee);
System.out.

println(myCoffee.getDescription() +" : "+myCoffee.

getCost() +"€");

// On ajoute du sucre par-dessus
myCoffee =new

SugarDecorator(myCoffee);
System.out.

println(myCoffee.getDescription() +" : "+myCoffee.

getCost() +"€");
```

#### Cas d'utilisation et Bonnes Pratiques {id="cas-d-utilisation-et-bonnes-pratiques_1"}

* **Quand l'utiliser ?** Quand vous voulez ajouter des responsabilités à des objets individuels de manière dynamique et
  transparente, et quand l'héritage est trop rigide (explosion du nombre de sous-classes pour gérer toutes les
  combinaisons).
* **Exemple connu :** Le package `java.io`. `new BufferedInputStream(new FileInputStream("file.txt"))` décore un
  `FileInputStream` avec un buffer pour améliorer les performances. Les deux sont des `InputStream`.
* Le Decorator respecte parfaitement le **Principe Ouvert/Fermé**.

### Le Pattern Facade (Façade)

#### L'intention {id="l-intention_1"}

**Fournir une interface unifiée et simplifiée à un ensemble d'interfaces dans un sous-système complexe.** La Façade
définit une interface de plus haut niveau qui rend le sous-système plus facile à utiliser.

Imaginez démarrer une séance de home cinema. Vous devez allumer la TV, la mettre sur le bon canal HDMI, allumer l'ampli,
sélectionner la bonne source, allumer le lecteur Blu-ray et baisser les lumières. C'est complexe. Une télécommande
universelle avec un bouton "Soirée Film" (`Facade`) ferait toutes ces actions pour vous. Elle ne remplace pas les
appareils du sous-système, elle en simplifie juste l'accès.

#### Structure et Implémentation {id="structure-et-impl-mentation_1"}

##### Diagramme UML {id="diagramme-uml_1"}

<code-block lang="plantuml">
@startuml
!theme vibrant
title "Diagramme du Pattern Facade"

class Client

class Facade {

- subsystemA: SubSystemA
- subsystemB: SubSystemB
- subsystemC: SubSystemC

+ operation(): void
  }
  note right: La Façade fournit une méthode\nsimple qui orchestre le sous-système.

package "Sous-système Complexe" {
class SubSystemA {

+ operationA1(): void
  }
  class SubSystemB {
+ operationB2(): void
  }
  class SubSystemC {
+ operationC3(): void
  }
  }

Client -u-> Facade

Facade -u-> SubSystemA
Facade -u-> SubSystemB
Facade -u-> SubSystemC

note on link
Le client n'interagit
qu'avec la Façade.
end note

note "operation() {\n subsystemA.operationA1();\n subsystemB.operationB2();\n subsystemC.operationC3();\n}" as N1
Facade .. N1
@enduml
</code-block>

### Exercice 6 : Simplifier l'emprunt d'un livre avec une Façade

Dans notre bibliothèque, le processus d'emprunt d'un livre est complexe et fait intervenir plusieurs services :

1. `UserService` : Pour vérifier si l'utilisateur a le droit d'emprunter.
2. `InventoryService` : Pour vérifier si le livre est en stock et décrémenter le stock.
3. `LoanService` : Pour enregistrer l'emprunt dans l'historique.

Votre mission est de créer une `BorrowingFacade` pour simplifier ce processus.

**Cahier des charges :**

1. **Le sous-système :** Créez trois classes de service simples : `UserService`, `InventoryService` et `LoanService`,
   chacune avec une ou deux méthodes. Vous n'avez pas besoin de les câbler à la base de données, des
   `System.out.println` suffiront pour simuler leur fonctionnement.
2. **La Façade :** Créez une classe `BorrowingFacade` qui a une seule méthode publique :
   `borrowBook(long userId, long bookId)`.
3. Cette méthode doit orchestrer les appels aux services du sous-système dans le bon ordre.
4. **Le Client :** Créez un `BorrowingController` avec un endpoint `POST /api/borrowing/{userId}/{bookId}` qui appelle
   la méthode de votre façade.
5. Fournissez la requête HTTP pour tester.

### Correction exercice 6 {collapsible='true''}

Parfait ! Cet exercice montre à quel point une façade peut nettoyer le code de haut niveau (comme les contrôleurs).

##### 1. Le sous-système complexe

Dans le package `fr.formation.spring.app.facade.subsystem` :

```java
// UserService.java
package fr.formation.spring.app.facade.subsystem;

import org.springframework.stereotype.Service;

@Service
public class UserService {
    public boolean canUserBorrow(long userId) {
        System.out.println("Vérification des droits pour l'utilisateur " + userId);
        // Logique complexe... on suppose que l'utilisateur a le droit.
        return true;
    }
}
```

```java
// InventoryService.java
package fr.formation.spring.app.facade.subsystem;

import org.springframework.stereotype.Service;

@Service
public class InventoryService {
    public boolean isBookInStock(long bookId) {
        System.out.println("Vérification du stock pour le livre " + bookId);
        return true; // Supposons qu'il y a du stock
    }

    public void decreaseStock(long bookId) {
        System.out.println("Décrémentation du stock pour le livre " + bookId);
    }
}
```

```java
// LoanService.java
package fr.formation.spring.app.facade.subsystem;

import org.springframework.stereotype.Service;

@Service
public class LoanService {
    public void recordLoan(long userId, long bookId) {
        System.out.println("Enregistrement de l'emprunt pour l'utilisateur "
                + userId + " et le livre " + bookId);
    }
}
```

##### 2 & 3. La `BorrowingFacade.java`

Dans le package `fr.formation.spring.app.facade` :

```java
package fr.formation.spring.app.facade;

import fr.formation.spring.app.facade.subsystem.InventoryService;
import fr.formation.spring.app.facade.subsystem.LoanService;
import fr.formation.spring.app.facade.subsystem.UserService;
import org.springframework.stereotype.Component;

@Component // On la déclare comme un bean Spring
public class BorrowingFacade {

    private final UserService userService;
    private final InventoryService inventoryService;
    private final LoanService loanService;

    // Injection des dépendances du sous-système
    public BorrowingFacade(UserService us, InventoryService is, LoanService ls) {
        this.userService = us;
        this.inventoryService = is;
        this.loanService = ls;
    }

    /**
     * Orchestre le processus d'emprunt.
     * @return true si l'emprunt a réussi, false sinon.
     */
    public boolean borrowBook(long userId, long bookId) {
        System.out.println("--- Début du processus d'emprunt via la Façade ---");

        if (!userService.canUserBorrow(userId)) {
            System.err.println("L'utilisateur n'a pas le droit d'emprunter.");
            return false;
        }

        if (!inventoryService.isBookInStock(bookId)) {
            System.err.println("Le livre n'est plus en stock.");
            return false;
        }

        inventoryService.decreaseStock(bookId);
        loanService.recordLoan(userId, bookId);

        System.out.println("--- Emprunt réussi ! ---");
        return true;
    }
}
```

##### 4. Le `BorrowingController.java`

Dans le package `fr.formation.spring.app.controllers` :

```java
package fr.formation.spring.app.controllers;

import fr.formation.spring.app.facade.BorrowingFacade;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/borrowing")
public class BorrowingController {

    private final BorrowingFacade borrowingFacade;

    public BorrowingController(BorrowingFacade borrowingFacade) {
        this.borrowingFacade = borrowingFacade;
    }

    @PostMapping("/{userId}/{bookId}")
    public ResponseEntity<String> borrowBook(
            @PathVariable long userId,
            @PathVariable long bookId) {

        // Le contrôleur est très simple ! Il délègue tout à la façade.
        boolean success = borrowingFacade.borrowBook(userId, bookId);

        if (success) {
            return ResponseEntity.ok("Livre emprunté avec succès.");
        } else {
            return ResponseEntity.badRequest().body("L'emprunt a échoué.");
        }
    }
}
```

##### 5. La requête HTTP

```http
### Emprunter un livre via la façade

POST http://localhost:8080/api/borrowing/101/1
```

En exécutant cette requête, vous verrez dans la console de votre application la séquence d'appels orchestrée par la
façade.

## Pour aller plus loin

### Le Pattern Proxy

* **Intention :** Fournir un substitut ou un placeholder pour un autre objet afin de contrôler l'accès à celui-ci.
* **Analogie :** Une carte de crédit est un proxy pour votre argent en banque. Elle vous donne accès à votre argent mais
  avec un contrôle (limites, code PIN...).
* **Types courants :**
    * **Proxy de protection :** Contrôle l'accès à un objet en fonction des droits. Ex: un service accessible uniquement
      aux administrateurs.
    * **Proxy virtuel :** Retarde la création d'un objet coûteux jusqu'à ce qu'il soit réellement nécessaire (lazy
      loading).
    * **Proxy distant :** Représente un objet qui se trouve dans un autre espace d'adressage (sur un autre serveur).
* **Relation avec Spring :** Spring AOP (Programmation Orientée Aspect) utilise massivement les proxies ! Quand vous
  mettez `@Transactional` sur une méthode, Spring crée un proxy autour de votre bean. Ce proxy démarre une transaction
  avant l'appel de votre méthode et la commit (ou la rollback) après. Vous n'écrivez que la logique métier, le proxy
  gère l'aspect technique.

### Le Pattern Composite

* **Intention :** Composer des objets en structures arborescentes pour représenter des hiérarchies "partie-tout". Le
  Composite permet aux clients de traiter les objets individuels et les compositions d'objets de manière uniforme.
* **Analogie :** Un dossier dans un système de fichiers. Un dossier peut contenir des fichiers (feuilles) mais aussi
  d'autres dossiers (composites), qui à leur tour contiennent des fichiers et des dossiers. Que ce soit un fichier ou un
  dossier, vous pouvez appliquer la même opération `supprimer()` ou `renommer()`.
* **Structure :** Nécessite une interface commune (`Component`) implémentée à la fois par les objets "feuilles" (`Leaf`)
  et les conteneurs (`Composite`).

## Auto-évaluation

Vérifiez votre compréhension des patterns de structure.

<warning>
Les corrections de cette auto-évaluation se trouvent à la fin de l'ensemble du support de cours.
</warning>

**Questions à Choix Multiple (QCM)**

1. Vous devez intégrer une vieille API qui communique en SOAP (XML) dans votre application moderne qui utilise des
   objets REST/JSON. Quel pattern est le plus approprié ?
    * a) Decorator
    * b) Facade
    * c) Adapter
    * d) Proxy

2. Vous voulez ajouter une fonctionnalité de logging et de mise en cache à votre `BookService` sans modifier son code
   source. Quel pattern est idéal pour cela ?
    * a) Adapter
    * b) Decorator
    * c) Facade
    * d) Composite

3. Quel est l'objectif principal du pattern Facade ?
    * a) Ajouter des fonctionnalités à un objet existant.
    * b) Rendre compatibles deux interfaces différentes.
    * c) Fournir une interface simplifiée pour un système complexe.
    * d) Traiter un groupe d'objets de la même manière qu'un seul objet.

4. La principale différence entre l'Adapter et le Decorator est que :
    * a) L'Adapter change l'interface d'un objet, tandis que le Decorator ne la change pas.
    * b) Le Decorator change l'interface d'un objet, tandis que l'Adapter ne la change pas.
    * c) L'Adapter ne peut être utilisé qu'avec de vieilles classes.
    * d) Le Decorator est plus complexe à implémenter.

**Questions Ouvertes**

5. Expliquez comment le pattern Decorator applique le Principe Ouvert/Fermé.
6. Donnez un exemple concret où vous pourriez hésiter entre utiliser une Facade et simplement refactoriser le
   sous-système lui-même. Quels facteurs influenceraient votre décision ?

## Conclusion

Excellent travail ! Vous avez maintenant les outils pour structurer vos applications de manière logique et robuste.

Vous avez appris à :

* Faire le pont entre des mondes incompatibles avec l'**Adapter**.
* Enrichir vos objets à la carte et à l'infini avec le **Decorator**.
* Masquer la complexité et simplifier la vie de vos clients avec la **Facade**.

Ces patterns de structure sont essentiels pour maîtriser le couplage dans vos applications. Ils vous permettent de
construire des systèmes où les composants sont bien organisés, faciles à remplacer et à étendre.

Nous savons maintenant créer des objets et les assembler. La dernière pièce du puzzle est de définir comment ils
interagissent et communiquent entre eux. C'est l'objet de notre prochain et dernier grand chapitre : les **Patterns
Comportementaux**.