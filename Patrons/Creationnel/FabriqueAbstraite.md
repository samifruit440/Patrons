# 🏭 Patron Fabrique Abstraite (Abstract Factory)

> **Patron Créationnel** - Fournit une interface pour créer des familles d'objets liés sans spécifier leurs classes concrètes.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Fournir une interface pour créer des **familles d'objets apparentés** ou dépendants sans spécifier leurs classes concrètes.

---

## 🎯 Problème Résolu

- Comment créer des objets qui vont ensemble (famille cohérente)?
- Comment garantir la compatibilité entre objets créés?
- Comment changer de famille d'objets facilement?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         PATRON FABRIQUE ABSTRAITE                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│         ┌───────────────────────────┐                                           │
│         │     «interface»           │                                           │
│         │   FabriqueAbstraite       │                                           │
│         ├───────────────────────────┤                                           │
│         │ +creerProduitA() : IA     │                                           │
│         │ +creerProduitB() : IB     │                                           │
│         └─────────────┬─────────────┘                                           │
│                       │                                                         │
│         ┌─────────────┴─────────────┐                                           │
│         │                           │                                           │
│    ┌────▼───────────────┐    ┌──────▼──────────────┐                            │
│    │  FabriqueConcrete1 │    │  FabriqueConcrete2  │                            │
│    ├────────────────────┤    ├─────────────────────┤                            │
│    │ +creerProduitA()   │    │ +creerProduitA()    │                            │
│    │   {new ProduitA1}  │    │   {new ProduitA2}   │                            │
│    │ +creerProduitB()   │    │ +creerProduitB()    │                            │
│    │   {new ProduitB1}  │    │   {new ProduitB2}   │                            │
│    └─────────┬──────────┘    └──────────┬──────────┘                            │
│              │ crée                     │ crée                                  │
│              ▼                          ▼                                       │
│                                                                                 │
│   ┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐     │
│   │                        INTERFACES PRODUITS                            │     │
│   │                                                                       │     │
│   │       ┌──────────────────┐              ┌──────────────────┐          │     │
│   │       │   «interface»    │              │   «interface»    │          │     │
│   │       │       IA         │              │       IB         │          │     │
│   │       ├──────────────────┤              ├──────────────────┤          │     │
│   │       │ +operationA()    │              │ +operationB()    │          │     │
│   │       └────────┬─────────┘              └────────┬─────────┘          │     │
│   │                │                                 │                    │     │
│   │       ┌────────┴────────┐               ┌────────┴────────┐           │     │
│   │       │                 │               │                 │           │     │
│   │  ┌────▼─────┐     ┌─────▼────┐     ┌────▼─────┐     ┌─────▼────┐      │     │
│   │  │ProduitA1 │     │ProduitA2 │     │ProduitB1 │     │ProduitB2 │      │     │
│   │  ├──────────┤     ├──────────┤     ├──────────┤     ├──────────┤      │     │
│   │  │+operat...│     │+operat...│     │+operat...│     │+operat...│      │     │
│   │  └──────────┘     └──────────┘     └──────────┘     └──────────┘      │     │
│   │   (Famille 1)      (Famille 2)      (Famille 1)      (Famille 2)      │     │
│   └ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘     │
│                                                                                 │
│   Les produits d'une même famille sont conçus pour fonctionner ensemble         │
│   Famille 1: ProduitA1 + ProduitB1  |  Famille 2: ProduitA2 + ProduitB2         │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 🆚 Factory Method vs Abstract Factory

```
┌─────────────────────────────────────────────────────────────────┐
│            FACTORY METHOD vs ABSTRACT FACTORY                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   FACTORY METHOD                 ABSTRACT FACTORY               │
│   ┌────────────────┐            ┌────────────────┐              │
│   │   Createur     │            │   Fabrique     │              │
│   ├────────────────┤            ├────────────────┤              │
│   │ +creerProduit()│            │+creerProduitA()│              │
│   └────────────────┘            │+creerProduitB()│              │
│          │                      │+creerProduitC()│              │
│          ▼                      └────────────────┘              │
│   ┌────────────┐                       │                        │
│   │  1 produit │                       ▼                        │
│   └────────────┘               ┌────────────────┐               │
│                                │ FAMILLE de     │               │
│                                │ produits liés  │               │
│                                └────────────────┘               │
│                                                                 │
│   • UN type de produit          • PLUSIEURS types de produits   │
│   • Via héritage               • Via composition                │
│   • Une méthode                • Plusieurs méthodes             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interfaces Produits

```cpp
#include <iostream>
#include <string>
#include <memory>

// Interface Produit A: Bouton
class IBouton {
public:
    virtual void afficher() const = 0;
    virtual void cliquer() = 0;
    virtual ~IBouton() = default;
};

// Interface Produit B: Case à cocher
class ICheckbox {
public:
    virtual void afficher() const = 0;
    virtual void cocher() = 0;
    virtual bool estCoche() const = 0;
    virtual ~ICheckbox() = default;
};

