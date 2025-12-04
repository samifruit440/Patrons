# ⌨️ Patron Commande (Command)

> **Patron Comportemental** - Encapsule une requête comme un objet, permettant de paramétrer les clients, mettre en file d'attente les requêtes et supporter les opérations annulables.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Encapsuler une requête comme un **objet**, permettant ainsi de:
- Paramétrer les clients avec différentes requêtes
- Mettre en file d'attente ou journaliser les requêtes
- Supporter les **opérations annulables** (undo/redo)

---

## 🎯 Problème Résolu

- Comment découpler l'émetteur d'une requête de son exécuteur?
- Comment implémenter undo/redo?
- Comment mettre des requêtes en file d'attente?
- Comment journaliser les opérations?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON COMMANDE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────┐      ┌──────────────────┐                     │
│  │   Invocateur │      │   «interface»    │                     │
│  ├──────────────┤      │     Commande     │                     │
│  │ -commande    │─────►├──────────────────┤                     │
│  ├──────────────┤      │ +executer()      │                     │
│  │ +setCommande()│     │ +annuler()       │                     │
│  │ +executer()  │      └────────┬─────────┘                     │
│  └──────────────┘               │                               │
│                                 │                               │
│                    ┌────────────┴────────────┐                  │
│                    │                         │                  │
│           ┌────────▼────────┐      ┌─────────▼────────┐         │
│           │CommandeConcreteA│      │CommandeConcreteB │         │
│           ├─────────────────┤      ├──────────────────┤         │
│           │ -recepteur      │      │ -recepteur       │         │
│           │ -etatPrecedent  │      │ -etatPrecedent   │         │
│           ├─────────────────┤      ├──────────────────┤         │
│           │ +executer() {   │      │ +executer()      │         │
│           │   sauverEtat()  │      │ +annuler()       │         │
│           │   recepteur.    │      └──────────────────┘         │
│           │     action()    │                │                  │
│           │ }               │                │                  │
│           │ +annuler() {    │                ▼                  │
│           │   restaurer()   │      ┌──────────────────┐         │
│           │ }               │      │    Recepteur     │         │
│           └─────────────────┘      ├──────────────────┤         │
│                    │               │ +action()        │         │
│                    │               │ +autreAction()   │         │
│                    └──────────────►└──────────────────┘         │
│                        utilise                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Flux d'Exécution

```
┌─────────────────────────────────────────────────────────────────┐
│                    SÉQUENCE COMMANDE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client      Invocateur     Commande       Récepteur           │
│     │             │             │              │                │
│     │ créer cmd   │             │              │                │
│     │────────────────────────►  │              │                │
│     │             │             │              │                │
│     │ setCommande │             │              │                │
│     │────────────►│             │              │                │
│     │             │             │              │                │
│     │ appuyer()   │             │              │                │
│     │────────────►│             │              │                │
│     │             │ executer()  │              │                │
│     │             │────────────►│              │                │
│     │             │             │   action()   │                │
│     │             │             │─────────────►│                │
│     │             │             │              │                │
│     │ annuler()   │             │              │                │
│     │────────────►│             │              │                │
│     │             │  annuler()  │              │                │
│     │             │────────────►│              │                │
│     │             │             │ restaurer()  │                │
│     │             │             │─────────────►│                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code

### Interface Commande

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <stack>
#include <memory>

// Interface Commande
class ICommande {
public:
    virtual void executer() = 0;
    virtual void annuler() = 0;
    virtual std::string getDescription() const = 0;
    virtual ~ICommande() = default;
};
```

### Récepteur (Document)

```cpp
// Récepteur: Document texte
class Document {
private:
    std::string contenu;
    
public:
    void insererTexte(const std::string& texte, size_t position) {
        if (position > contenu.length()) position = contenu.length();
        contenu.insert(position, texte);
        std::cout << "   ✏️  Inséré '" << texte << "' à position " << position << std::endl;
    }
    
    void supprimerTexte(size_t position, size_t longueur) {
        if (position < contenu.length()) {
            contenu.erase(position, longueur);
            std::cout << "   🗑️  Supprimé " << longueur << " caractères à position " << position << std::endl;
        }
    }
    
    void remplacerTexte(size_t position, size_t longueur, const std::string& nouveau) {
        supprimerTexte(position, longueur);
        insererTexte(nouveau, position);
    }
    
    std::string getContenu() const { return contenu; }
    
    void afficher() const {
        std::cout << "   📄 Document: \"" << contenu << "\"" << std::endl;
    }
};
```

