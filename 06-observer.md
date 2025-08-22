# Le Pattern Observer : Notifier des objets d'un changement d'état

## Le problème : un couplage élevé et rigide

Imaginons que nous ajoutions un système de sécurité à notre voiture. Ce `CarSecuritySystem` a une responsabilité principale : détecter une intrusion. Lorsqu'une intrusion est détectée, plusieurs actions doivent se produire simultanément :
-   La sirène doit se déclencher.
-   Les phares doivent clignoter.
-   Un SMS doit être envoyé au propriétaire.

Une première approche, très naïve, serait de coder tout cela directement dans la classe `CarSecuritySystem` :

```php
class CarSecuritySystem {
    private Siren $siren;
    private Headlights $headlights;
    private SmsNotifier $smsNotifier;

    // ... constructeur pour initialiser les objets

    public function detectIntrusion() {
        echo "!!! INTRUSION DÉTECTÉE !!!\n";

        // Le système de sécurité est directement responsable de tout.
        $this->siren->sound();
        $this->headlights->flash();
        $this->smsNotifier->send("123-456-789", "Alerte : Intrusion détectée dans votre véhicule.");
    }
}
```

Cette approche pose un problème majeur de **couplage fort**. Le `CarSecuritySystem` connaît les classes concrètes `Siren`, `Headlights` et `SmsNotifier`. Que se passe-t-il si demain nous voulons également envoyer une notification par email ? Ou verrouiller les portes automatiquement ? Nous serions obligés de **modifier la classe `CarSecuritySystem`** à chaque fois. Cela viole le Principe Ouvert/Fermé.

Comment un objet (le sujet) peut-il notifier une liste d'autres objets (les observateurs) d'un changement d'état, sans savoir qui ils sont exactement ?

## La solution : un système d'abonnement

Le **Pattern Observer** (ou **Observateur**) propose une solution élégante. Pensez à un abonnement à un magazine :
1.  Le magazine (le **Sujet**) ne connaît pas personnellement ses milliers d'abonnés. Il a simplement une liste d'adresses.
2.  Les abonnés (les **Observateurs**) s'inscrivent pour recevoir les notifications.
3.  Quand un nouveau numéro sort (un changement d'état), l'éditeur (le Sujet) parcourt sa liste et envoie une copie à chaque abonné.

Le principe est le suivant :
1.  Le **Sujet** est l'objet qui contient l'état important. Il maintient une liste d'Observateurs.
2.  Il expose des méthodes pour s'abonner (`attach`) et se désabonner (`detach`).
3.  Lorsqu'un événement se produit, le Sujet parcourt sa liste et appelle une méthode de notification (`update`) sur chaque **Observateur**.

Les Observateurs sont ainsi découplés du Sujet. Le Sujet ne sait rien de ce qu'ils font, il se contente de les prévenir.

### Mettre en place le Pattern Observer

PHP fournit des interfaces natives (`SplSubject`, `SplObserver`) qui sont parfaites pour cela.

#### 1. Les Interfaces (Sujet et Observateur)

-   `SplSubject` : Demande à la classe d'implémenter `attach(SplObserver $o)`, `detach(SplObserver $o)`, et `notify()`.
-   `SplObserver` : Demande à la classe d'implémenter `update(SplSubject $subject)`.

#### 2. Le Sujet Concret (Concrete Subject)

C'est notre système de sécurité. Il implémente `SplSubject` et notifiera ses observateurs quand une intrusion est détectée.

```php
// Fichier: CarSecuritySystem.php (le Sujet)
class CarSecuritySystem implements SplSubject {
    private SplObjectStorage $observers;
    private bool $isIntrusionDetected = false;

    public function __construct() {
        $this->observers = new SplObjectStorage();
    }

    public function attach(SplObserver $observer): void {
        $this->observers->attach($observer);
    }

    public function detach(SplObserver $observer): void {
        $this->observers->detach($observer);
    }

    public function notify(): void {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }

    // La logique métier qui déclenche la notification
    public function detectIntrusion(): void {
        echo "Système : Détection d'une activité suspecte...\n";
        $this->isIntrusionDetected = true;
        $this->notify(); // On prévient tous les abonnés !
    }

    public function isIntrusionDetected(): bool {
        return $this->isIntrusionDetected;
    }
}
```

#### 3. Les Observateurs Concrets (Concrete Observers)

Ce sont les classes qui réagissent à la notification. Chacune implémente `SplObserver`.

```php
// Fichier: observers/Siren.php
class Siren implements SplObserver {
    public function update(SplSubject $subject): void {
        // L'observateur peut vérifier de quel sujet il s'agit
        if ($subject instanceof CarSecuritySystem && $subject->isIntrusionDetected()) {
            echo "Sirène : WEE-WOO-WEE-WOO !!!\n";
        }
    }
}

// Fichier: observers/HeadlightsNotifier.php
class HeadlightsNotifier implements SplObserver {
    public function update(SplSubject $subject): void {
        if ($subject instanceof CarSecuritySystem && $subject->isIntrusionDetected()) {
            echo "Phares : *CLIGNOTE* *CLIGNOTE*\n";
        }
    }
}

// Fichier: observers/SmsNotifier.php
class SmsNotifier implements SplObserver {
    public function update(SplSubject $subject): void {
        if ($subject instanceof CarSecuritySystem && $subject->isIntrusionDetected()) {
            echo "SMS : Envoi d'une alerte au propriétaire.\n";
        }
    }
}
```

#### Utilisation du Pattern Observer

Le code client assemble le tout. Il décide quels observateurs s'abonnent à quel sujet.

```php
// Fichier: index.php

// 1. On crée le sujet
$securitySystem = new CarSecuritySystem();

// 2. On crée plusieurs observateurs
$siren = new Siren();
$lights = new HeadlightsNotifier();
$sms = new SmsNotifier();

// 3. On abonne les observateurs au sujet
$securitySystem->attach($siren);
$securitySystem->attach($lights);
$securitySystem->attach($sms);

// 4. On déclenche l'événement. Le sujet va notifier tout le monde.
$securitySystem->detectIntrusion();

echo "\n--- Le propriétaire désactive la sirène à distance ---\n";
// On peut se désabonner à la volée !
$securitySystem->detach($siren);

// Un autre événement se produit (le voleur est toujours là)
$securitySystem->detectIntrusion();
```

**Résultat :**
```
Système : Détection d'une activité suspecte...
Sirène : WEE-WOO-WEE-WOO !!!
Phares : *CLIGNOTE* *CLIGNOTE*
SMS : Envoi d'une alerte au propriétaire.

--- Le propriétaire désactive la sirène à distance ---
Système : Détection d'une activité suspecte...
Phares : *CLIGNOTE* *CLIGNOTE*
SMS : Envoi d'une alerte au propriétaire.
```

## Avantages du Pattern Observer

-   **Découplage Élevé** : Le sujet ne connaît que l'interface de l'observateur. Il ignore totalement ce que font les observateurs concrets (`Siren`, `SmsNotifier`...). Ils sont interchangeables.
-   **Respect du Principe Ouvert/Fermé** : On peut introduire de nouveaux observateurs (ex: `EmailNotifier`) sans jamais modifier le code du sujet (`CarSecuritySystem`).
-   **Relations dynamiques** : Il est possible d'ajouter ou de retirer des observateurs à n'importe quel moment pendant l'exécution du programme.