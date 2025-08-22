# Le Pattern Façade : Simplifier une interface complexe

## Le problème : un sous-système décourageant

Notre `BasicCar` est pour l'instant très simple. Mais une vraie voiture est un système complexe. Pour démarrer, il ne suffit pas de "démarrer le moteur". Une séquence d'opérations complexes et interdépendantes doit avoir lieu :
1.  L'ordinateur de bord doit vérifier tous les systèmes (pression des pneus, niveaux d'huile, etc.).
2.  Le système d'injection de carburant doit s'amorcer.
3.  L'allumage doit produire une étincelle.
4.  Le moteur doit enfin démarrer.
5.  Le système de verrouillage centralisé doit peut-être se désactiver.

Si le code client (par exemple, le code qui répond quand le conducteur appuie sur le bouton "Start") devait gérer tout ça, il serait très complexe :

```php
// Code client SANS façade
$computer = new OnBoardComputer();
$fuelInjector = new FuelInjector();
$ignition = new IgnitionSystem();
$engine = new Engine();

$computer->checkSystems();
$fuelInjector->prime();
$fuelInjector->inject();
$ignition->powerOn();
$engine->ignite();
```

C'est **compliqué** et **fortement couplé**. Le client doit connaître chaque composant du sous-système de démarrage et l'ordre exact des opérations. Si demain on ajoute une étape (vérifier la batterie), il faudra modifier ce code partout où il est utilisé.

Comment fournir une interface simple pour démarrer la voiture, sans exposer toute cette complexité ?

## La solution : une façade unifiée

Le **Pattern Façade** (ou **Façade**) résout ce problème en introduisant une classe unique qui agit comme un point d'entrée simplifié pour le sous-système. Pensez au bouton "Start" de votre voiture : vous appuyez sur un seul bouton, et la voiture (la façade) se charge d'orchestrer l'ordinateur de bord, l'injection et le moteur pour vous.

Le principe est le suivant :
1.  La façade connaît les classes du sous-système qu'elle doit piloter.
2.  Elle expose des méthodes simples et de haut niveau (ex: `startCar()`).
3.  Quand une de ces méthodes est appelée, la façade se charge d'invoquer les bonnes méthodes sur les bons objets du sous-système, dans le bon ordre.

Le client n'interagit plus qu'avec la façade, ignorant complètement la complexité qui se cache derrière.

### Mettre en place le Pattern Façade

#### 1. Le sous-système complexe

D'abord, nous avons nos différentes classes qui représentent les composants de la voiture. Elles sont spécialisées et ne se connaissent pas.

```php
// Fichier: subsystem/Engine.php
class Engine {
    public function ignite(): void { echo "Le moteur s'allume.\n"; }
    public function shutdown(): void { echo "Le moteur s'éteint.\n"; }
}

// Fichier: subsystem/FuelInjector.php
class FuelInjector {
    public function inject(): void { echo "Le carburant est injecté.\n"; }
}

// Fichier: subsystem/OnBoardComputer.php
class OnBoardComputer {
    public function checkSystems(): void { echo "L'ordinateur de bord vérifie les systèmes... OK.\n"; }
}
```

#### 2. La classe Façade

C'est notre voiture "intelligente". Elle encapsule la complexité du démarrage. Elle est composée des objets du sous-système.

```php
// Fichier: CarFacade.php
class CarFacade {
    private Engine $engine;
    private FuelInjector $injector;
    private OnBoardComputer $computer;

    public function __construct(Engine $engine, FuelInjector $injector, OnBoardComputer $computer) {
        $this->engine = $engine;
        $this->injector = $injector;
        $this->computer = $computer;
    }

    /**
     * Une méthode simple pour une tâche complexe.
     * Le client n'a plus besoin de connaître les sous-systèmes.
     */
    public function startCar(): void {
        echo "Démarrage de la voiture...\n";
        $this->computer->checkSystems();
        $this->injector->inject();
        $this->engine->ignite();
        echo "Voiture prête !\n";
    }

    /**
     * Une autre méthode simple pour arrêter la voiture.
     */
    public function stopCar(): void {
        echo "\nArrêt de la voiture...\n";
        $this->engine->shutdown();
    }
}
```

#### Utilisation de la Façade

Le code client devient incroyablement simple et lisible.

```php
// Fichier: index.php

// 1. On instancie les composants du sous-système (ceci pourrait être géré
// par une Factory ou un conteneur d'injection de dépendances).
$engine = new Engine();
$injector = new FuelInjector();
$computer = new OnBoardComputer();

// 2. On crée la façade avec ces composants.
$myCar = new CarFacade($engine, $injector, $computer);

// 3. On utilise l'interface simplifiée !
$myCar->startCar();
$myCar->stopCar();
```

**Résultat :**
```
Démarrage de la voiture...
L'ordinateur de bord vérifie les systèmes... OK.
Le carburant est injecté.
Le moteur s'allume.
Voiture prête !

Arrêt de la voiture...
Le moteur s'éteint.
```

## Avantages du Pattern Façade

-   **Simplicité** : Fournit une interface simple et unifiée pour un sous-système complexe (le démarrage d'une voiture).
-   **Découplage** : Réduit le couplage entre le code client et les composants internes de la voiture. Si on change le système d'injection, seul le code de la `CarFacade` doit être adapté, pas celui du client.
-   **Lisibilité** : Le code client est centré sur l'intention (`startCar()`) plutôt que sur les détails techniques.
-   **Point de contrôle centralisé** : La façade est un bon endroit pour ajouter des logs ou des vérifications de sécurité pour l'ensemble du processus.
