# 🎨 Patron Décorateur (Decorator)

> **Patron Structurel** - Attache dynamiquement des responsabilités supplémentaires à un objet.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

**Attacher dynamiquement des responsabilités supplémentaires** à un objet. Les décorateurs offrent une alternative flexible à l'héritage pour étendre les fonctionnalités.

---

## 🎯 Problème Résolu

- Comment ajouter des fonctionnalités à un objet sans modifier sa classe?
- Comment éviter l'explosion combinatoire de sous-classes?
- Comment ajouter/retirer des comportements à l'exécution?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON DÉCORATEUR                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌──────────────────┐                         │
│                    │   «interface»    │                         │
│                    │    Composant     │                         │
│                    ├──────────────────┤                         │
│                    │ +operation()     │                         │
│                    └────────┬─────────┘                         │
│                             │                                   │
│              ┌──────────────┴──────────────┐                    │
│              │                             │                    │
│    ┌─────────▼─────────┐       ┌───────────▼────────────┐       │
│    │ ComposantConcret  │       │  «abstract»            │       │
│    ├───────────────────┤       │   Decorateur           │       │
│    │ +operation()      │       ├────────────────────────┤       │
│    └───────────────────┘       │ -composant: Composant  │       │
│                                ├────────────────────────┤       │
│                                │ +operation()           │       │
│                                └───────────┬────────────┘       │
│                                            │                    │
│                         ┌──────────────────┼──────────────┐     │
│                         │                  │              │     │
│               ┌─────────▼────────┐  ┌──────▼─────┐  ┌─────▼────┐│
│               │ DecorateurA      │  │DecorateurB │  │Decorat...││
│               ├──────────────────┤  ├────────────┤  └──────────┘│
│               │ +operation()     │  │+operation()│              │
│               │ +comportementA() │  └────────────┘              │
│               └──────────────────┘                              │
│                                                                 │
│   operation() {                                                 │
│       composant.operation(); // délègue                         │
│       comportementA();       // ajoute                          │
│   }                                                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🧅 Structure en Couches

```
┌─────────────────────────────────────────────────────────────────┐
│                  EMPILEMENT DES DÉCORATEURS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│         ┌───────────────────────────────────────┐               │
│         │          DecorateurLait               │               │
│         │   ┌───────────────────────────────┐   │               │
│         │   │      DecorateurSucre          │   │               │
│         │   │   ┌───────────────────────┐   │   │               │
│         │   │   │    CaféSimple         │   │   │               │
│         │   │   │    (Composant)        │   │   │               │
│         │   │   └───────────────────────┘   │   │               │
│         │   └───────────────────────────────┘   │               │
│         └───────────────────────────────────────┘               │
│                                                                 │
│   Appel: café.getPrix()                                         │
│   1. DecorateurLait.getPrix()                                   │
│      → composant.getPrix() + 0.50                               │
│      2. DecorateurSucre.getPrix()                               │
│         → composant.getPrix() + 0.25                            │
│         3. CaféSimple.getPrix()                                 │
│            → 2.00                                               │
│                                                                 │
│   Total: 2.00 + 0.25 + 0.50 = 2.75$                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Composant

```cpp
#include <iostream>
#include <string>
#include <memory>

// Interface Composant
class IBoisson {
public:
    virtual std::string getDescription() const = 0;
    virtual double getPrix() const = 0;
    virtual ~IBoisson() = default;
};
```

### Composants Concrets

```cpp
// Composant Concret 1: Espresso
class Espresso : public IBoisson {
public:
    std::string getDescription() const override {
        return "Espresso";
    }
    
    double getPrix() const override {
        return 2.50;
    }
};

// Composant Concret 2: Café Filtre
class CafeFiltre : public IBoisson {
public:
    std::string getDescription() const override {
        return "Café Filtre";
    }
    
    double getPrix() const override {
        return 1.50;
    }
};

// Composant Concret 3: Thé
class The : public IBoisson {
public:
    std::string getDescription() const override {
        return "Thé";
    }
    
    double getPrix() const override {
        return 2.00;
    }
};
```

### Décorateur Abstrait

```cpp
// Décorateur Abstrait
class DecorateurBoisson : public IBoisson {
protected:
    std::unique_ptr<IBoisson> boisson;  // Composant enveloppé
    
public:
    DecorateurBoisson(std::unique_ptr<IBoisson> b) 
        : boisson(std::move(b)) {}
    
