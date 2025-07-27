# Hors-Série : Le Musée des Horreurs - Les Anti-Patterns

## Objectifs Pédagogiques

À la fin de ce module, vous serez capable de :

* **Définir** ce qu'est un "Anti-Pattern".
* **Identifier** plusieurs anti-patterns courants au niveau du code, de l'architecture et du processus.
* **Expliquer** pourquoi ces anti-patterns sont néfastes pour la maintenabilité, la testabilité et l'évolution d'un
  projet.
* **Associer** chaque anti-pattern à un principe de bonne conception (comme S.O.L.I.D. ou D.R.Y.) ou à un pattern
  correctif.
* **Développer** un esprit critique lors des revues de code pour repérer et corriger ces mauvaises pratiques.

## Introduction : Naviguer dans le Champ de Mines

Vous avez maintenant une excellente boîte à outils de Design Patterns. Mais sur le terrain, vous rencontrerez des
projets qui n'ont pas suivi ces plans. Ces projets sont souvent des champs de mines, truffés de solutions qui semblaient
une bonne idée sur le moment, mais qui se sont révélées être des pièges coûteux à long terme.

Ces pièges ont un nom : les **Anti-Patterns**. Un anti-pattern n'est pas simplement du "mauvais code". C'est une *
*solution couramment utilisée à un problème, mais qui est contre-productive**. C'est une fausse bonne idée, une ornière
dans laquelle de nombreux développeurs sont tombés avant vous.

Apprendre à les reconnaître, c'est comme avoir une carte du champ de mines. Cela vous protégera, vous et votre équipe,
de décisions qui peuvent paralyser un projet.

## L'essentiel : Catalogue des Erreurs Courantes

Nous allons classer ces anti-patterns par portée, du plus local (code) au plus global (architecture).

### Anti-Patterns de Développement

Ce sont les erreurs que l'on commet au quotidien en écrivant du code.

<tabs>
<tab title="Spaghetti Code (Code Spaghetti)">
    <p><b>Description :</b> Un code sans structure claire, où le flux d'exécution saute partout, comme un plat de spaghettis emmêlés. Les méthodes sont longues, les dépendances sont croisées et il est presque impossible de suivre la logique. </p>
    <p><b>Symptômes :</b> Faible cohésion, fort couplage, utilisation abusive des variables globales, structures de contrôle complexes et imbriquées. </p>
    <p><b>Conséquences :</b> Maintenance impossible, débuggage cauchemardesque, le moindre changement a des effets de bord imprévisibles.</p>
    <p><b>Remède :</b> Refactoring ! Appliquer des principes comme <b>S.O.L.I.D.</b>, extraire des logiques dans des classes et méthodes plus petites, et utiliser des Design Patterns (<b>Strategy</b>, <b>Command</b>, <b>Facade</b>...) pour clarifier les responsabilités.</p>
</tab>
<tab title="God Object / God Class (Objet Divin)">
    <p><b>Description :</b> Une classe qui concentre une quantité énorme de responsabilités non liées entre elles. Elle sait tout et fait tout. (Ex: une classe <code>Manager</code> qui gère les utilisateurs, les produits, les commandes et les paiements).</p>
    <p><b>Symptômes :</b> Fichiers de plusieurs milliers de lignes, des dizaines de dépendances injectées, une liste de méthodes interminable.</p>
    <p><b>Conséquences :</b> Violation flagrante du <b>Principe de Responsabilité Unique (SRP)</b>. La classe est impossible à tester, à comprendre et à maintenir. Tout changement est risqué.</p>
    <p><b>Remède :</b> Décomposer la classe en plusieurs classes plus petites et cohésives, chacune avec une seule responsabilité (ex: <code>UserManager</code>, <code>ProductService</code>, etc.).</p>
