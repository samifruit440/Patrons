# 🔄 Patron Itérateur (Iterator)

> **Patron Comportemental** - Fournit un moyen d'accéder séquentiellement aux éléments d'une collection sans exposer sa représentation interne.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Fournir un moyen d'accéder **séquentiellement** aux éléments d'un objet agrégat sans exposer sa **représentation sous-jacente**.

---

## 🎯 Problème Résolu

- Comment parcourir une collection sans connaître sa structure interne?
- Comment avoir plusieurs parcours simultanés sur une même collection?
- Comment fournir une interface uniforme pour différentes structures?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON ITÉRATEUR                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────┐              ┌──────────────────┐         │
│  │   «interface»    │              │   «interface»    │         │
│  │    Iterateur     │              │     Agregat      │         │
│  ├──────────────────┤              ├──────────────────┤         │
│  │ +premier()       │              │ +creerIterateur()│         │
│  │ +suivant()       │◄─────────────│                  │         │
│  │ +estTermine()    │    crée      └────────┬─────────┘         │
│  │ +elementCourant()│                       │                   │
│  └────────┬─────────┘                       │                   │
│           │                                 │                   │
│           │                                 │                   │
│  ┌────────▼─────────┐              ┌────────▼─────────┐         │
│  │IterateurConcret  │              │  AgregatConcret  │         │
│  ├──────────────────┤              ├──────────────────┤         │
│  │ -agregat         │◄────────────►│ -elements[]      │         │
│  │ -indexCourant    │   accède à   ├──────────────────┤         │
│  ├──────────────────┤              │+creerIterateur() │         │
│  │ +premier()       │              │  {new Iterateur} │         │
│  │ +suivant()       │              └──────────────────┘         │
│  │ +estTermine()    │                                           │
│  │ +elementCourant()│                                           │
│  └──────────────────┘                                           │
│                                                                 │
│   L'itérateur encapsule la logique de parcours                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Types d'Itérateurs

```
┌─────────────────────────────────────────────────────────────────┐
│                  TYPES D'ITÉRATEURS                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ITÉRATEUR EXTERNE              ITÉRATEUR INTERNE              │
│   ┌──────────────────┐          ┌──────────────────┐            │
│   │ Client contrôle  │          │ Collection ctrl. │            │
│   │ le parcours      │          │ le parcours      │            │
│   │                  │          │                  │            │
│   │ while(!it.end()) │          │ col.forEach(     │            │
│   │   it.next();     │          │   action         │            │
│   │                  │          │ );               │            │
│   └──────────────────┘          └──────────────────┘            │
│                                                                 │
│   + Flexible                    + Simple à utiliser             │
│   + Plusieurs itérateurs        - Moins de contrôle             │
│   - Plus verbeux                                                │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ITÉRATEUR BIDIRECTIONNEL       ITÉRATEUR ALÉATOIRE            │
│   ┌──────────────────┐          ┌──────────────────┐            │
│   │ +precedent()     │          │ +allerA(index)   │            │
│   │ +suivant()       │          │ +sauter(n)       │            │
│   └──────────────────┘          └──────────────────┘            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Itérateur

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>
#include <stdexcept>

// Interface Itérateur générique
template<typename T>
class IIterateur {
public:
    virtual void premier() = 0;
    virtual void suivant() = 0;
    virtual bool estTermine() const = 0;
    virtual T& elementCourant() = 0;
    virtual ~IIterateur() = default;
};
```

### Collection et Itérateur Concrets