    // Par défaut, délègue au composant
    std::string getDescription() const override {
        return boisson->getDescription();
    }
    
    double getPrix() const override {
        return boisson->getPrix();
    }
};
```

### Décorateurs Concrets

```cpp
// Décorateur: Lait
class Lait : public DecorateurBoisson {
public:
    Lait(std::unique_ptr<IBoisson> b) : DecorateurBoisson(std::move(b)) {}
    
    std::string getDescription() const override {
        return boisson->getDescription() + ", Lait";
    }
    
    double getPrix() const override {
        return boisson->getPrix() + 0.50;
    }
};

// Décorateur: Sucre
class Sucre : public DecorateurBoisson {
public:
    Sucre(std::unique_ptr<IBoisson> b) : DecorateurBoisson(std::move(b)) {}
    
    std::string getDescription() const override {
        return boisson->getDescription() + ", Sucre";
    }
    
    double getPrix() const override {
        return boisson->getPrix() + 0.25;
    }
};

// Décorateur: Crème Fouettée
class CremeFouettee : public DecorateurBoisson {
public:
    CremeFouettee(std::unique_ptr<IBoisson> b) : DecorateurBoisson(std::move(b)) {}
    
    std::string getDescription() const override {
        return boisson->getDescription() + ", Crème Fouettée";
    }
    
    double getPrix() const override {
        return boisson->getPrix() + 0.75;
    }
};

// Décorateur: Chocolat
class Chocolat : public DecorateurBoisson {
public:
    Chocolat(std::unique_ptr<IBoisson> b) : DecorateurBoisson(std::move(b)) {}
    
    std::string getDescription() const override {
        return boisson->getDescription() + ", Chocolat";
    }
    
    double getPrix() const override {
        return boisson->getPrix() + 0.60;
    }
};

// Décorateur: Vanille
class Vanille : public DecorateurBoisson {
public:
    Vanille(std::unique_ptr<IBoisson> b) : DecorateurBoisson(std::move(b)) {}
    
    std::string getDescription() const override {
        return boisson->getDescription() + ", Vanille";
    }
    
    double getPrix() const override {
        return boisson->getPrix() + 0.45;
    }
};
```

### Utilisation

```cpp
void afficherCommande(const IBoisson& boisson) {
    std::cout << boisson.getDescription() 
              << " → " << boisson.getPrix() << "$" << std::endl;
}

int main() {
    std::cout << "=== Menu du Café ===" << std::endl << std::endl;
    
    // Commande 1: Espresso simple
    std::unique_ptr<IBoisson> commande1 = std::make_unique<Espresso>();
    std::cout << "Commande 1: ";
    afficherCommande(*commande1);
    
    // Commande 2: Espresso avec lait et sucre
    std::unique_ptr<IBoisson> commande2 = 
        std::make_unique<Sucre>(
            std::make_unique<Lait>(
                std::make_unique<Espresso>()
            )
        );
    std::cout << "Commande 2: ";
    afficherCommande(*commande2);
    
    // Commande 3: Café filtre avec crème et double sucre
    std::unique_ptr<IBoisson> commande3 =
        std::make_unique<Sucre>(
            std::make_unique<Sucre>(  // Double sucre!
                std::make_unique<CremeFouettee>(
                    std::make_unique<CafeFiltre>()
                )
            )
        );
    std::cout << "Commande 3: ";
    afficherCommande(*commande3);
    
    // Commande 4: Mocha (espresso + chocolat + lait + crème)
    std::unique_ptr<IBoisson> mocha =
        std::make_unique<CremeFouettee>(
            std::make_unique<Lait>(
                std::make_unique<Chocolat>(
                    std::make_unique<Espresso>()
                )
            )
        );
    std::cout << "Commande 4 (Mocha): ";
    afficherCommande(*mocha);
    
    // Commande 5: Thé vanille avec lait
    std::unique_ptr<IBoisson> theVanille =
        std::make_unique<Lait>(
            std::make_unique<Vanille>(
                std::make_unique<The>()
            )
        );
    std::cout << "Commande 5: ";
    afficherCommande(*theVanille);
    
    return 0;
}
```

### Sortie

```
=== Menu du Café ===

