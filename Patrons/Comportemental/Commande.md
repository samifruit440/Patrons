# ⌨️ Patron Commande (Command)

> **Patron Comportemental** - Encapsule une requête comme un objet, permettant l'undo/redo et le découplage émetteur-exécuteur.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Encapsuler une requête comme un **objet**, permettant ainsi de:
- Découpler l'émetteur d'une requête de son exécuteur
- Supporter les **opérations annulables** (undo/redo)
- Mettre en file d'attente ou journaliser les requêtes

---

## 🎯 Problème Résolu

- Comment découpler l'objet qui invoque une opération de celui qui l'exécute?
- Comment supporter les opérations annulables (undo/redo)?
- Comment mettre des requêtes en file d'attente ou les journaliser?
- Comment composer des opérations simples en macro-commandes?

---

## 📊 Diagramme UML

```
┌──────────────────────────────────────────────────────────────────────────┐
│                         PATRON COMMANDE                                  │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────────────────┐           ┌────────────────────┐           │
│  │     Invocateur           │           │   «interface»      │           │
│  │     (Invoker)            │           │    ICommande       │           │
│  ├──────────────────────────┤           ├────────────────────┤           │
│  │ -commandes: ICommande[]  │◆──────── │ +executer()        │           │
│  │ -historique              │  compose  │ +annuler()         │           │
│  ├──────────────────────────┤           └──────────▲─────────┘           │
│  │ +executer(cmd)           │                      │                     │
│  │ +undo()                  │                      │                     │
│  │ +redo()                  │                      │                     │
│  └──────────────────────────┘                      │                     │
│                                                    │                     │
│                            ┌───────────────────────┴─────────────────┐   │
│                            │                       │                 │   │
│                    ┌───────┴────────┐  ┌───────────┴──┐  ┌───────────┴─┐ │
│                    │InsererCommande │  │ SupprimerCmd │  │  AnnulerCmd │ │
│                    ├────────────────┤  ├──────────────┤  ├─────────────┤ │
│                    │ -recepteur     │  │ -recepteur   │  │ -recepteur  │ │
│                    │ -texte         │  │ -position    │  │ -texte      │ │
│                    │ -position      │  │ -longueur    │  │ -position   │ │
│                    ├────────────────┤  ├──────────────┤  ├─────────────┤ │
│                    │ +executer()    │  │ +executer()  │  │ +executer() │ │
│                    │ +annuler()     │  │ +annuler()   │  │ +annuler()  │ │
│                    └────────────────┘  └──────────────┘  └─────────────┘ │
│                           ◆                   ◆                 ◆     │
│                           │                    │ dépendent de     │      │
│                           │                    │ (reçoivent)      │      │
│                           └────────┬───────────┴──────────────────┘      │
│                                    │                                     │
│                                    │                                     │
│                           ┌─────────────────┐                            │
│                           │  Récepteur      │                            │
│                           │  (Document)     │                            │
│                           ├─────────────────┤                            │
│                           │ -contenu        │                            │
│                           ├─────────────────┤                            │
│                           │ +insererTexte() │                            │
│                           │ +supprimerTexte │                            │
│                           │ +getContenu()   │                            │
│                           └─────────────────┘                            │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Flux d'Exécution

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLUX D'EXÉCUTION                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client              Invocateur           Commande   Récepteur │
│     │                     │                   │          │      │
│     │ crée commande       │                   │          │      │
│     ├────────────────────────────────────────►│          │      │
│     │                     │                   │          │      │
│     │ passe cmd à invoker │                   │          │      │
│     ├────────────────────►│                   │          │      │
│     │                     │                   │          │      │
│     │                     │ executer()        │          │      │
│     │                     ├──────────────────►│          │      │
│     │                     │                   │          │      │
│     │                     │                   │ action() │      │
│     │                     │                   ├─────────►│      │
│     │                     │                   │          │      │
│     │                     │ stock historique  │          │      │
│     │                     │◄──────────────────┤          │      │
│     │                     │                   │          │      │
│   UNDO:                   │                   │          │      │
│     │                     │ annuler()         │          │      │
│     │                     ├──────────────────►│          │      │
│     │                     │                   │ inverse()│      │
│     │                     │                   ├─────────►│      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎭 Les 4 Éléments du Patron

| Élément | Rôle | Exemple |
|---------|------|---------|
| **ICommande** | Interface commune avec `executer()` et `annuler()` | `class ICommande` |
| **Commande Concrète** | Encapsule l'action + référence au récepteur | `InsererCommande` |
| **Récepteur** | Exécute l'action réelle (logique métier) | `Document` |
| **Invocateur** | Déclenche les commandes, gère l'historique | `GestionnaireCommandes` |


## 💻 Exemple Complet

### 1. Interface Commande

```cpp
class ICommande {
public:
    virtual void executer() = 0;
    virtual void annuler() = 0;
    virtual ~ICommande() = default;
};
```

### 2. Récepteur (la logique métier)

```cpp
class Document {
    std::string contenu;
public:
    void insererTexte(const std::string& texte, size_t pos) {
        contenu.insert(pos, texte);
    }
    void supprimerTexte(size_t pos, size_t len) {
        contenu.erase(pos, len);
    }
    std::string getContenu() const { return contenu; }
};
```

### 3. Commande Concrète

```cpp
class InsererCommande : public ICommande {
    Document& doc;          // ← RÉCEPTEUR
    std::string texte;      // ← PARAMÈTRES
    size_t position;
    
public:
    InsererCommande(Document& d, const std::string& t, size_t pos)
        : doc(d), texte(t), position(pos) {}
    