### Commandes Concrètes

```cpp
// Commande: Insérer du texte
class InsererCommande : public ICommande {
private:
    Document& document;
    std::string texte;
    size_t position;
    
public:
    InsererCommande(Document& doc, const std::string& t, size_t pos)
        : document(doc), texte(t), position(pos) {}
    
    void executer() override {
        document.insererTexte(texte, position);
    }
    
    void annuler() override {
        document.supprimerTexte(position, texte.length());
    }
    
    std::string getDescription() const override {
        return "Insérer '" + texte + "'";
    }
};

// Commande: Supprimer du texte
class SupprimerCommande : public ICommande {
private:
    Document& document;
    size_t position;
    size_t longueur;
    std::string texteSupprime;  // Pour l'annulation
    
public:
    SupprimerCommande(Document& doc, size_t pos, size_t len)
        : document(doc), position(pos), longueur(len) {}
    
    void executer() override {
        // Sauvegarder le texte avant suppression
        texteSupprime = document.getContenu().substr(position, longueur);
        document.supprimerTexte(position, longueur);
    }
    
    void annuler() override {
        document.insererTexte(texteSupprime, position);
    }
    
    std::string getDescription() const override {
        return "Supprimer " + std::to_string(longueur) + " caractères";
    }
};

// Commande: Remplacer du texte
class RemplacerCommande : public ICommande {
private:
    Document& document;
    size_t position;
    size_t longueur;
    std::string nouveauTexte;
    std::string ancienTexte;  // Pour l'annulation
    
public:
    RemplacerCommande(Document& doc, size_t pos, size_t len, const std::string& nouveau)
        : document(doc), position(pos), longueur(len), nouveauTexte(nouveau) {}
    
    void executer() override {
        ancienTexte = document.getContenu().substr(position, longueur);
        document.remplacerTexte(position, longueur, nouveauTexte);
    }
    
    void annuler() override {
        document.remplacerTexte(position, nouveauTexte.length(), ancienTexte);
    }
    
    std::string getDescription() const override {
        return "Remplacer '" + ancienTexte + "' par '" + nouveauTexte + "'";
    }
};
```

### Gestionnaire de Commandes (Invocateur)

