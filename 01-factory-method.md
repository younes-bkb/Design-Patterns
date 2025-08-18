# Le Pattern Factory : Créer des objets sans connaître leur classe exacte

## Le problème : une création d'objets rigide et éparpillée

Imaginez une application de logistique qui doit créer différents types de véhicules. Sans un pattern spécifique, votre code pourrait rapidement ressembler à une longue suite de conditions `if/elseif` ou `switch` :

```php
// Fichier : index.php

$orderType = 'truck'; // Vient d'une base de données ou d'un formulaire
$vehicle = null;

if ($orderType === 'car') {
    $vehicle = new Car("Peugeot 208");
} elseif ($orderType === 'truck') {
    $vehicle = new Truck("Volvo FH");
} // ... et ainsi de suite pour chaque véhicule

if ($vehicle) {
    $vehicle->transport();
}
```

Ce code pose plusieurs problèmes :
1.  **Couplage Fort** : Le code principal est directement lié aux classes concrètes (`Car`, `Truck`). Il doit connaître le nom de chaque classe.
2.  **Violation du principe Ouvert/Fermé** : Pour ajouter un nouveau type de véhicule (ex: `Plane`), vous devez **modifier** ce bloc de conditions. Votre code n'est pas "fermé à la modification".
3.  **Duplication** : Si vous avez besoin de créer des véhicules à plusieurs endroits, vous devrez copier-coller cette logique partout.

## La solution : déléguer la création à une "usine"

Le **Pattern Factory** (ou **Fabrique**) résout ce problème en introduisant une classe dont la seule responsabilité est de **créer des objets**.

Au lieu de dire `new Car()` dans votre code principal, vous demandez à une "usine" (la Factory) : "Donne-moi un véhicule de type 'voiture'". L'usine se charge de la logique de création et vous retourne le bon objet.

Le code principal n'a plus besoin de connaître les classes `Car` ou `Truck`. Il a juste besoin de connaître la `VehicleFactory`.

### Mettre en place le Pattern Factory

Voici comment restructurer notre code avec une Factory.

#### 1. Créer une interface commune (le contrat)

C'est l'étape la plus importante. Elle garantit que tous les objets produits par notre usine auront les mêmes capacités.

```php
// Fichier: contracts/TransportInterface.php

interface TransportInterface {
    public function transport(): string;
}
```

#### 2. Créer les classes concrètes (les produits)

Chaque classe implémente l'interface, respectant ainsi le contrat.

```php
// Fichier: classes/Car.php
class Car implements TransportInterface {
    public function transport(): string { return "La voiture roule sur la route."; }
}

// Fichier: classes/Truck.php
class Truck implements TransportInterface {
    public function transport(): string { return "Le camion transporte la marchandise sur l'autoroute."; }
}
```

#### 3. Créer la Factory (l'usine)

C'est le cœur du pattern. Au lieu d'un `switch`, nous utilisons ici une approche plus élégante et flexible : un **tableau de correspondance** (array map).

```php
// Fichier: factories/VehicleFactory.php

class VehicleFactory {
    // On associe un type (une chaîne de caractères) à une classe concrète.
    private array $vehicleMap = [
        'car' => Car::class,
        'truck' => Truck::class,
    ];

    /**
     * Crée un objet Véhicule en fonction du type demandé.
     * @param string $type
     * @return TransportInterface
     * @throws \InvalidArgumentException Si le type est inconnu.
     */
    public function create(string $type): TransportInterface {
        if (!array_key_exists($type, $this->vehicleMap)) {
            throw new \InvalidArgumentException("Type de véhicule inconnu : $type");
        }

        $className = $this->vehicleMap[$type];
        return new $className();
    }
}
```
Cette version est plus souple : pour ajouter un nouveau véhicule, il suffira de l'ajouter au tableau `$vehicleMap`, sans toucher à la logique de la méthode `create`.

#### 4. Utiliser la Factory (le client)

Notre code principal devient incroyablement simple et découplé.

```php
// Fichier: index.php

$factory = new VehicleFactory();

$orderTypes = ['car', 'truck', 'truck', 'car'];
$vehicles = [];

foreach ($orderTypes as $type) {
    // On demande à l'usine de nous fabriquer le bon objet.
    // Le client n'a aucune idée de si un 'new Car()' ou 'new Truck()' est appelé.
    $vehicles[] = $factory->create($type);
}

// Grâce à l'interface, on sait que chaque objet aura une méthode transport().
foreach ($vehicles as $vehicle) {
    echo $vehicle->transport() . PHP_EOL;
}
```

**Résultat :**
```
La voiture roule sur la route.
Le camion transporte la marchandise sur l'autoroute.
Le camion transporte la marchandise sur l'autoroute.
La voiture roule sur la route.
```

## Avantages du Pattern Factory

-   **Découplage** : Votre code client n'est plus lié aux classes concrètes. Il ne dépend que de l'interface et de la factory.
-   **Centralisation** : Toute la logique de création est regroupée à un seul endroit.
-   **Extensibilité** : Pour ajouter un `Boat`, il suffit de créer la classe `Boat` (qui implémente `TransportInterface`) et de l'ajouter au tableau `$vehicleMap` de la factory. **Aucune autre partie du code n'a besoin d'être modifiée**.


Le Pattern Factory est un premier pas essentiel vers un code plus souple, maintenable et respectant les grands principes de la conception logicielle.