    void executer() override {
        doc.insererTexte(texte, position);
    }
    
    void annuler() override {
        doc.supprimerTexte(position, texte.length());
    }
};
```

### 4. Invocateur (gère l'historique)

```cpp
class GestionnaireCommandes {
    std::stack<std::shared_ptr<ICommande>> undoStack;
    std::stack<std::shared_ptr<ICommande>> redoStack;
    
public:
    void executer(std::shared_ptr<ICommande> cmd) {
        cmd->executer();
        undoStack.push(cmd);
        while (!redoStack.empty()) redoStack.pop();  // Nouvelle action efface redo
    }
    
    void undo() {
        if (undoStack.empty()) return;
        auto cmd = undoStack.top(); undoStack.pop();
        cmd->annuler();
        redoStack.push(cmd);
    }
    
    void redo() {
        if (redoStack.empty()) return;
        auto cmd = redoStack.top(); redoStack.pop();
        cmd->executer();
        undoStack.push(cmd);
    }
};
```

### Utilisation

```cpp
Document doc;
GestionnaireCommandes gestionnaire;

// Exécuter
auto cmd1 = std::make_shared<InsererCommande>(doc, "Bonjour", 0);
gestionnaire.executer(cmd1);  // doc = "Bonjour"

auto cmd2 = std::make_shared<InsererCommande>(doc, " monde", 7);
gestionnaire.executer(cmd2);  // doc = "Bonjour monde"

// Undo
gestionnaire.undo();          // doc = "Bonjour"
gestionnaire.undo();          // doc = ""

// Redo
gestionnaire.redo();          // doc = "Bonjour"
```

---

## 🎯 Récepteur : Présent ou Absent ?

| Aspect | ✅ Avec Récepteur | ❌ Sans Récepteur |
|--------|-------------------|-------------------|
| **Undo/Redo** | Facile | Difficile |
| **Découplage** | Fort | Faible |
| **Exemple** | `InsererCmd(Document&)` | `PrintCmd()` |
| **Usage** | Éditeurs, transactions | Commandes simples |

**Recommandation :** Toujours utiliser un récepteur pour le support undo/redo.

---

## 🎮 Macro Commande (Bonus)

```cpp
class MacroCommande : public ICommande {
    std::vector<std::shared_ptr<ICommande>> commandes;
public:
    void ajouter(std::shared_ptr<ICommande> cmd) { commandes.push_back(cmd); }
    
    void executer() override {
        for (auto& cmd : commandes) cmd->executer();
    }
    
    void annuler() override {
        for (auto it = commandes.rbegin(); it != commandes.rend(); ++it)
            (*it)->annuler();  // Ordre inverse!
    }
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Découplage** | L'invocateur ne connaît pas la logique métier (ICommande) |
| **Undo/Redo** | Chaque commande encapsule l'action et son inverse facilement |
| **Extensibilité** | Ajouter nouvelles commandes sans modifier le code existant |
| **Composition** | Créer des macro-commandes complexes à partir de simples |
| **File d'attente** | Stocker et exécuter les commandes plus tard |
| **Journalisation** | Tracer toutes les opérations effectuées pour audit |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Nombre de classes** | Une classe par commande concrète (inflation de code) |
| **Mémoire** | Stockage de tout l'historique pour undo/redo |
| **Complexité** | Plus de niveaux d'indirection et d'abstractions |
| **Performance** | Allocations dynamiques pour chaque commande |

---

## 🎯 Cas d'Utilisation

- **Éditeurs de texte** — Undo/Redo
- **GUI** — Boutons, menus, raccourcis
- **Transactions** — Commit/Rollback
- **Jeux** — Actions du joueur

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Memento] | À venir ? |
| [Composite](../Structurel/Composite.md) | MacroCommandes |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ État](./Etat.md)