// Interface Produit C: Champ de texte
class ITextField {
public:
    virtual void afficher() const = 0;
    virtual void setTexte(const std::string& texte) = 0;
    virtual std::string getTexte() const = 0;
    virtual ~ITextField() = default;
};
```

### Famille Windows

```cpp
// ═══════════════════════════════════════
// FAMILLE WINDOWS
// ═══════════════════════════════════════

class BoutonWindows : public IBouton {
public:
    void afficher() const override {
        std::cout << "[🪟 Windows Button]" << std::endl;
    }
    void cliquer() override {
        std::cout << "   → Clic Windows avec effet 3D" << std::endl;
    }
};

class CheckboxWindows : public ICheckbox {
    bool coche = false;
public:
    void afficher() const override {
        std::cout << "[🪟 Windows Checkbox] " 
                  << (coche ? "☑" : "☐") << std::endl;
    }
    void cocher() override {
        coche = !coche;
        std::cout << "   → Toggle Windows checkbox" << std::endl;
    }
    bool estCoche() const override { return coche; }
};

class TextFieldWindows : public ITextField {
    std::string texte;
public:
    void afficher() const override {
        std::cout << "[🪟 Windows TextField] |" << texte << "|" << std::endl;
    }
    void setTexte(const std::string& t) override {
        texte = t;
        std::cout << "   → Input Windows: " << t << std::endl;
    }
    std::string getTexte() const override { return texte; }
};
```

### Famille macOS

```cpp
// ═══════════════════════════════════════
// FAMILLE MACOS
// ═══════════════════════════════════════

class BoutonMac : public IBouton {
public:
    void afficher() const override {
        std::cout << "[🍎 Mac Button]" << std::endl;
    }
    void cliquer() override {
        std::cout << "   → Clic Mac avec effet aqua" << std::endl;
    }
};

class CheckboxMac : public ICheckbox {
    bool coche = false;
public:
    void afficher() const override {
        std::cout << "[🍎 Mac Checkbox] " 
                  << (coche ? "✓" : "○") << std::endl;
    }
    void cocher() override {
        coche = !coche;
        std::cout << "   → Toggle Mac checkbox avec animation" << std::endl;
    }
    bool estCoche() const override { return coche; }
};

class TextFieldMac : public ITextField {
    std::string texte;
public:
    void afficher() const override {
        std::cout << "[🍎 Mac TextField] ⟨" << texte << "⟩" << std::endl;
    }
    void setTexte(const std::string& t) override {
        texte = t;
        std::cout << "   → Input Mac avec autocomplete: " << t << std::endl;
    }
    std::string getTexte() const override { return texte; }
};
```

### Famille Linux

```cpp
// ═══════════════════════════════════════
// FAMILLE LINUX (GTK)
// ═══════════════════════════════════════

class BoutonLinux : public IBouton {
public:
    void afficher() const override {
        std::cout << "[🐧 Linux Button]" << std::endl;
    }
    void cliquer() override {
        std::cout << "   → Clic Linux GTK" << std::endl;
    }
};

class CheckboxLinux : public ICheckbox {
    bool coche = false;
public:
    void afficher() const override {
        std::cout << "[🐧 Linux Checkbox] " 
                  << (coche ? "[X]" : "[ ]") << std::endl;
    }
    void cocher() override {
        coche = !coche;
        std::cout << "   → Toggle Linux checkbox" << std::endl;
    }
    bool estCoche() const override { return coche; }
};

class TextFieldLinux : public ITextField {
    std::string texte;
public:
    void afficher() const override {
        std::cout << "[🐧 Linux TextField] [" << texte << "]" << std::endl;
    }
    void setTexte(const std::string& t) override {
        texte = t;
        std::cout << "   → Input Linux: " << t << std::endl;
    }
    std::string getTexte() const override { return texte; }
};
```

### Fabrique Abstraite et Concrètes

```cpp
// ═══════════════════════════════════════
// FABRIQUE ABSTRAITE
// ═══════════════════════════════════════

class IFabriqueUI {
public:
    virtual std::unique_ptr<IBouton> creerBouton() = 0;
    virtual std::unique_ptr<ICheckbox> creerCheckbox() = 0;
    virtual std::unique_ptr<ITextField> creerTextField() = 0;
    virtual std::string getNomPlateforme() const = 0;
    virtual ~IFabriqueUI() = default;
};

// Fabrique Windows
class FabriqueWindows : public IFabriqueUI {
public:
    std::unique_ptr<IBouton> creerBouton() override {
        return std::make_unique<BoutonWindows>();
    }
    std::unique_ptr<ICheckbox> creerCheckbox() override {
        return std::make_unique<CheckboxWindows>();
    }
    std::unique_ptr<ITextField> creerTextField() override {
        return std::make_unique<TextFieldWindows>();
    }
    std::string getNomPlateforme() const override { return "Windows"; }
};

