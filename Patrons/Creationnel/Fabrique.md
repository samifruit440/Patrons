# 🏭 Patron Fabrique (Factory Method)

> **Patron Créationnel** - Définit une interface pour créer un objet, mais laisse les sous-classes décider quelle classe instancier.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Définir une **interface pour créer un objet**, mais laisser les **sous-classes décider** quelle classe instancier. La Fabrique délègue l'instanciation aux sous-classes.

---

## 🎯 Problème Résolu

- Comment créer des objets sans spécifier leur classe concrète?
- Comment découpler le code client de la création d'objets?
- Comment permettre l'extension des types créés?

---

## 📊 Diagramme UML

```
┌───────────────────────────────────────────────────────────────────────────┐
│                    PATRON FABRIQUE                                        │
├───────────────────────────────────────────────────────────────────────────┤
│                                                                           │
│    ┌──────────────────────┐                 ┌──────────────────────┐      │
│    │     «abstract»       │                 │      «interface»     │      │
│    │      Createur        │                 │       Produit        │      │
│    ├──────────────────────┤                 ├──────────────────────┤      │
│    │ +operation() {       │                 │ +utiliser()          │      │
│    │   p = creerProduit() │                 └──────────┬───────────┘      │
│    │   p.utiliser()       │                            │                  │
│    │ }                    │                            │                  │
│    │ +creerProduit()* ────┼──────creates──────────────►│                  │
│    └──────────┬───────────┘                            │                  │
│               │                                        │                  │
│    ┌──────────┴───────────┐                 ┌──────────┴───────────┐      │
│    │                      │                 │                      │      │
│    ▼                      ▼                 ▼                      ▼      │
│ ┌────────────────┐  ┌────────────────┐  ┌────────────┐       ┌───────────┐│
│ │CreateurConcretA│  │CreateurConcretB│  │ ProduitA   │       │ProduitB   ││
│ ├────────────────┤  ├────────────────┤  ├────────────┤       ├───────────┤│
│ │+creerProduit() │  │+creerProduit() │  │+utiliser() │       │+utiliser()││
│ │{new ProduitA}  │  │{new ProduitB}  │  └────────────┘       └───────────┘│
│ └────────────────┘  └────────────────┘                                    │
│                                                                           │
│   Chaque créateur concret décide quel produit instancier                  │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
```

---

## 🆚 Fabrique Simple vs Factory Method

```
┌─────────────────────────────────────────────────────────────────┐
│                FABRIQUE SIMPLE (pas un patron GoF)              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌────────────────────────┐                                    │
│   │    FabriqueSimple      │  Une seule classe avec des if/else │
│   ├────────────────────────┤                                    │
│   │ +creer(type) {         │                                    │
│   │   if (type == "A")     │                                    │
│   │     return new A();    │                                    │
│   │   if (type == "B")     │                                    │
│   │     return new B();    │                                    │
│   │ }                      │                                    │
│   └────────────────────────┘                                    │
│                                                                 │
│   ⚠️ Problème: Viole Open/Closed (modifier pour ajouter)        │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                 FACTORY METHOD (patron GoF)                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌────────────────┐        ┌────────────────┐                  │
│   │  CreateurA     │        │  CreateurB     │                  │
│   ├────────────────┤        ├────────────────┤                  │
│   │+creerProduit() │        │+creerProduit() │                  │
│   │{return new A}  │        │{return new B}  │                  │
│   └────────────────┘        └────────────────┘                  │
│                                                                 │
│   ✅ Chaque créateur = une classe, extensible                   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Produit

```cpp
#include <iostream>
#include <string>
#include <memory>

// Interface Produit
class IDocument {
public:
    virtual void ouvrir() = 0;
    virtual void fermer() = 0;
    virtual void sauvegarder() = 0;
    virtual std::string getType() const = 0;
    virtual ~IDocument() = default;
};
```

### Produits Concrets

```cpp
// Produit Concret: Document Word
class DocumentWord : public IDocument {
private:
    std::string contenu;
    
public:
    void ouvrir() override {
        std::cout << "📝 Ouverture document Word (.docx)" << std::endl;
        std::cout << "   → Chargement de la mise en forme OOXML" << std::endl;
    }
    
    void fermer() override {
        std::cout << "📝 Fermeture document Word" << std::endl;
    }
    
    void sauvegarder() override {
        std::cout << "💾 Sauvegarde au format .docx" << std::endl;
    }
    
    std::string getType() const override { return "Word"; }
};

// Produit Concret: Document PDF
class DocumentPDF : public IDocument {
public:
    void ouvrir() override {
        std::cout << "📄 Ouverture document PDF" << std::endl;
        std::cout << "   → Rendu des pages avec Poppler" << std::endl;
    }
    
    void fermer() override {
        std::cout << "📄 Fermeture document PDF" << std::endl;
    }
    
    void sauvegarder() override {
        std::cout << "💾 Sauvegarde au format .pdf" << std::endl;
    }
    
    std::string getType() const override { return "PDF"; }
};

// Produit Concret: Document Excel
class DocumentExcel : public IDocument {
public:
    void ouvrir() override {
        std::cout << "📊 Ouverture classeur Excel (.xlsx)" << std::endl;
        std::cout << "   → Chargement des feuilles et formules" << std::endl;
    }
    
    void fermer() override {
        std::cout << "📊 Fermeture classeur Excel" << std::endl;
    }
    
    void sauvegarder() override {
        std::cout << "💾 Sauvegarde au format .xlsx" << std::endl;
    }
    
    std::string getType() const override { return "Excel"; }
};
```

### Créateur Abstrait

```cpp
// Créateur Abstrait
class Application {
public:
    // FACTORY METHOD - à implémenter par les sous-classes
    virtual std::unique_ptr<IDocument> creerDocument() = 0;
    