</tab>
<tab title="Copy-Paste Programming (Programmation par Copier-Coller)">
    <p><b>Description :</b> Un développeur a besoin d'une fonctionnalité similaire à une autre. Au lieu de créer une abstraction réutilisable, il copie-colle le bloc de code et le modifie légèrement.</p>
    <p><b>Symptômes :</b> Blocs de code dupliqués à plusieurs endroits de l'application.</p>
    <p><b>Conséquences :</b> Violation du principe <b>D.R.Y. (Don't Repeat Yourself)</b>. Si un bug est découvert dans le code original, il faut penser à le corriger dans toutes les copies. La maintenance devient un cauchemar.</p>
    <p><b>Remède :</b> Extraire le code dupliqué dans une méthode ou une classe commune. Utiliser des patterns comme le <b>Template Method</b> ou <b>Strategy</b> pour gérer les variations.</p>
</tab>
<tab title="Magic Numbers / Hardcoded Strings (Nombres Magiques / Chaînes en dur)">
    <p><b>Description :</b> Utiliser des nombres ou des chaînes de caractères littérales directement dans le code, sans explication.</p>
    <p><b>Exemple :</b> <code>if (user.getStatus() == 2) { ... }</code> ou <code>db.connect("jdbc:mysql://192.168.1.50/prod_db")</code>.</p>
    <p><b>Conséquences :</b> Le code est illisible (que signifie `2` ?). Si la valeur doit changer, il faut la chercher et la remplacer partout (risque d'oubli).</p>
    <p><b>Remède :</b> Remplacer les nombres magiques par des **constantes nommées** (ex: <code>public static final int STATUS_ACTIVE = 2;</code>) ou des **énumérations (`enum`)**. Externaliser les chaînes de configuration dans des **fichiers de propriétés**.</p>
</tab>
</tabs>

### Anti-Patterns d'Architecture

Ce sont des erreurs de conception à plus grande échelle.

<tabs>
<tab title="Singletonitis (La Singleton-ite)">
    <p><b>Description :</b> L'abus du pattern Singleton pour tout et n'importe quoi, souvent par paresse pour éviter de gérer les dépendances.</p>
    <p><b>Symptômes :</b> Utilisation massive d'appels statiques comme <code>MyManager.getInstance().doSomething()</code>.</p>
    <p><b>Conséquences :</b> Crée un état global caché, rend les tests unitaires très difficiles (comment mocker un appel statique ?), et crée un couplage fort dans toute l'application.</p>
    <p><b>Remède :</b> Le pattern <b>Dependency Injection (DI)</b>. Au lieu que la classe aille chercher sa dépendance, la dépendance lui est fournie de l'extérieur. C'est le cœur des frameworks comme Spring.</p>
</tab>
<tab title="Leaky Abstraction (Abstraction qui fuit)">
    <p><b>Description :</b> Une abstraction est censée cacher les détails d'implémentation, mais elle force l'utilisateur à connaître ces détails pour fonctionner correctement.</p>
    <p><b>Exemple :</b> Une méthode <code>getUsers()</code> qui retourne une <code>List&lt;User&gt;</code>, mais si le client essaie de l'utiliser sans savoir que derrière se cache une session Hibernate ouverte, il obtient une <code>LazyInitializationException</code>. L'abstraction "liste d'utilisateurs" a laissé fuiter un détail d'implémentation de l'ORM.</p>
    <p><b>Conséquences :</b> L'abstraction est inutile, voire dangereuse. Elle ne simplifie pas les choses et crée des bugs difficiles à comprendre.</p>
    <p><b>Remède :</b> Concevoir des abstractions plus robustes. Dans l'exemple, la solution serait de s'assurer que les données nécessaires sont chargées avant de fermer la session, ou d'utiliser un <b>DTO</b> qui est déconnecté de la session ORM.</p>
</tab>
<tab title="Vendor Lock-In (Couplage à la Technologie)">
    <p><b>Description :</b> Concevoir une application de telle manière qu'elle est excessivement dépendante d'un fournisseur ou d'une technologie spécifique, rendant une future migration quasi impossible ou très coûteuse.</p>
    <p><b>Exemple :</b> Utiliser massivement des fonctionnalités propriétaires d'une base de données Oracle au lieu du SQL standard, ou construire toute sa logique métier autour des fonctions spécifiques d'un service cloud (ex: AWS Lambda, Azure Functions).</p>
    <p><b>Conséquences :</b> Perte de pouvoir de négociation, coûts élevés, incapacité à adopter de meilleures technologies à l'avenir.</p>
    <p><b>Remède :</b> Utiliser des standards et des abstractions. Préférer JPA (standard) à l'API native de Hibernate. Utiliser le pattern <b>Adapter</b> ou <b>Facade</b> pour isoler le code qui interagit avec des services externes, afin de pouvoir les remplacer plus facilement.</p>
</tab>
</tabs>

### Exercice 14 : La Revue de Code de l'Horreur

Vous êtes chargé de faire la revue de code d'un collègue sur une nouvelle fonctionnalité. Voici la classe qu'il a
produite. Votre mission est d'agir en tant que "Quality Gate" (barrière de qualité).

```java
// Classe pour traiter les commandes clients
public class OrderProcessor {

    public void processOrder(int orderId) {
        // 1. Récupération de la commande
        System.out.println("Connexion à jdbc:mysql://server1/db...");
        // ... code JDBC pour récupérer la commande avec l'id `orderId`
        // Supposons qu'on récupère un statut et un email client
        int status = 2; // Statut "PAYÉE"
        String clientEmail = "test@test.com";

        // 2. Traitement en fonction du statut
        if (status == 1) { // 1 = "CRÉÉE"
            System.out.println("Commande pas encore payée.");
        } else if (status == 2) { // 2 = "PAYÉE"
            System.out.println("La commande est payée, préparation de l'envoi.");
            // Logique de préparation...

            // 3. Envoi d'une notification par email
            // Copié-collé d'une autre classe
            try {
                // Configuration du serveur mail
                Properties props = new Properties();
                props.put("mail.smtp.host", "smtp.myprovider.com");
                // ... autres propriétés

                // Envoi...
                System.out.println("Envoi d'un email de confirmation à " + clientEmail);
            } catch (Exception e) {
                // gestion d'erreur...
            }
        } else if (status == 3) { // 3 = "EXPÉDIÉE"
            System.out.println("Commande déjà expédiée.");
        }
    }
}
```

**Votre mission :**
Identifiez au moins 4 anti-patterns dans ce code. Pour chacun, nommez-le, expliquez pourquoi c'est un problème, et
proposez le remède approprié.

### Correction exercice 14 {collapsible='true''}

Excellent travail de détective ! Cette classe est un véritable cas d'école. Voici une analyse possible :

1. **Anti-Pattern :** **God Method / Violation du SRP**.
    * **Problème :** La méthode `processOrder` fait tout : elle se connecte à la base de données, elle contient la
      logique métier de traitement de la commande, et elle gère l'envoi de notifications par email. C'est beaucoup trop
      de responsabilités.
    * **Remède :** Décomposer la méthode en plusieurs classes/services distincts, injectés via **Dependency Injection
      ** : un `OrderRepository` pour l'accès aux données, un `OrderProcessingService` pour la logique métier, et un
      `NotificationService` pour les emails.

2. **Anti-Pattern :** **Magic Numbers (Nombres Magiques)**.
    * **Problème :** Le code est rempli de `if (status == 1)`, `if (status == 2)`. Un nouveau développeur (ou même vous
      dans 6 mois) n'aura aucune idée de ce que ces chiffres signifient sans chercher dans la documentation ou la base
      de données.
    * **Remède :** Utiliser une **énumération (`enum`)** `OrderStatus { CREATED, PAID, SHIPPED }`. Le code deviendrait
      alors `if (status == OrderStatus.PAID)`, ce qui est infiniment plus lisible et sûr.

3. **Anti-Pattern :** **Hardcoded Strings (Chaînes en dur)**.
    * **Problème :** La chaîne de connexion JDBC (`"jdbc:mysql://server1/db..."`) et la configuration du serveur SMTP (
      `"smtp.myprovider.com"`) sont écrites directement dans le code. Si le serveur change, il faut recompiler et
      redéployer l'application. C'est une catastrophe en termes de maintenance.
    * **Remède :** Externaliser ces valeurs dans un **fichier de configuration** (ex: `application.properties` dans
      Spring) et les injecter dans les beans appropriés.

4. **Anti-Pattern :** **Copy-Paste Programming**.
    * **Problème :** Le commentaire indique que la logique d'envoi d'email a été copiée d'une autre classe. Si la
      configuration du serveur SMTP change ou si un bug est trouvé dans ce bloc, il y a un risque élevé qu'on oublie de
      le corriger dans l'un des deux endroits.
    * **Remède :** Appliquer le principe **D.R.Y.** en créant une classe `NotificationService` (ou `EmailService`)
      centralisée, qui contiendra cette logique une seule et unique fois. Les autres classes utiliseront ce service.

En appliquant ces corrections, on transformerait ce code fragile et opaque en une structure modulaire, lisible et
maintenable, typique d'une application professionnelle.

## Auto-évaluation

Testez votre capacité à repérer les mauvaises odeurs de code.

**Questions à Choix Multiple (QCM)**

1. Votre collègue a écrit une classe `Utils` de 5000 lignes avec 200 méthodes statiques qui manipulent des chaînes, des
   dates, des fichiers et font des appels API. C'est un exemple flagrant de :
    * a) Spaghetti Code
    * b) God Object
    * c) Leaky Abstraction
    * d) Vendor Lock-In