Commande 1: Espresso → 2.5$
Commande 2: Espresso, Lait, Sucre → 3.25$
Commande 3: Café Filtre, Crème Fouettée, Sucre, Sucre → 2.75$
Commande 4 (Mocha): Espresso, Chocolat, Lait, Crème Fouettée → 4.35$
Commande 5: Thé, Vanille, Lait → 2.95$
```

---

## 🆚 Décorateur vs Héritage

```
┌─────────────────────────────────────────────────────────────────┐
│            EXPLOSION COMBINATOIRE AVEC HÉRITAGE                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Sans décorateur (héritage):                                   │
│   - EspressoAvecLait                                            │
│   - EspressoAvecSucre                                           │
│   - EspressoAvecLaitEtSucre                                     │
│   - EspressoAvecLaitEtCreme                                     │
│   - EspressoAvecSucreEtCreme                                    │
│   - EspressoAvecLaitSucreCreme                                  │
│   - CafeAvecLait                                                │
│   - CafeAvecSucre                                               │
│   - ... (2^n combinaisons!)                                     │
│                                                                 │
│   Avec décorateur:                                              │
│   - 3 boissons de base + 5 décorateurs = 8 classes              │
│   - Mais TOUTES les combinaisons possibles!                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎨 Exemple Visuel: Système de Points

```cpp
// Interface
class IPointAffichable {
public:
    virtual void dessiner() const = 0;
    virtual ~IPointAffichable() = default;
};

// Composant concret
class Point : public IPointAffichable {
    double x, y;
public:
    Point(double x, double y) : x(x), y(y) {}
    void dessiner() const override {
        std::cout << "Point(" << x << ", " << y << ")";
    }
};

// Décorateur abstrait
class DecorateurPoint : public IPointAffichable {
protected:
    std::unique_ptr<IPointAffichable> point;
public:
    DecorateurPoint(std::unique_ptr<IPointAffichable> p) : point(std::move(p)) {}
};

// Décorateur: Couleur
class PointColore : public DecorateurPoint {
    std::string couleur;
public:
    PointColore(std::unique_ptr<IPointAffichable> p, const std::string& c)
        : DecorateurPoint(std::move(p)), couleur(c) {}
    
    void dessiner() const override {
        std::cout << "[" << couleur << "] ";
        point->dessiner();
    }
};

// Décorateur: Texture
class PointTexture : public DecorateurPoint {
    std::string texture;
public:
    PointTexture(std::unique_ptr<IPointAffichable> p, const std::string& t)
        : DecorateurPoint(std::move(p)), texture(t) {}
    
    void dessiner() const override {
        point->dessiner();
        std::cout << " {texture: " << texture << "}";
    }
};

// Usage
auto point = std::make_unique<PointTexture>(
    std::make_unique<PointColore>(
        std::make_unique<Point>(3.0, 4.0),
        "Rouge"
    ),
    "Metal"
);
point->dessiner();
// Sortie: [Rouge] Point(3, 4) {texture: Metal}
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Flexibilité** | Ajouter/retirer comportements à l'exécution |
| **Single Responsibility** | Chaque décorateur = une responsabilité |
| **Open/Closed** | Étendre sans modifier le composant |
| **Composition** | Alternative à l'héritage multiple |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Beaucoup d'objets** | Plusieurs petits objets similaires |
| **Ordre important** | L'ordre d'empilement peut affecter le résultat |
| **Complexité** | Débogage de la chaîne de décorateurs |
| **Identité** | `decorated != original` (pas la même instance) |

---

## 🎯 Cas d'Utilisation

1. **Streams I/O** - Java InputStream/OutputStream
2. **GUI** - Bordures, scrollbars, etc.
3. **Middleware** - Chaîne de traitements HTTP
4. **Logging** - Enrichir les messages
5. **Cache** - Décorer un repository

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Composite](./Composite.md) | Structure similaire mais but différent |
| [Stratégie](../Comportemental/Strategie.md) | Alternative: changer l'intérieur vs envelopper |
| [Proxy](./Proxy.md) | Interface identique mais contrôle l'accès |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Façade](./Facade.md)