    // Méthode template utilisant la factory method
    void nouveauDocument() {
        std::cout << "\n=== Création d'un nouveau document ===" << std::endl;
        
        // La factory method retourne le bon type
        auto doc = creerDocument();
        
        documents.push_back(std::move(doc));
        documents.back()->ouvrir();
        
        std::cout << "✓ Document " << documents.back()->getType() 
                  << " créé avec succès!" << std::endl;
    }
    
    void sauvegarderTous() {
        std::cout << "\n=== Sauvegarde de tous les documents ===" << std::endl;
        for (const auto& doc : documents) {
            doc->sauvegarder();
        }
    }
    
    void fermerTous() {
        std::cout << "\n=== Fermeture de tous les documents ===" << std::endl;
        for (const auto& doc : documents) {
            doc->fermer();
        }
        documents.clear();
    }
    
    virtual ~Application() = default;
    
protected:
    std::vector<std::unique_ptr<IDocument>> documents;
};
```

### Créateurs Concrets

```cpp
// Créateur Concret: Application Word
class ApplicationWord : public Application {
public:
    std::unique_ptr<IDocument> creerDocument() override {
        return std::make_unique<DocumentWord>();
    }
};

// Créateur Concret: Application PDF
class ApplicationPDF : public Application {
public:
    std::unique_ptr<IDocument> creerDocument() override {
        return std::make_unique<DocumentPDF>();
    }
};

// Créateur Concret: Application Excel
class ApplicationExcel : public Application {
public:
    std::unique_ptr<IDocument> creerDocument() override {
        return std::make_unique<DocumentExcel>();
    }
};
```

### Utilisation

```cpp
void testerApplication(Application& app, int nbDocs) {
    for (int i = 0; i < nbDocs; i++) {
        app.nouveauDocument();
    }
    app.sauvegarderTous();
    app.fermerTous();
}

int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║     PATRON FABRIQUE - DEMO            ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Le client utilise l'application sans connaître le type de document
    
    std::cout << "\n━━━ Microsoft Word ━━━" << std::endl;
    ApplicationWord word;
    testerApplication(word, 2);
    
    std::cout << "\n━━━ Adobe Acrobat ━━━" << std::endl;
    ApplicationPDF pdf;
    testerApplication(pdf, 1);
    
    std::cout << "\n━━━ Microsoft Excel ━━━" << std::endl;
    ApplicationExcel excel;
    testerApplication(excel, 2);
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║     PATRON FABRIQUE - DEMO            ║
╚═══════════════════════════════════════╝

━━━ Microsoft Word ━━━

=== Création d'un nouveau document ===
📝 Ouverture document Word (.docx)
   → Chargement de la mise en forme OOXML
✓ Document Word créé avec succès!

=== Création d'un nouveau document ===
📝 Ouverture document Word (.docx)
   → Chargement de la mise en forme OOXML
✓ Document Word créé avec succès!

=== Sauvegarde de tous les documents ===
💾 Sauvegarde au format .docx
💾 Sauvegarde au format .docx

=== Fermeture de tous les documents ===
📝 Fermeture document Word
📝 Fermeture document Word

━━━ Adobe Acrobat ━━━

=== Création d'un nouveau document ===
📄 Ouverture document PDF
   → Rendu des pages avec Poppler
✓ Document PDF créé avec succès!

=== Sauvegarde de tous les documents ===
💾 Sauvegarde au format .pdf

=== Fermeture de tous les documents ===
📄 Fermeture document PDF

━━━ Microsoft Excel ━━━

=== Création d'un nouveau document ===
📊 Ouverture classeur Excel (.xlsx)
   → Chargement des feuilles et formules
✓ Document Excel créé avec succès!
...
```

---

## 🔧 Variante: Factory Method Paramétré

```cpp
class FabriqueVehicule {
public:
    enum Type { VOITURE, MOTO, CAMION };
    
    // Factory method avec paramètre
    virtual std::unique_ptr<IVehicule> creer(Type type) {
        switch (type) {
            case VOITURE: return creerVoiture();
            case MOTO:    return creerMoto();
            case CAMION:  return creerCamion();
        }
        return nullptr;
    }
    
protected:
    virtual std::unique_ptr<IVehicule> creerVoiture() = 0;
    virtual std::unique_ptr<IVehicule> creerMoto() = 0;
    virtual std::unique_ptr<IVehicule> creerCamion() = 0;
};

class FabriqueVehiculeSport : public FabriqueVehicule {
protected:
    std::unique_ptr<IVehicule> creerVoiture() override {
        return std::make_unique<VoitureSport>();
    }
    // ...
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Découplage** | Le client ne connaît pas les classes concrètes |
| **Open/Closed** | Ajouter des produits sans modifier le code existant |
| **Single Responsibility** | La création est séparée de l'utilisation |
| **Flexibilité** | Peut retourner des objets existants (pool) |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Beaucoup de classes** | Une classe par produit + une par créateur |
| **Complexité** | Plus de code que la création directe |
| **Héritage** | Requiert l'héritage pour les créateurs |

---

## 🎯 Cas d'Utilisation

1. **Frameworks** - Création d'objets spécifiques à l'application
2. **Plugins** - Charger dynamiquement des implémentations
3. **Tests** - Injecter des mocks via une fabrique
4. **Multi-plateforme** - Créer des objets selon l'OS

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Fabrique Abstraite](./FabriqueAbstraite.md) | Familles de produits |
| [Prototype](./Singleton.md) | Alternative: cloner au lieu de créer |
| [Template Method](../Comportemental/PatronMethode.md) | Factory method EST un template method |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Fabrique Abstraite](./FabriqueAbstraite.md)
