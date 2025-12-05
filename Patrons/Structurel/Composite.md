# 🌳 Patron Composite

> **Patron Structurel** - Compose des objets en structures arborescentes pour représenter des hiérarchies partie-tout.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Composer des objets en **structures arborescentes** pour représenter des hiérarchies **partie-tout**. Le Composite permet aux clients de traiter de manière **uniforme** les objets individuels et les compositions d'objets.

---

## 🎯 Problème Résolu

- Comment représenter une hiérarchie arborescente d'objets?
- Comment traiter uniformément les objets simples et composés?
- Comment propager des opérations dans une structure récursive?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON COMPOSITE                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                 ┌──────────────────────┐                        │
│                 │    «interface»       │                        │
│                 │     Composant        │◄────────────────────┐  │
│                 ├──────────────────────┤                     │  │
│                 │ +operation()         │                     │  │
│                 │ +ajouter(Composant)  │                     │  │
│                 │ +retirer(Composant)  │                     │  │
│                 │ +getEnfant(i)        │                     │  │
│                 └──────────▲───────────┘                     │  │
│                            │                                 │  │
│              ┌─────────────┴─────────────┐                   │  │
│              │                           │                   │  │
│    ┌─────────┴─────────┐       ┌─────────┴─────────┐         │  │
│    │      Feuille      │       │     Composite     │◆───────┘  │
│    ├───────────────────┤       ├───────────────────┤   enfants  │
│    │ +operation() {    │       │ -enfants[]        │            │
│    │   // action simple│       ├───────────────────┤            │
│    │ }                 │       │ +operation() {    │            │
│    └───────────────────┘       │   for(e: enfants) │            │
│                                │     e.operation() │            │
│                                │ }                 │            │
│                                │ +ajouter(c)       │            │
│                                │ +retirer(c)       │            │
│                                └───────────────────┘            │
│                                                                 │
│   Le client traite Feuille et Composite de la même façon        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🌲 Structure Arborescente

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXEMPLE: SYSTÈME DE FICHIERS                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                        ┌─────────────┐                          │
│                        │  Racine (/) │ ◄── Composite            │
│                        └──────┬──────┘                          │
│               ┌───────────────┼───────────────┐                 │
│               │               │               │                 │
│        ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐          │
│        │    home     │ │     etc     │ │    var      │          │
│        │ (Composite) │ │ (Composite) │ │ (Composite) │          │
│        └──────┬──────┘ └─────────────┘ └─────────────┘          │
│               │                                                 │
│        ┌──────┴──────┐                                          │
│        │             │                                          │
│ ┌──────▼──────┐ ┌────▼─────┐                                    │
│ │   user1     │ │  user2   │                                    │
│ │ (Composite) │ │(Composite)│                                   │
│ └──────┬──────┘ └──────────┘                                    │
│        │                                                        │
│ ┌──────┴──────────────┐                                         │
│ │          │          │                                         │
│ ▼          ▼          ▼                                         │
│ fichier1  fichier2  docs/                                       │
│ (Feuille) (Feuille) (Composite)                                 │
│                                                                 │
│   Appel récursif: racine.getTaille() additionne tout            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Composant

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>
#include <algorithm>

// Interface Composant
class IComposantFichier {
public:
    virtual std::string getNom() const = 0;
    virtual int getTaille() const = 0;
    virtual void afficher(int indentation = 0) const = 0;
    
    // Méthodes de gestion des enfants (par défaut: erreur)
    virtual void ajouter(std::shared_ptr<IComposantFichier> composant) {
        throw std::runtime_error("Opération non supportée");
    }
    virtual void retirer(std::shared_ptr<IComposantFichier> composant) {
        throw std::runtime_error("Opération non supportée");
    }
    virtual std::shared_ptr<IComposantFichier> getEnfant(int index) const {
        throw std::runtime_error("Opération non supportée");
    }
    virtual bool estComposite() const { return false; }
    
    virtual ~IComposantFichier() = default;
    
protected:
    std::string indent(int n) const {
        return std::string(n * 2, ' ');
    }
};
```

### Feuille (Fichier)

```cpp
// Feuille: Fichier simple
class Fichier : public IComposantFichier {
private:
    std::string nom;
    int taille;  // en Ko
    
public:
    Fichier(const std::string& n, int t) : nom(n), taille(t) {}
    
    std::string getNom() const override { return nom; }
    
    int getTaille() const override { return taille; }
    
    void afficher(int indentation = 0) const override {
        std::cout << indent(indentation) << "📄 " << nom 
                  << " (" << taille << " Ko)" << std::endl;
    }
};
```

### Composite (Dossier)

```cpp
// Composite: Dossier contenant fichiers et sous-dossiers
class Dossier : public IComposantFichier {
private:
    std::string nom;
    std::vector<std::shared_ptr<IComposantFichier>> enfants;
    
public:
    Dossier(const std::string& n) : nom(n) {}
    
    std::string getNom() const override { return nom; }
    
    // Taille = somme récursive des enfants
    int getTaille() const override {
        int total = 0;
        for (const auto& enfant : enfants) {
            total += enfant->getTaille();
        }
        return total;
    }
    
    void afficher(int indentation = 0) const override {
        std::cout << indent(indentation) << "📁 " << nom 
                  << "/ (" << getTaille() << " Ko)" << std::endl;
        for (const auto& enfant : enfants) {
            enfant->afficher(indentation + 1);
        }
    }
    
    void ajouter(std::shared_ptr<IComposantFichier> composant) override {
        enfants.push_back(composant);
    }
    
