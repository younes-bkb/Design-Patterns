# Le Pattern Strategy : Rendre des algorithmes interchangeables

## Le problème : des `if/else` à n'en plus finir

Notre voiture est maintenant équipée d'un système de navigation GPS. La première version de ce GPS a une seule mission : calculer l'itinéraire le plus rapide. Mais maintenant, nous voulons offrir plus d'options à nos utilisateurs :
-   Calculer l'itinéraire le plus **court** (moins de kilomètres).
-   Calculer l'itinéraire le plus **écologique** (qui consomme le moins de carburant).
-   Calculer l'itinéraire **touristique** (qui passe par des points d'intérêt).

Une approche instinctive serait d'utiliser une longue structure conditionnelle (`if/else` ou `switch`) à l'intérieur de notre classe `NavigationSystem`.

```php
class NavigationSystem {
    public function calculateRoute(string $origin, string $destination, string $mode) {
        if ($mode === 'fastest') {
            // Logique complexe pour trouver la route la plus rapide...
            echo "Calcul de la route la plus RAPIDE.\n";
        } elseif ($mode === 'shortest') {
            // Logique complexe pour trouver la route la plus courte...
            echo "Calcul de la route la plus COURTE.\n";
        } elseif ($mode === 'scenic') {
            // Logique complexe pour trouver la route touristique...
            echo "Calcul de la route TOURISTIQUE.\n";
        }
        // ...
    }
}
```

Cette approche est une très mauvaise idée. La méthode `calculateRoute` devient un monstre difficile à lire et à maintenir. Chaque fois que nous voudrons ajouter une nouvelle option de calcul d'itinéraire, nous devrons **modifier cette classe**, ce qui viole le Principe Ouvert/Fermé. Les différents algorithmes sont **fortement couplés** à la classe `NavigationSystem`.

Comment encapsuler des variantes d'un algorithme et les rendre interchangeables pour le client ?

## La solution : déléguer à des stratégies

Le **Pattern Strategy** (ou **Stratégie**) consiste à définir une famille d'algorithmes, à encapsuler chacun d'entre eux dans une classe distincte et à les rendre interchangeables. L'objet qui a besoin de l'algorithme (le "Contexte") ne connaît que l'interface commune, pas l'implémentation concrète.

Pensez à un GPS : vous (le Contexte) voulez aller d'un point A à un point B. Vous pouvez déléguer le calcul de l'itinéraire à différentes stratégies : "itinéraire le plus rapide", "itinéraire le plus court", "itinéraire à pied". Vous choisissez la stratégie, et le GPS l'exécute pour vous.

Le principe est le suivant :
1.  On définit une **interface `Strategy`** qui déclare la méthode que tous les algorithmes devront implémenter (ex: `calculate()`).
2.  On crée plusieurs **Stratégies Concrètes**, une pour chaque variante de l'algorithme. Chacune implémente l'interface `Strategy`.
3.  Le **Contexte** (notre `NavigationSystem`) possède une référence vers un objet de type `Strategy`.
4.  Le Contexte ne fait pas le travail lui-même ; il **délègue** l'appel à l'objet `Strategy` qu'il détient. Il peut changer de stratégie à tout moment.

### Mettre en place le Pattern Strategy

#### 1. L'interface de la Stratégie (Strategy)

C'est le contrat commun pour tous nos algorithmes de calcul d'itinéraire.

```php
// Fichier: strategies/RouteStrategyInterface.php
interface RouteStrategyInterface {
    public function calculate(string $origin, string $destination): string;
}
```

#### 2. Les Stratégies Concrètes (Concrete Strategies)

Chaque classe représente un algorithme spécifique. Elles sont interchangeables.

```php
// Fichier: strategies/FastestRouteStrategy.php
class FastestRouteStrategy implements RouteStrategyInterface {
    public function calculate(string $origin, string $destination): string {
        return "Itinéraire le plus RAPIDE de $origin à $destination via l'autoroute.";
    }
}

// Fichier: strategies/ShortestRouteStrategy.php
class ShortestRouteStrategy implements RouteStrategyInterface {
    public function calculate(string $origin, string $destination): string {
        return "Itinéraire le plus COURT de $origin à $destination via les routes nationales.";
    }
}

// Fichier: strategies/ScenicRouteStrategy.php
class ScenicRouteStrategy implements RouteStrategyInterface {
    public function calculate(string $origin, string $destination): string {
        return "Itinéraire TOURISTIQUE de $origin à $destination en passant par le point de vue.";
    }
}
```

#### 3. Le Contexte (Context)

C'est la classe qui utilisera une des stratégies. Elle ne sait pas quelle stratégie est utilisée, seulement qu'elle peut appeler la méthode `calculate`.

```php
// Fichier: NavigationSystem.php
class NavigationSystem {
    private RouteStrategyInterface $routeStrategy;

    // On peut injecter une stratégie par défaut
    public function __construct(RouteStrategyInterface $strategy) {
        $this->routeStrategy = $strategy;
    }

    // Permet de changer de stratégie à la volée !
    public function setStrategy(RouteStrategyInterface $strategy): void {
        $this->routeStrategy = $strategy;
    }

    public function getDirections(string $origin, string $destination): void {
        // Le contexte délègue le travail à l'objet stratégie.
        $route = $this->routeStrategy->calculate($origin, $destination);
        echo "GPS : $route\n";
    }
}
```

#### Utilisation du Pattern Strategy

Le code client peut maintenant choisir et changer l'algorithme de calcul d'itinéraire dynamiquement.

```php
// Fichier: index.php

// 1. Crée le contexte avec une stratégie par défaut (la plus rapide).
$gps = new NavigationSystem(new FastestRouteStrategy());
echo "Choix initial : Aller au travail.\n";
$gps->getDirections("Maison", "Bureau");

echo "\n--- C'est le week-end, changeons de stratégie ! ---\n";

// 2. On change la stratégie à la volée pour une balade.
$gps->setStrategy(new ScenicRouteStrategy());
$gps->getDirections("Maison", "Montagne");

echo "\n--- Plus d'essence, il faut prendre le plus court ! ---\n";

// 3. On change à nouveau pour économiser des kilomètres.
$gps->setStrategy(new ShortestRouteStrategy());
$gps->getDirections("Montagne", "Station Service");
```

**Résultat :**
```
Choix initial : Aller au travail.
GPS : Itinéraire le plus RAPIDE de Maison à Bureau via l'autoroute.

--- C'est le week-end, changeons de stratégie ! ---
GPS : Itinéraire TOURISTIQUE de Maison à Montagne en passant par le point de vue.

--- Plus d'essence, il faut prendre le plus court ! ---
GPS : Itinéraire le plus COURT de Montagne à Station Service via les routes nationales.
```

## Avantages du Pattern Strategy

-   **Algorithmes interchangeables** : Permet de changer l'algorithme utilisé par un objet même après son instanciation.
-   **Respect du Principe Ouvert/Fermé** : On peut introduire de nouvelles stratégies sans jamais modifier le code du Contexte (`NavigationSystem`).
-   **Élimine les structures conditionnelles** : Remplace les `if/else` ou `switch` par une composition d'objets, ce qui est beaucoup plus propre et maintenable.
-   **Découplage** : Sépare la logique de l'algorithme de la classe qui l'utilise. Le `NavigationSystem` ne se soucie pas de *comment* la route est calculée.