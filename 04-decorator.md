# Le Pattern Decorator : Ajouter des fonctionnalités dynamiquement à un objet

## Le problème : l'explosion des sous-classes

Notre système de véhicules fonctionne bien. Nous avons une interface `VehicleInterface` et une classe `Car`. Maintenant, nous voulons proposer des options pour nos voitures. Par exemple :
-   Une voiture avec un turbo.
-   Une voiture avec un système de navigation GPS avancé.
-   Une voiture avec un turbo ET un GPS.
-   Une voiture avec un blindage.
-   Une voiture avec un blindage ET un turbo...

La première idée qui vient à l'esprit est d'utiliser l'héritage :
-   `class CarWithTurbo extends Car`
-   `class CarWithGps extends Car`
-   `class CarWithTurboAndGps extends CarWithTurbo`

Cette approche est une très mauvaise idée et mène à une **explosion de classes**. Pour seulement 3 options, nous aurions besoin de 7 classes différentes pour couvrir toutes les combinaisons ! C'est impossible à maintenir. De plus, cette approche est **statique** : une voiture est créée avec un turbo et le gardera pour toujours. On ne peut pas ajouter ou retirer une option dynamiquement.

Comment ajouter des responsabilités à un objet de manière flexible et sans créer une multitude de sous-classes ?

## La solution : "habiller" l'objet avec des décorateurs

Le **Pattern Decorator** (ou **Décorateur**) résout ce problème en nous permettant d'envelopper un objet dans d'autres objets "décorateurs". Imaginez des poupées russes : chaque poupée ajoute une "couche" à la poupée qu'elle contient.

Le principe est le suivant :
1.  Les décorateurs et l'objet de base partagent la **même interface**. Notre système pourra donc les manipuler de la même manière.
2.  Un décorateur "a un" objet de cette même interface (il l'enveloppe).
3.  Quand on appelle une méthode sur le décorateur, celui-ci **ajoute son propre comportement** (avant ou après) puis **délègue** l'appel à l'objet qu'il enveloppe.

Cela nous permet d'empiler les décorateurs les uns sur les autres pour créer la combinaison de fonctionnalités souhaitée.

### Mettre en place le Pattern Decorator

#### 1. L'interface commune (Component)

C'est le contrat que tous les objets (de base et décorateurs) doivent respecter.

```php
interface VehicleInterface {
    public function start(): string;
    public function getPrice(): int;
}
```

#### 2. La classe de base (Concrete Component)

C'est l'objet nu que nous allons décorer.

```php
class BasicCar implements VehicleInterface {
    public function start(): string {
        return "Le moteur de la voiture démarre.";
    }

    public function getPrice(): int {
        return 20000; // Prix de base
    }
}
```

#### 3. Le Decorator de base (Abstract Decorator)

C'est une classe abstraite qui simplifie la création des décorateurs concrets. Elle implémente l'interface et contient l'objet enveloppé.

```php
abstract class VehicleDecorator implements VehicleInterface {
    protected VehicleInterface $vehicle;

    public function __construct(VehicleInterface $vehicle) {
        $this->vehicle = $vehicle;
    }

    // Le décorateur délègue simplement l'appel à l'objet enveloppé.
    // Les décorateurs concrets surchargeront cette méthode pour ajouter leur logique.
    public function start(): string {
        return $this->vehicle->start();
    }

    public function getPrice(): int {
        return $this->vehicle->getPrice();
    }
}
```

#### 4. Les Decorators concrets (Concrete Decorators)

Ce sont les "options" que nous allons ajouter. Chaque décorateur hérite du `VehicleDecorator`.

```php
// Fichier: decorators/TurboDecorator.php
class TurboDecorator extends VehicleDecorator {
    public function start(): string {
        // Ajoute son comportement APRÈS l'appel à l'objet parent
        return $this->vehicle->start() . " | Turbo activé !";
    }

    public function getPrice(): int {
        // Ajoute son coût au prix de l'objet enveloppé
        return $this->vehicle->getPrice() + 5000;
    }
}

// Fichier: decorators/GpsDecorator.php
class GpsDecorator extends VehicleDecorator {
    public function start(): string {
        // Ajoute son comportement AVANT l'appel à l'objet parent
        $gpsBoot = "Initialisation du GPS... ";
        return $gpsBoot . $this->vehicle->start();
    }

    public function getPrice(): int {
        return $this->vehicle->getPrice() + 1000;
    }
}```

#### Utilisation des Decorators

Maintenant, nous pouvons assembler notre voiture avec les options que nous voulons, de manière dynamique.

```php
// Fichier: index.php

// 1. Une voiture de base
$simpleCar = new BasicCar();
echo "Voiture simple : " . $simpleCar->start() . " | Prix : " . $simpleCar->getPrice() . "€\n";

// 2. Une voiture avec un turbo
// On "habille" la voiture de base avec le décorateur Turbo
$turboCar = new TurboDecorator(new BasicCar());
echo "Voiture Turbo : " . $turboCar->start() . " | Prix : " . $turboCar->getPrice() . "€\n";

// 3. Une voiture avec un GPS et un Turbo !
// On empile les décorateurs. L'ordre compte !
$fullOptionCar = new TurboDecorator(new GpsDecorator(new BasicCar()));
echo "Voiture Full Option : " . $fullOptionCar->start() . " | Prix : " . $fullOptionCar->getPrice() . "€\n";
```

**Résultat :**
```
Voiture simple : Le moteur de la voiture démarre. | Prix : 20000€
Voiture Turbo : Le moteur de la voiture démarre. | Turbo activé ! | Prix : 25000€
Voiture Full Option : Initialisation du GPS... Le moteur de la voiture démarre. | Turbo activé ! | Prix : 26000€
```

## Avantages du Pattern Decorator

-   **Flexibilité extrême** : Permet d'ajouter des fonctionnalités à des objets individuels de manière dynamique, sans affecter les autres objets.
-   **Respect du Principe Ouvert/Fermé** : On peut introduire de nouvelles fonctionnalités (de nouveaux décorateurs) sans jamais modifier le code des classes existantes.
-   **Évite l'explosion de sous-classes** : Remplace l'héritage pour l'ajout de responsabilités.
-   **Responsabilités décomposées** : Chaque décorateur a une seule et petite responsabilité (ajouter un turbo, ajouter un GPS).