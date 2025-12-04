# 📊 Patron Stratégie (Strategy)

> **Patron Comportemental** - Définit une famille d'algorithmes interchangeables encapsulés dans des classes séparées.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Définir une **famille d'algorithmes**, encapsuler chacun d'eux et les rendre **interchangeables**. La Stratégie permet de faire varier l'algorithme **indépendamment** des clients qui l'utilisent.

---

## 🎯 Problème Résolu

- Comment éviter les gros `switch/case` pour choisir un algorithme?
- Comment changer d'algorithme à l'exécution?
- Comment ajouter de nouveaux algorithmes sans modifier le code existant?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON STRATÉGIE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│     ┌─────────────────────┐       ┌──────────────────────┐      │
│     │      Contexte       │       │     «interface»      │      │
│     ├─────────────────────┤       │      IStrategie      │      │
│     │ -strategie          │──────►├──────────────────────┤      │
│     ├─────────────────────┤       │ +executer()          │      │
│     │ +setStrategie(s)    │       └──────────┬───────────┘      │
│     │ +executerStrategie()│                  │                  │
│     └─────────────────────┘                  │                  │
│                                              │                  │
│     executerStrategie() {       ┌────────────┼────────────┐     │
│         strategie.executer();   │            │            │     │
│     }                           │            │            │     │
│                        ┌────────▼───┐ ┌──────▼─────┐ ┌────▼────┐│
│                        │StrategieA  │ │StrategieB  │ │Strateg..││
│                        ├────────────┤ ├────────────┤ ├─────────┤│
│                        │+executer() │ │+executer() │ │executer ││
│                        │{ algo A }  │ │{ algo B }  │ │         ││
│                        └────────────┘ └────────────┘ └─────────┘│
│                                                                 │
│   Le client peut changer de stratégie à tout moment             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Changement Dynamique

```
┌─────────────────────────────────────────────────────────────────┐
│                    STRATÉGIE À L'EXÉCUTION                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client                Contexte              Stratégie         │
│     │                      │                      │             │
│     │ setStrategie(A)      │                      │             │
│     │─────────────────────►│                      │             │
│     │                      │                      │             │
│     │ executer()           │                      │             │
│     │─────────────────────►│    executer()        │             │
│     │                      │─────────────────────►│             │
│     │                      │   [Algo A s'exécute] │             │
│     │                      │◄─────────────────────│             │
│     │                      │                      │             │
│     │ setStrategie(B)      │    ┌───────────┐     │             │
│     │─────────────────────►│───►│StrategieB │     │             │
│     │                      │    └───────────┘     │             │
│     │ executer()           │                      │             │
│     │─────────────────────►│    executer()        │             │
│     │                      │─────────────────────►│             │
│     │                      │   [Algo B s'exécute] │             │
│                                                                 │
│   L'algorithme change SANS modifier le Contexte                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Stratégie

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>
#include <algorithm>

// Interface Stratégie de tri
class IStrategieTri {
public:
    virtual void trier(std::vector<int>& donnees) = 0;
    virtual std::string getNom() const = 0;
    virtual ~IStrategieTri() = default;
};
```

### Stratégies Concrètes

```cpp
// Stratégie 1: Tri à bulles
class TriBulles : public IStrategieTri {
public:
    void trier(std::vector<int>& donnees) override {
        std::cout << "   🔄 Exécution du tri à bulles..." << std::endl;
        int n = donnees.size();
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (donnees[j] > donnees[j + 1]) {
                    std::swap(donnees[j], donnees[j + 1]);
                }
            }
        }
    }
    
    std::string getNom() const override { 
        return "Tri à bulles (O(n²))"; 
    }
};

// Stratégie 2: Tri rapide (QuickSort)
class TriRapide : public IStrategieTri {
private:
    int partition(std::vector<int>& arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (arr[j] <= pivot) {
                i++;
                std::swap(arr[i], arr[j]);
            }
        }
        std::swap(arr[i + 1], arr[high]);
        return i + 1;
    }
    
    void quickSort(std::vector<int>& arr, int low, int high) {
        if (low < high) {
            int pi = partition(arr, low, high);
            quickSort(arr, low, pi - 1);
            quickSort(arr, pi + 1, high);
        }
    }
    
public:
    void trier(std::vector<int>& donnees) override {
        std::cout << "   ⚡ Exécution du tri rapide..." << std::endl;
        if (!donnees.empty()) {
            quickSort(donnees, 0, donnees.size() - 1);
        }
    }
    
    std::string getNom() const override { 
        return "Tri rapide (O(n log n))"; 
    }
};

// Stratégie 3: Tri par insertion
class TriInsertion : public IStrategieTri {
public:
    void trier(std::vector<int>& donnees) override {
        std::cout << "   📥 Exécution du tri par insertion..." << std::endl;
        int n = donnees.size();
        for (int i = 1; i < n; i++) {
            int key = donnees[i];
            int j = i - 1;
            while (j >= 0 && donnees[j] > key) {
                donnees[j + 1] = donnees[j];
                j--;
            }
            donnees[j + 1] = key;
        }
    }
    
    std::string getNom() const override { 
        return "Tri par insertion (O(n²))"; 
    }
};

// Stratégie 4: Tri STL (std::sort)
class TriSTL : public IStrategieTri {
public:
    void trier(std::vector<int>& donnees) override {
        std::cout << "   📚 Exécution du tri STL (IntroSort)..." << std::endl;
        std::sort(donnees.begin(), donnees.end());
    }
    
    std::string getNom() const override { 
        return "Tri STL std::sort (O(n log n))"; 
    }
};
```

