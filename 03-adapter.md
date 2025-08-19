# Le Pattern Adapter : Faire collaborer des interfaces incompatibles

## Le problème : des interfaces qui ne "parlent pas la même langue"

Imaginez que notre système de gestion de flotte est bien établi. Tous nos objets (`Car`, `Truck`, etc.) implémentent une interface `VehicleInterface` qui possède une méthode `drive()`. Notre code principal peut donc parcourir la flotte et appeler `drive()` sur chaque véhicule sans se poser de questions.

Maintenant, nous devons intégrer un nouveau type de machine : un drone de livraison ultra-moderne. Le problème ? Nous l'avons acheté à une société externe, et sa classe `SuperDrone` n'a pas de méthode `drive()`. À la place, sa méthode pour se déplacer s'appelle `fly()`.

```php
// Notre système attend des objets comme ça :
interface VehicleInterface {
    public function drive(): string;
}

// Mais la classe que l'on doit intégrer ressemble à ça :
class SuperDrone {
    public function takeOff(): void { /* ... */ }
    public function fly(): string {
        return "Le drone vole dans les airs.";
    }
    public function land(): void { /* ... */ }
}```

Notre code principal, qui fait `$vehicle->drive()`, va planter s'il reçoit un objet `SuperDrone`. Nous ne pouvons pas (ou ne voulons pas) modifier le code de la classe `SuperDrone` car elle vient d'une bibliothèque externe.

Comment faire pour que le drone soit "compris" par notre système comme un `Vehicle` ?

## La solution : créer un "traducteur"

Le **Pattern Adapter** (ou **Adaptateur**) résout ce problème en créant une classe intermédiaire qui sert de "traducteur" ou d'"adaptateur" (comme un adaptateur de prise électrique).

Cet adaptateur va :
1.  Se présenter à notre système comme un objet compatible (il implémentera `VehicleInterface`).
2.  "Envelopper" l'objet incompatible (le `SuperDrone`).
3.  Quand notre système appellera la méthode `drive()` sur l'adaptateur, celui-ci, en interne, traduira cet appel et exécutera la méthode `fly()` de l'objet `SuperDrone`.

### Mettre en place le Pattern Adapter

#### 1. L'interface cible (Target)

C'est l'interface que notre système client attend. Nous l'avons déjà.

```php
interface VehicleInterface {
    public function drive(): string;
}
```
Nos classes habituelles (`Car`, `Truck`) implémentent cette interface.
```php
class Car implements VehicleInterface {
    public function drive(): string {
        return "La voiture roule sur la route.";
    }
}
```

#### 2. La classe inadaptée (Adaptee)

C'est la classe externe que nous voulons utiliser, mais qui a une interface incompatible.

```php
class SuperDrone {
    public function fly(): string {
        return "Le drone de livraison vole vers sa destination.";
    }
}
```

#### 3. L'Adapter

C'est la classe qui fait le lien entre les deux. Elle "adapte" le `SuperDrone` pour qu'il soit vu comme un `VehicleInterface`.

```php
// Fichier: adapters/DroneAdapter.php

class DroneAdapter implements VehicleInterface {
    private SuperDrone $drone;

    // L'adaptateur prend l'objet inadapté dans son constructeur
    public function __construct(SuperDrone $drone) {
        $this->drone = $drone;
    }

    // On implémente la méthode attendue par le système...
    public function drive(): string {
        // ... et on la "traduit" en un appel à la méthode de l'objet inadapté.
        return $this->drone->fly();
    }
}
```

#### Utilisation de l'Adapter

Le code client manipule tous les objets via l'interface `VehicleInterface`, sans jamais se soucier des détails d'implémentation.

```php
// Fichier: index.php

$fleet = [];
$fleet[] = new Car(); // Un objet standard

// On crée l'objet incompatible...
$drone = new SuperDrone();
// ...puis on l'enveloppe dans son adaptateur.
$adaptedDrone = new DroneAdapter($drone);

// On ajoute l'ADAPTATEUR à notre flotte, pas le drone directement !
$fleet[] = $adaptedDrone;

// Notre code principal fonctionne sans aucune modification !
foreach ($fleet as $vehicle) {
    // Il appelle drive() sur la voiture ET sur l'adaptateur de drone.
    echo $vehicle->drive() . PHP_EOL;
}
```
**Résultat :**
```
La voiture roule sur la route.
Le drone de livraison vole vers sa destination.
```

Le système a "conduit" le drone, sans même savoir que c'était un drone qui volait !

## Avantages du Pattern Adapter

-   **Intégration transparente** : Permet de faire fonctionner ensemble des classes qui n'ont pas été conçues pour, sans modifier leur code source.
-   **Respect du Principe de Responsabilité Unique** : La logique métier reste dans les classes d'origine. L'adaptateur n'a qu'une seule responsabilité : traduire l'interface.
-   **Découplage** : Le code client ne dépend que de l'interface cible, pas des implémentations concrètes (ni de l'adaptateur, ni de l'inadapté).