```cpp
// Forward declaration
template<typename T> class ListeIterateur;

// Collection (Agrégat)
template<typename T>
class Liste {
private:
    std::vector<T> elements;
    
public:
    void ajouter(const T& element) {
        elements.push_back(element);
    }
    
    void retirer(size_t index) {
        if (index < elements.size()) {
            elements.erase(elements.begin() + index);
        }
    }
    
    T& get(size_t index) {
        return elements.at(index);
    }
    
    const T& get(size_t index) const {
        return elements.at(index);
    }
    
    size_t taille() const {
        return elements.size();
    }
    
    // Factory method pour créer l'itérateur
    std::unique_ptr<IIterateur<T>> creerIterateur();
    
    // Itérateur inverse
    std::unique_ptr<IIterateur<T>> creerIterateurInverse();
};

// Itérateur Concret
template<typename T>
class ListeIterateur : public IIterateur<T> {
private:
    Liste<T>& liste;
    size_t indexCourant;
    
public:
    ListeIterateur(Liste<T>& l) : liste(l), indexCourant(0) {}
    
    void premier() override {
        indexCourant = 0;
    }
    
    void suivant() override {
        indexCourant++;
    }
    
    bool estTermine() const override {
        return indexCourant >= liste.taille();
    }
    
    T& elementCourant() override {
        if (estTermine()) {
            throw std::out_of_range("Itérateur terminé");
        }
        return liste.get(indexCourant);
    }
};

// Itérateur Inverse
template<typename T>
class ListeIterateurInverse : public IIterateur<T> {
private:
    Liste<T>& liste;
    int indexCourant;  // int car peut être négatif
    
public:
    ListeIterateurInverse(Liste<T>& l) 
        : liste(l), indexCourant(l.taille() - 1) {}
    
    void premier() override {
        indexCourant = liste.taille() - 1;
    }
    
    void suivant() override {
        indexCourant--;
    }
    
    bool estTermine() const override {
        return indexCourant < 0;
    }
    
    T& elementCourant() override {
        if (estTermine()) {
            throw std::out_of_range("Itérateur terminé");
        }
        return liste.get(indexCourant);
    }
};

// Implémentation des factory methods
template<typename T>
std::unique_ptr<IIterateur<T>> Liste<T>::creerIterateur() {
    return std::make_unique<ListeIterateur<T>>(*this);
}

template<typename T>
std::unique_ptr<IIterateur<T>> Liste<T>::creerIterateurInverse() {
    return std::make_unique<ListeIterateurInverse<T>>(*this);
}
```

### Itérateur avec Filtre

```cpp
// Itérateur avec condition de filtrage
template<typename T>
class ListeIterateurFiltre : public IIterateur<T> {
private:
    Liste<T>& liste;
    size_t indexCourant;
    std::function<bool(const T&)> predicat;
    
    void avancerVersProchainValide() {
        while (!estTermineInterne() && !predicat(liste.get(indexCourant))) {
            indexCourant++;
        }
    }
    
    bool estTermineInterne() const {
        return indexCourant >= liste.taille();
    }
    
public:
    ListeIterateurFiltre(Liste<T>& l, std::function<bool(const T&)> p) 
        : liste(l), indexCourant(0), predicat(p) {
        avancerVersProchainValide();
    }
    
    void premier() override {
        indexCourant = 0;
        avancerVersProchainValide();
    }
    
    void suivant() override {
        indexCourant++;
        avancerVersProchainValide();
    }
    
    bool estTermine() const override {
        return estTermineInterne();
    }
    
    T& elementCourant() override {
        return liste.get(indexCourant);
    }
};
```

### Exemple: Collection de Produits