### Contexte

```cpp
// Contexte: Gestionnaire de données
class GestionnaireDonnees {
private:
    std::vector<int> donnees;
    std::shared_ptr<IStrategieTri> strategie;
    
public:
    GestionnaireDonnees() : strategie(nullptr) {}
    
    // Injection de la stratégie
    void setStrategie(std::shared_ptr<IStrategieTri> s) {
        strategie = s;
        std::cout << "📌 Stratégie définie: " << s->getNom() << std::endl;
    }
    
    void setDonnees(const std::vector<int>& d) {
        donnees = d;
    }
    
    void trier() {
        if (!strategie) {
            std::cout << "❌ Aucune stratégie définie!" << std::endl;
            return;
        }
        
        std::cout << "\n🔧 Tri en cours avec: " << strategie->getNom() << std::endl;
        strategie->trier(donnees);
        std::cout << "   ✅ Tri terminé!" << std::endl;
    }
    
    void afficher() const {
        std::cout << "   Données: [";
        for (size_t i = 0; i < donnees.size(); i++) {
            std::cout << donnees[i];
            if (i < donnees.size() - 1) std::cout << ", ";
        }
        std::cout << "]" << std::endl;
    }
    
    // Sélection automatique selon la taille
    void trierAuto() {
        if (donnees.size() < 10) {
            setStrategie(std::make_shared<TriInsertion>());
        } else if (donnees.size() < 1000) {
            setStrategie(std::make_shared<TriRapide>());
        } else {
            setStrategie(std::make_shared<TriSTL>());
        }
        trier();
    }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔════════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON STRATÉGIE - DEMO           ║" << std::endl;
    std::cout << "╚════════════════════════════════════════╝" << std::endl;
    
    GestionnaireDonnees gestionnaire;
    
    // Données de test
    std::vector<int> donnees = {64, 34, 25, 12, 22, 11, 90, 42};
    
    std::cout << "\n=== Données initiales ===" << std::endl;
    gestionnaire.setDonnees(donnees);
    gestionnaire.afficher();
    
    // Test 1: Tri à bulles
    std::cout << "\n=== Test 1: Tri à bulles ===" << std::endl;
    gestionnaire.setDonnees(donnees);  // Reset
    gestionnaire.setStrategie(std::make_shared<TriBulles>());
    gestionnaire.trier();
    gestionnaire.afficher();
    
    // Test 2: Tri rapide
    std::cout << "\n=== Test 2: Tri rapide ===" << std::endl;
    gestionnaire.setDonnees(donnees);  // Reset
    gestionnaire.setStrategie(std::make_shared<TriRapide>());
    gestionnaire.trier();
    gestionnaire.afficher();
    
    // Test 3: Changer de stratégie dynamiquement
    std::cout << "\n=== Test 3: Changement dynamique ===" << std::endl;
    gestionnaire.setDonnees({5, 2, 8, 1, 9});
    std::cout << "Avant: ";
    gestionnaire.afficher();
    
    gestionnaire.setStrategie(std::make_shared<TriInsertion>());
    gestionnaire.trier();
    gestionnaire.afficher();
    
    // Test 4: Sélection automatique
    std::cout << "\n=== Test 4: Sélection automatique ===" << std::endl;
    gestionnaire.setDonnees({3, 1, 4, 1, 5});
    gestionnaire.trierAuto();
    gestionnaire.afficher();
    
    return 0;
}
```

### Sortie