// Fabrique macOS
class FabriqueMac : public IFabriqueUI {
public:
    std::unique_ptr<IBouton> creerBouton() override {
        return std::make_unique<BoutonMac>();
    }
    std::unique_ptr<ICheckbox> creerCheckbox() override {
        return std::make_unique<CheckboxMac>();
    }
    std::unique_ptr<ITextField> creerTextField() override {
        return std::make_unique<TextFieldMac>();
    }
    std::string getNomPlateforme() const override { return "macOS"; }
};

// Fabrique Linux
class FabriqueLinux : public IFabriqueUI {
public:
    std::unique_ptr<IBouton> creerBouton() override {
        return std::make_unique<BoutonLinux>();
    }
    std::unique_ptr<ICheckbox> creerCheckbox() override {
        return std::make_unique<CheckboxLinux>();
    }
    std::unique_ptr<ITextField> creerTextField() override {
        return std::make_unique<TextFieldLinux>();
    }
    std::string getNomPlateforme() const override { return "Linux"; }
};
```

### Application Client

```cpp
// ═══════════════════════════════════════
// APPLICATION (CLIENT)
// ═══════════════════════════════════════

class Application {
private:
    std::unique_ptr<IBouton> boutonOK;
    std::unique_ptr<IBouton> boutonAnnuler;
    std::unique_ptr<ICheckbox> checkboxAccepter;
    std::unique_ptr<ITextField> champNom;
    
public:
    // L'application reçoit la fabrique - ne sait pas quelle famille
    Application(IFabriqueUI& fabrique) {
        std::cout << "\n═══ Création UI pour " 
                  << fabrique.getNomPlateforme() << " ═══" << std::endl;
        
        boutonOK = fabrique.creerBouton();
        boutonAnnuler = fabrique.creerBouton();
        checkboxAccepter = fabrique.creerCheckbox();
        champNom = fabrique.creerTextField();
    }
    
    void afficherInterface() {
        std::cout << "\n--- Interface ---" << std::endl;
        champNom->afficher();
        checkboxAccepter->afficher();
        boutonOK->afficher();
        boutonAnnuler->afficher();
    }
    
    void simulerInteraction() {
        std::cout << "\n--- Interactions ---" << std::endl;
        champNom->setTexte("Jean Dupont");
        checkboxAccepter->cocher();
        boutonOK->cliquer();
    }
};
```

### Utilisation

```cpp
// Sélecteur de fabrique selon l'OS
std::unique_ptr<IFabriqueUI> getFabrique() {
    #ifdef _WIN32
        return std::make_unique<FabriqueWindows>();
    #elif __APPLE__
        return std::make_unique<FabriqueMac>();
    #else
        return std::make_unique<FabriqueLinux>();
    #endif
}

int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║    PATRON FABRIQUE ABSTRAITE - DEMO   ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Démonstration avec chaque fabrique
    
    // Windows
    FabriqueWindows fabWin;
    Application appWin(fabWin);
    appWin.afficherInterface();
    appWin.simulerInteraction();
    
    // macOS
    FabriqueMac fabMac;
    Application appMac(fabMac);
    appMac.afficherInterface();
    appMac.simulerInteraction();
    
    // Linux
    FabriqueLinux fabLinux;
    Application appLinux(fabLinux);
    appLinux.afficherInterface();
    appLinux.simulerInteraction();
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║    PATRON FABRIQUE ABSTRAITE - DEMO   ║
╚═══════════════════════════════════════╝

═══ Création UI pour Windows ═══

--- Interface ---
[🪟 Windows TextField] ||
[🪟 Windows Checkbox] ☐
[🪟 Windows Button]
[🪟 Windows Button]

--- Interactions ---
   → Input Windows: Jean Dupont
   → Toggle Windows checkbox
   → Clic Windows avec effet 3D

═══ Création UI pour macOS ═══

--- Interface ---
[🍎 Mac TextField] ⟨⟩
[🍎 Mac Checkbox] ○
[🍎 Mac Button]
[🍎 Mac Button]

--- Interactions ---
   → Input Mac avec autocomplete: Jean Dupont
   → Toggle Mac checkbox avec animation
   → Clic Mac avec effet aqua
...
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Cohérence** | Produits d'une famille sont compatibles |
| **Isolation** | Le client ne connaît que les interfaces |
| **Échange facile** | Changer de famille = changer de fabrique |
| **Open/Closed** | Nouvelles familles sans modifier le code |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Complexité** | Beaucoup d'interfaces et classes |
| **Rigidité** | Ajouter un nouveau produit = modifier toutes les fabriques |
| **Over-engineering** | Parfois trop complexe pour de petits projets |

---

## 🎯 Cas d'Utilisation

1. **UI multi-plateforme** - Windows, macOS, Linux
2. **Thèmes** - Clair/Sombre avec composants cohérents
3. **Jeux vidéo** - Familles d'ennemis, d'armes, etc.
4. **Persistence** - Famille SQL vs NoSQL

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Fabrique](./Fabrique.md) | Version simplifiée (un produit) |
| [Singleton](./Singleton.md) | La fabrique est souvent un singleton |
| [Prototype]| À venir ?|

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Singleton](./Singleton.md)