    void retirer(std::shared_ptr<IComposantFichier> composant) override {
        enfants.erase(
            std::remove(enfants.begin(), enfants.end(), composant),
            enfants.end()
        );
    }
    
    std::shared_ptr<IComposantFichier> getEnfant(int index) const override {
        if (index >= 0 && index < enfants.size()) {
            return enfants[index];
        }
        return nullptr;
    }
    
    bool estComposite() const override { return true; }
    
    int nombreEnfants() const { return enfants.size(); }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON COMPOSITE - DEMO          ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Créer la structure de fichiers
    auto racine = std::make_shared<Dossier>("home");
    
    // Dossier user
    auto user = std::make_shared<Dossier>("user");
    user->ajouter(std::make_shared<Fichier>("profile.txt", 2));
    user->ajouter(std::make_shared<Fichier>(".bashrc", 1));
    
    // Sous-dossier Documents
    auto documents = std::make_shared<Dossier>("Documents");
    documents->ajouter(std::make_shared<Fichier>("rapport.pdf", 1500));
    documents->ajouter(std::make_shared<Fichier>("notes.txt", 5));
    
    // Sous-sous-dossier Projets
    auto projets = std::make_shared<Dossier>("Projets");
    projets->ajouter(std::make_shared<Fichier>("main.cpp", 25));
    projets->ajouter(std::make_shared<Fichier>("Makefile", 2));
    
    documents->ajouter(projets);
    user->ajouter(documents);
    
    // Dossier Photos
    auto photos = std::make_shared<Dossier>("Photos");
    photos->ajouter(std::make_shared<Fichier>("vacances.jpg", 3500));
    photos->ajouter(std::make_shared<Fichier>("famille.png", 2800));
    user->ajouter(photos);
    
    racine->ajouter(user);
    
    // Afficher toute la structure
    std::cout << "\n=== Structure du système de fichiers ===" << std::endl;
    racine->afficher();
    
    // Le client traite tout uniformément
    std::cout << "\n=== Tailles calculées récursivement ===" << std::endl;
    std::cout << "Taille totale de " << racine->getNom() << ": " 
              << racine->getTaille() << " Ko" << std::endl;
    std::cout << "Taille de Documents: " << documents->getTaille() << " Ko" << std::endl;
    std::cout << "Taille de Photos: " << photos->getTaille() << " Ko" << std::endl;
    std::cout << "Taille de rapport.pdf: " << 1500 << " Ko" << std::endl;
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON COMPOSITE - DEMO          ║
╚═══════════════════════════════════════╝

=== Structure du système de fichiers ===
📁 home/ (7835 Ko)
  📁 user/ (7835 Ko)
    📄 profile.txt (2 Ko)
    📄 .bashrc (1 Ko)
    📁 Documents/ (1532 Ko)
      📄 rapport.pdf (1500 Ko)
      📄 notes.txt (5 Ko)
      📁 Projets/ (27 Ko)
        📄 main.cpp (25 Ko)
        📄 Makefile (2 Ko)
    📁 Photos/ (6300 Ko)
      📄 vacances.jpg (3500 Ko)
      📄 famille.png (2800 Ko)

=== Tailles calculées récursivement ===
Taille totale de home: 7835 Ko
Taille de Documents: 1532 Ko
Taille de Photos: 6300 Ko
Taille de rapport.pdf: 1500 Ko
```

---

## 🎨 Autre Exemple: Interface Graphique

```cpp
// Composant UI
class IComposantUI {
public:
    virtual void dessiner() = 0;
    virtual void deplacer(int dx, int dy) = 0;
    virtual ~IComposantUI() = default;
};

// Feuille: Bouton
class Bouton : public IComposantUI {
    int x, y;
    std::string texte;
public:
    Bouton(int x, int y, const std::string& t) : x(x), y(y), texte(t) {}
    void dessiner() override {
        std::cout << "Bouton [" << texte << "] à (" << x << "," << y << ")" << std::endl;
    }
    void deplacer(int dx, int dy) override { x += dx; y += dy; }
};

// Composite: Panneau
class Panneau : public IComposantUI {
    std::vector<std::shared_ptr<IComposantUI>> enfants;
    int x, y;
public:
    Panneau(int x, int y) : x(x), y(y) {}
    
    void ajouter(std::shared_ptr<IComposantUI> c) { enfants.push_back(c); }
    
    void dessiner() override {
        std::cout << "Panneau à (" << x << "," << y << ") {" << std::endl;
        for (auto& e : enfants) e->dessiner();
        std::cout << "}" << std::endl;
    }
    
    void deplacer(int dx, int dy) override {
        x += dx; y += dy;
        for (auto& e : enfants) e->deplacer(dx, dy);  // Récursif!
    }
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Uniformité** | Traite feuilles et composites identiquement |
| **Extensibilité** | Ajouter nouveaux types de composants facilement |
| **Récursivité** | Opérations propagées naturellement |
| **Simplicité client** | Le client ignore la structure interne |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Trop général** | Interface parfois trop générique |
| **Sécurité** | Difficile de restreindre les types d'enfants |
| **Complexité** | Les opérations de gestion dans l'interface |

---

## 🎯 Cas d'Utilisation

1. **Systèmes de fichiers** - Fichiers et dossiers
2. **GUI** - Widgets et conteneurs
3. **Documents** - Sections et paragraphes
4. **Organisations** - Employés et départements
5. **Graphiques** - Formes simples et groupes

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Décorateur](./Decorateur.md) | Structure similaire mais but différent |
| [Itérateur](../Comportemental/Iterateur.md) | Parcourir le composite |
| [Visiteur](../Comportemental/Visiteur.md) | Opérations sur les éléments |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Décorateur](./Decorateur.md)