```
╔════════════════════════════════════════╗
║      PATRON STRATÉGIE - DEMO           ║
╚════════════════════════════════════════╝

=== Données initiales ===
   Données: [64, 34, 25, 12, 22, 11, 90, 42]

=== Test 1: Tri à bulles ===
📌 Stratégie définie: Tri à bulles (O(n²))

🔧 Tri en cours avec: Tri à bulles (O(n²))
   🔄 Exécution du tri à bulles...
   ✅ Tri terminé!
   Données: [11, 12, 22, 25, 34, 42, 64, 90]

=== Test 2: Tri rapide ===
📌 Stratégie définie: Tri rapide (O(n log n))

🔧 Tri en cours avec: Tri rapide (O(n log n))
   ⚡ Exécution du tri rapide...
   ✅ Tri terminé!
   Données: [11, 12, 22, 25, 34, 42, 64, 90]

=== Test 3: Changement dynamique ===
Avant:    Données: [5, 2, 8, 1, 9]
📌 Stratégie définie: Tri par insertion (O(n²))

🔧 Tri en cours avec: Tri par insertion (O(n²))
   📥 Exécution du tri par insertion...
   ✅ Tri terminé!
   Données: [1, 2, 5, 8, 9]

=== Test 4: Sélection automatique ===
📌 Stratégie définie: Tri par insertion (O(n²))

🔧 Tri en cours avec: Tri par insertion (O(n²))
   📥 Exécution du tri par insertion...
   ✅ Tri terminé!
   Données: [1, 1, 3, 4, 5]
```

---

## 🎨 Autre Exemple: Compression

```cpp
// Interface Stratégie de compression
class IStrategieCompression {
public:
    virtual std::string compresser(const std::string& donnees) = 0;
    virtual std::string decompresser(const std::string& donnees) = 0;
    virtual ~IStrategieCompression() = default;
};

// Stratégies concrètes
class CompressionZip : public IStrategieCompression {
public:
    std::string compresser(const std::string& d) override {
        return "[ZIP:" + d + "]";
    }
    std::string decompresser(const std::string& d) override {
        return d.substr(5, d.length() - 6);
    }
};

class CompressionRAR : public IStrategieCompression {
public:
    std::string compresser(const std::string& d) override {
        return "[RAR:" + d + "]";
    }
    std::string decompresser(const std::string& d) override {
        return d.substr(5, d.length() - 6);
    }
};

// Contexte
class Archiveur {
    std::shared_ptr<IStrategieCompression> strategie;
public:
    void setStrategie(std::shared_ptr<IStrategieCompression> s) {
        strategie = s;
    }
    
    std::string archiver(const std::string& fichier) {
        return strategie->compresser(fichier);
    }
};
```

---

## 🆚 Stratégie vs État

```
┌─────────────────────────────────────────────────────────────────┐
│                STRATÉGIE vs ÉTAT                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   STRATÉGIE                      ÉTAT                           │
│   ──────────                     ────                           │
│                                                                 │
│   • Algorithmes alternatifs      • Comportements selon état     │
│   • Client choisit la stratégie  • Transitions automatiques     │
│   • Indépendant du contexte      • États connaissent contexte   │
│   • Pas de changement auto       • Changements internes         │
│                                                                 │
│   Exemple: Tri                   Exemple: Distributeur          │
│   ┌─────────────┐                ┌─────────────┐                │
│   │ setStrategie│                │ etatCourant │                │
│   │ (TriRapide) │                │ .action()   │                │
│   └─────────────┘                │    ↓        │                │
│         ↓                        │ transition  │                │
│   [Tri exécuté]                  │ automatique │                │
│                                  └─────────────┘                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Stratégie avec Lambda (C++11+)

```cpp
#include <functional>

class ContexteLambda {
    std::function<void(std::vector<int>&)> strategie;
    
public:
    void setStrategie(std::function<void(std::vector<int>&)> s) {
        strategie = s;
    }
    
    void executer(std::vector<int>& donnees) {
        strategie(donnees);
    }
};

// Utilisation
ContexteLambda ctx;

// Stratégie inline avec lambda
ctx.setStrategie([](std::vector<int>& v) {
    std::sort(v.begin(), v.end());
});

// Ou tri inversé
ctx.setStrategie([](std::vector<int>& v) {
    std::sort(v.begin(), v.end(), std::greater<int>());
});
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Flexibilité** | Changement d'algorithme à l'exécution |
| **Isolation** | Chaque algorithme dans sa classe |
| **Extensibilité** | Nouvelle stratégie = nouvelle classe |
| **Testabilité** | Stratégies testables indépendamment |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Nombre de classes** | Une classe par algorithme |
| **Client averti** | Le client doit connaître les stratégies |
| **Overhead** | Indirection supplémentaire |

---

## 🎯 Cas d'Utilisation

1. **Algorithmes de tri** - QuickSort, MergeSort, BubbleSort
2. **Compression** - ZIP, RAR, GZIP
3. **Calcul de prix** - Remises, taxes, promotions
4. **Validation** - Différentes règles de validation
5. **Routage** - Différents algorithmes de chemin

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [État](./Etat.md) | Structure similaire, but différent |
| [Patron de Méthode](./PatronMethode.md) | Héritage vs composition |
| [Décorateur](../Structurel/Decorateur.md) | Peut décorer une stratégie |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Commande](./Commande.md)