```cpp
// Invocateur avec historique undo/redo
class GestionnaireCommandes {
private:
    std::stack<std::shared_ptr<ICommande>> historiqueUndo;
    std::stack<std::shared_ptr<ICommande>> historiqueRedo;
    
public:
    void executerCommande(std::shared_ptr<ICommande> commande) {
        std::cout << "▶️  Exécution: " << commande->getDescription() << std::endl;
        commande->executer();
        historiqueUndo.push(commande);
        
        // Vider le redo après une nouvelle commande
        while (!historiqueRedo.empty()) {
            historiqueRedo.pop();
        }
    }
    
    void annuler() {
        if (historiqueUndo.empty()) {
            std::cout << "⚠️  Rien à annuler!" << std::endl;
            return;
        }
        
        auto commande = historiqueUndo.top();
        historiqueUndo.pop();
        
        std::cout << "↩️  Annulation: " << commande->getDescription() << std::endl;
        commande->annuler();
        historiqueRedo.push(commande);
    }
    
    void retablir() {
        if (historiqueRedo.empty()) {
            std::cout << "⚠️  Rien à rétablir!" << std::endl;
            return;
        }
        
        auto commande = historiqueRedo.top();
        historiqueRedo.pop();
        
        std::cout << "↪️  Rétablissement: " << commande->getDescription() << std::endl;
        commande->executer();
        historiqueUndo.push(commande);
    }
    
    void afficherHistorique() const {
        std::cout << "📜 Historique: " << historiqueUndo.size() << " actions"
                  << " (redo: " << historiqueRedo.size() << ")" << std::endl;
    }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON COMMANDE - DEMO           ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Créer le document et le gestionnaire
    Document doc;
    GestionnaireCommandes gestionnaire;
    
    // Exécuter des commandes
    std::cout << "\n=== Édition du document ===" << std::endl;
    
    gestionnaire.executerCommande(
        std::make_shared<InsererCommande>(doc, "Bonjour", 0));
    doc.afficher();
    
    gestionnaire.executerCommande(
        std::make_shared<InsererCommande>(doc, " le monde!", 7));
    doc.afficher();
    
    gestionnaire.executerCommande(
        std::make_shared<RemplacerCommande>(doc, 8, 5, "tout"));
    doc.afficher();
    
    gestionnaire.afficherHistorique();
    
    // Annuler
    std::cout << "\n=== Annulations ===" << std::endl;
    
    gestionnaire.annuler();
    doc.afficher();
    
    gestionnaire.annuler();
    doc.afficher();
    
    gestionnaire.afficherHistorique();
    
    // Rétablir
    std::cout << "\n=== Rétablissements ===" << std::endl;
    
    gestionnaire.retablir();
    doc.afficher();
    
    gestionnaire.retablir();
    doc.afficher();
    
    gestionnaire.afficherHistorique();
    
    // Nouvelle commande après undo (efface le redo)
    std::cout << "\n=== Nouvelle commande ===" << std::endl;
    gestionnaire.annuler();  // Undo une action
    gestionnaire.executerCommande(
        std::make_shared<InsererCommande>(doc, " chers amis", 17));
    doc.afficher();
    
    gestionnaire.retablir();  // Devrait échouer
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON COMMANDE - DEMO           ║
╚═══════════════════════════════════════╝

=== Édition du document ===
▶️  Exécution: Insérer 'Bonjour'
   ✏️  Inséré 'Bonjour' à position 0
   📄 Document: "Bonjour"
▶️  Exécution: Insérer ' le monde!'
   ✏️  Inséré ' le monde!' à position 7
   📄 Document: "Bonjour le monde!"
▶️  Exécution: Remplacer 'le mo' par 'tout'
   🗑️  Supprimé 5 caractères à position 8
   ✏️  Inséré 'tout' à position 8
   📄 Document: "Bonjour toutonde!"
📜 Historique: 3 actions (redo: 0)

=== Annulations ===
↩️  Annulation: Remplacer 'le mo' par 'tout'
   🗑️  Supprimé 4 caractères à position 8
   ✏️  Inséré 'le mo' à position 8
   📄 Document: "Bonjour le monde!"
↩️  Annulation: Insérer ' le monde!'
   🗑️  Supprimé 10 caractères à position 7
   📄 Document: "Bonjour"
📜 Historique: 1 actions (redo: 2)

=== Rétablissements ===
↪️  Rétablissement: Insérer ' le monde!'
   ✏️  Inséré ' le monde!' à position 7
   📄 Document: "Bonjour le monde!"
↪️  Rétablissement: Remplacer 'le mo' par 'tout'
   🗑️  Supprimé 5 caractères à position 8
   ✏️  Inséré 'tout' à position 8
   📄 Document: "Bonjour toutonde!"
📜 Historique: 3 actions (redo: 0)

=== Nouvelle commande ===
...
```

---

## 🎮 Exemple: Macro Commande

```cpp
// Commande composée (macro)
class MacroCommande : public ICommande {
private:
    std::vector<std::shared_ptr<ICommande>> commandes;
    std::string nom;
    
public:
    MacroCommande(const std::string& n) : nom(n) {}
    
    void ajouter(std::shared_ptr<ICommande> cmd) {
        commandes.push_back(cmd);
    }
    
    void executer() override {
        for (auto& cmd : commandes) {
            cmd->executer();
        }
    }
    
    void annuler() override {
        // Annuler en ordre inverse
        for (auto it = commandes.rbegin(); it != commandes.rend(); ++it) {
            (*it)->annuler();
        }
    }
    
    std::string getDescription() const override {
        return "Macro: " + nom;
    }
};

// Utilisation
auto macro = std::make_shared<MacroCommande>("Formater titre");
macro->ajouter(std::make_shared<InsererCommande>(doc, "# ", 0));
macro->ajouter(std::make_shared<InsererCommande>(doc, " #", 10));
gestionnaire.executerCommande(macro);
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Découplage** | L'émetteur ignore le récepteur |
| **Undo/Redo** | Facile à implémenter |
| **Extensibilité** | Nouvelles commandes sans modifier le code |
| **Composition** | Macros de commandes |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Complexité** | Beaucoup de petites classes |
| **Mémoire** | Stockage de l'historique |
| **État** | Gérer l'état pour l'annulation |

---

## 🎯 Cas d'Utilisation

1. **Éditeurs de texte** - Undo/Redo
2. **Transactions** - Commit/Rollback
3. **GUI** - Boutons, menus, raccourcis
4. **Tâches planifiées** - Files de commandes
5. **Jeux** - Actions du joueur

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Memento](./Etat.md) | Sauvegarder l'état pour annulation |
| [Composite](../Structurel/Composite.md) | MacroCommandes |
| [Prototype](../Creationnel/Singleton.md) | Cloner des commandes |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ État](./Etat.md)