```cpp
// Classe Produit
class Produit {
public:
    std::string nom;
    double prix;
    std::string categorie;
    
    Produit(const std::string& n, double p, const std::string& c)
        : nom(n), prix(p), categorie(c) {}
    
    void afficher() const {
        std::cout << "  • " << nom << " (" << categorie << ") - " 
                  << prix << "$" << std::endl;
    }
};

int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON ITÉRATEUR - DEMO          ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Créer une collection de produits
    Liste<Produit> inventaire;
    inventaire.ajouter(Produit("Laptop", 1299.99, "Électronique"));
    inventaire.ajouter(Produit("Souris", 29.99, "Électronique"));
    inventaire.ajouter(Produit("Café", 12.99, "Alimentation"));
    inventaire.ajouter(Produit("Clavier", 79.99, "Électronique"));
    inventaire.ajouter(Produit("Thé", 8.99, "Alimentation"));
    inventaire.ajouter(Produit("Écran", 399.99, "Électronique"));
    
    // Itération normale
    std::cout << "\n=== Parcours normal ===" << std::endl;
    auto it = inventaire.creerIterateur();
    for (it->premier(); !it->estTermine(); it->suivant()) {
        it->elementCourant().afficher();
    }
    
    // Itération inverse
    std::cout << "\n=== Parcours inverse ===" << std::endl;
    auto itInverse = inventaire.creerIterateurInverse();
    for (itInverse->premier(); !itInverse->estTermine(); itInverse->suivant()) {
        itInverse->elementCourant().afficher();
    }
    
    // Itération avec filtre
    std::cout << "\n=== Produits électroniques seulement ===" << std::endl;
    auto itFiltre = std::make_unique<ListeIterateurFiltre<Produit>>(
        inventaire,
        [](const Produit& p) { return p.categorie == "Électronique"; }
    );
    for (itFiltre->premier(); !itFiltre->estTermine(); itFiltre->suivant()) {
        itFiltre->elementCourant().afficher();
    }
    
    // Itération avec filtre sur prix
    std::cout << "\n=== Produits < 50$ ===" << std::endl;
    auto itPrix = std::make_unique<ListeIterateurFiltre<Produit>>(
        inventaire,
        [](const Produit& p) { return p.prix < 50.0; }
    );
    for (itPrix->premier(); !itPrix->estTermine(); itPrix->suivant()) {
        itPrix->elementCourant().afficher();
    }
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON ITÉRATEUR - DEMO          ║
╚═══════════════════════════════════════╝

=== Parcours normal ===
  • Laptop (Électronique) - 1299.99$
  • Souris (Électronique) - 29.99$
  • Café (Alimentation) - 12.99$
  • Clavier (Électronique) - 79.99$
  • Thé (Alimentation) - 8.99$
  • Écran (Électronique) - 399.99$

=== Parcours inverse ===
  • Écran (Électronique) - 399.99$
  • Thé (Alimentation) - 8.99$
  • Clavier (Électronique) - 79.99$
  • Café (Alimentation) - 12.99$
  • Souris (Électronique) - 29.99$
  • Laptop (Électronique) - 1299.99$

=== Produits électroniques seulement ===
  • Laptop (Électronique) - 1299.99$
  • Souris (Électronique) - 29.99$
  • Clavier (Électronique) - 79.99$
  • Écran (Électronique) - 399.99$

=== Produits < 50$ ===
  • Souris (Électronique) - 29.99$
  • Café (Alimentation) - 12.99$
  • Thé (Alimentation) - 8.99$
```

---

## 🔧 Style C++ Moderne (STL-compatible)

```cpp
// Itérateur compatible avec range-based for
template<typename T>
class Collection {
    std::vector<T> data;
    
public:
    // Types requis pour la STL
    using iterator = typename std::vector<T>::iterator;
    using const_iterator = typename std::vector<T>::const_iterator;
    
    iterator begin() { return data.begin(); }
    iterator end() { return data.end(); }
    const_iterator begin() const { return data.begin(); }
    const_iterator end() const { return data.end(); }
    
    void add(const T& item) { data.push_back(item); }
};

// Utilisation avec range-based for
Collection<int> col;
col.add(1); col.add(2); col.add(3);

for (const auto& item : col) {
    std::cout << item << std::endl;
}
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Encapsulation** | Structure interne cachée |
| **Polymorphisme** | Parcours uniforme de différentes structures |
| **Multiple itérateurs** | Plusieurs parcours simultanés |
| **Séparation** | La collection ne gère pas le parcours |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Complexité** | Plus de code pour collections simples |
| **Invalidation** | Modification pendant itération = problèmes |
| **Performance** | Overhead par rapport à l'accès direct |

---

## 🎯 Cas d'Utilisation

1. **Collections** - Listes, arbres, graphes
2. **Bases de données** - Curseurs, result sets
3. **Fichiers** - Lecture ligne par ligne
4. **Réseaux** - Flux de données

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Composite](../Structurel/Composite.md) | Itérateur pour parcourir le composite |
| [Visiteur](./Visiteur.md) | Alternative pour opérations sur éléments |
| [Factory Method](../Creationnel/Fabrique.md) | Création de l'itérateur |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Commande](./Commande.md)
