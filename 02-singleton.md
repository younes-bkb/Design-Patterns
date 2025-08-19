
# Le Pattern Singleton : Assurer une instance unique d'une classe

## Le problème : des ressources uniques et un état global

Dans notre application de gestion de véhicules, imaginons que nous ayons besoin d'un **registre d'immatriculation central**. Chaque fois qu'un nouveau véhicule est créé, il doit y être enregistré.

Il est crucial qu'il n'existe **qu'un seul et unique registre** pour toute l'application. Si différentes parties du code pouvaient créer leur propre registre (`new VehicleRegistry()`), nous aurions plusieurs listes d'immatriculations parallèles, ce qui mènerait au chaos :
-   Des numéros de plaques pourraient être attribués en double.
-   On ne pourrait jamais avoir la liste complète et fiable de tous les véhicules en circulation.

Comment forcer la classe `VehicleRegistry` à n'avoir **qu'une seule et unique instance** ?

## La solution : une classe qui gère sa propre instance

Le **Pattern Singleton** répond parfaitement à ce besoin. Une classe Singleton est une classe qui :
1.  S'assure qu'elle n'a qu'une seule instance d'elle-même.
2.  Fournit un point d'accès global (une méthode statique) pour récupérer cette unique instance.

La classe `VehicleRegistry` devient elle-même la gardienne de sa propre unicité.

### Mettre en place le Pattern Singleton

La mise en place d'un Singleton repose sur trois piliers :

1.  **Un constructeur privé (`private __construct`)** : Cela empêche la création d'instances depuis l'extérieur avec le mot-clé `new`.
2.  **Une propriété statique privée (`private static $instance`)** : Cette variable stockera l'unique instance de la classe.
3.  **Une méthode statique publique (`public static function getInstance()`)** : C'est le seul point d'accès pour obtenir l'instance.

#### Exemple : Un registre d'immatriculation de véhicules

```php
// Fichier: services/VehicleRegistry.php

class VehicleRegistry {
    // 2. La propriété qui stockera notre unique instance.
    private static ?self $instance = null;

    // Un tableau pour stocker les ID de nos véhicules
    private array $registeredVehicles = [];

    // 1. Le constructeur est privé ! Personne ne peut faire "new VehicleRegistry()".
    private function __construct() {
        // Simule l'initialisation du service de registre.
        echo "Service d'immatriculation initialisé.\n";
    }

    // 3. La méthode statique qui sert de point d'accès global.
    public static function getInstance(): self {
        // Si aucune instance n'a été créée...
        if (self::$instance === null) {
            // ... alors on la crée. C'est le SEUL endroit où `new` est appelé.
            self::$instance = new self();
        }

        // On retourne l'instance unique.
        return self::$instance;
    }

    // Méthodes utilitaires pour notre registre
    public function registerVehicle(string $vehicleId): void {
        if (!in_array($vehicleId, $this->registeredVehicles)) {
            $this->registeredVehicles[] = $vehicleId;
            echo "Véhicule {$vehicleId} immatriculé.\n";
        }
    }

    public function getRegisteredVehicles(): array {
        return $this->registeredVehicles;
    }

    // Pour empêcher le clonage de l'instance (sécurité supplémentaire)
    private function __clone() {}
}
```

#### Utilisation du Singleton

```php
// Fichier: index.php

// Dans une partie du code, on immatricule une voiture.
// On récupère l'instance. Le message "Service initialisé" s'affiche.
$registry1 = VehicleRegistry::getInstance();
$registry1->registerVehicle('CAR-123-AB');

// Ailleurs dans l'application, on immatricule un camion.
// Le message "Service initialisé" ne s'affiche PAS une seconde fois !
$registry2 = VehicleRegistry::getInstance();
$registry2->registerVehicle('TRUCK-456-CD');

// Prouvons que nous travaillons bien avec la même instance.
// Si on affiche la liste depuis la première variable, elle contient aussi le camion !
echo "Véhicules immatriculés (via registry1) : \n";
print_r($registry1->getRegisteredVehicles());

// On peut aussi vérifier que les deux variables pointent vers le MÊME objet
if ($registry1 === $registry2) {
    echo "registry1 et registry2 sont la même instance.\n";
}

// Essayer de créer un nouveau registre provoquera une erreur fatale
// $errorRegistry = new VehicleRegistry(); // PHP Fatal error: Uncaught Error: Call to private VehicleRegistry::__construct()
```
**Résultat :**
```
Service d'immatriculation initialisé.
Véhicule CAR-123-AB immatriculé.
Véhicule TRUCK-456-CD immatriculé.
Véhicules immatriculés (via registry1) : 
Array
(
    => CAR-123-AB
    => TRUCK-456-CD
)
registry1 et registry2 sont la même instance.
```

## Les Dangers du Singleton (À utiliser avec prudence)

Bien qu'utile, le Singleton est l'un des patterns les plus controversés car il peut être considéré comme un **anti-pattern** s'il est mal utilisé.

-   **Il introduit un état global** : Votre instance unique est accessible de n'importe où, ce qui rend le suivi des modifications de données difficile.
-   **Il cache les dépendances** : Une classe qui appelle `VehicleRegistry::getInstance()` à l'intérieur de ses méthodes cache le fait qu'elle dépend du registre. Il est préférable d'injecter les dépendances.
-   **Il complique les tests unitaires** : Comme l'instance est unique et globale, il est très difficile de la remplacer par un "faux" objet (un *mock*) pour isoler et tester un composant.

**Règle générale :** Avant d'utiliser un Singleton, demandez-vous s'il n'existe pas une meilleure solution, comme **l'injection de dépendances**.