2. Le principe "Don't Repeat Yourself" (D.R.Y.) est le remède direct à quel anti-pattern ?
    * a) Magic Numbers
    * b) Singletonitis
    * c) Copy-Paste Programming
    * d) God Object

3. L'utilisation intensive de `MyService.getInstance()` au lieu de l'injection de dépendances est un symptôme de :
    * a) Leaky Abstraction
    * b) Spaghetti Code
    * c) Vendor Lock-In
    * d) Singletonitis

4. Vous utilisez un ORM, mais vous êtes obligé de gérer manuellement l'ouverture et la fermeture des transactions dans
   votre couche métier, sinon vous obtenez des exceptions. C'est un exemple de :
    * a) God Object
    * b) Leaky Abstraction
    * c) Magic Numbers
    * d) Copy-Paste Programming

**Questions Ouvertes**

5. Expliquez pourquoi un "God Object" est si difficile à tester unitairement.
6. Donnez un exemple simple de "Magic Number" que vous pourriez rencontrer dans une application de e-commerce, et
   comment vous le corrigeriez.

## Conclusion

Félicitations ! Vous êtes maintenant non seulement un constructeur, mais aussi un inspecteur en bâtiment. Vous savez
reconnaître les fondations solides (les Design Patterns) et les murs fissurés (les Anti-Patterns).

Cette compétence est l'une des plus précieuses pour un concepteur. Elle vous permettra de :

* **Écrire du code propre** dès le départ.
* **Argumenter de manière constructive** lors des revues de code.
* **Estimer la "dette technique"** d'un projet existant.
* **Planifier des refactorings** efficaces pour améliorer la santé d'une application.

Le chemin vers la maîtrise de la conception logicielle est un apprentissage continu. Gardez l'esprit ouvert, restez
critique envers votre propre code, et n'oubliez jamais que le développeur qui lira votre code demain (qui sera souvent
une version future de vous-même) vous remerciera pour votre clarté et votre rigueur.

### Corrections de l'auto-évaluation (Anti-Patterns)

1. b)
2. c)
3. d)
4. b)
5. **Test d'un God Object :** Pour tester unitairement une méthode d'un God Object, il faut initialiser l'objet. Comme
   il a des dizaines de dépendances, il faut créer et configurer des "mocks" pour chacune d'entre elles, ce qui est
   extrêmement lourd. De plus, comme la classe a un état interne énorme et complexe, l'exécution d'une méthode peut
   avoir des effets de bord sur d'autres parties de l'objet, rendant le test d'une seule unité de comportement presque
   impossible.
6. **Exemple de Magic Number :** Dans une boucle qui traite les articles d'un panier, on pourrait voir
   `finalPrice += price * 0.20;`. Le `0.20` est un nombre magique. Que représente-t-il ? La TVA ? Une taxe spéciale ? *
   *Correction :** Créer une constante nommée `private static final double VAT_RATE = 0.20;`. Le code devient
   `finalPrice += price * VAT_RATE;`, ce qui est auto-documenté. Mieux encore, cette valeur devrait probablement venir
   d'un fichier de configuration ou d'une base de données.