# 1️⃣ Patron Singleton

> **Patron Créationnel** - Garantit qu'une classe n'a qu'une seule instance et fournit un point d'accès global à celle-ci.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Garantir qu'une classe n'a **qu'une seule instance** et fournir un **point d'accès global** à celle-ci.

---

## 🎯 Problème Résolu

- Comment s'assurer qu'une classe n'a qu'une seule instance?
- Comment fournir un point d'accès global sans variable globale?
- Comment contrôler l'accès à une ressource partagée (logger, configuration, registry)?

---

## 📊 Diagramme UML de Base

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON SINGLETON                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌───────────────────────────┐                │
│                    │        Singleton          │                │
│                    ├───────────────────────────┤                │
│                    │ -instance: Singleton      │ ◄── statique   │
│                    │ -données                  │                │
│                    ├───────────────────────────┤                │
│                    │ -Singleton() { }          │ ◄── privé      │
│                    │ +getInstance(): Singleton │ ◄── statique   │
│                    │ +operation()              │                │
│                    └───────────────────────────┘                │
│                              ▲                                  │
│                              │                                  │
│                       retourne                                  │
│                              │                                  │
│         ┌────────────────────┴────────────────────┐             │
│         │                                         │             │
│    ┌────▼────┐                              ┌─────▼────┐        │
│    │ Client1 │                              │ Client2  │        │
│    └─────────┘                              └──────────┘        │
│                                                                 │
│    Singleton::getInstance()                                     │
│    → Toujours la MÊME instance                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📋 Différentes Implémentations

Le patron Singleton peut être implémenté de plusieurs façons. Voici les trois approches principales :

### 1. Lazy Loading (création à la demande)

```cpp
class Singleton {
    static Singleton* instance;
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();  // Créé au premier appel
        }
        return instance;
    }
private:
    Singleton() = default;
};

Singleton* Singleton::instance = nullptr;
```

**Avantage :** Créé seulement si utilisé.  
**Inconvénient :** ⚠️ Pas thread-safe!

### 2. Meyer's Singleton (C++11 - Recommandé)

```cpp
class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance;  // Créé une seule fois garantie
        return instance;
    }
private:
    Singleton() = default;
};
```

**Avantage :** Thread-safe automatiquement (C++11), simple, élégant.  
**Idéal pour :** La plupart des cas modernes.

### 3. Eager Loading (création au démarrage)

```cpp
class Singleton {
    static Singleton instance;
public:
    static Singleton* getInstance() {
        return &instance;
    }
private:
    Singleton() = default;
};

Singleton Singleton::instance;  // Créé au chargement de la classe
```

**Avantage :** Thread-safe, simple.  
**Inconvénient :** Créé même s'il n'est pas utilisé.

---

## 💻 Exemple Pratique : Invoker + Registry

Voici un exemple tiré du cours LOG2400 montrant comment utiliser le Singleton pour gérer l'historique des commandes (undo/redo).

### Types de Base

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <list>
#include <map>
#include <ostream>

namespace PolyIcone3D {

// Alias et interfaces
class CommandAbs;
using CmdPtr = std::shared_ptr<CommandAbs>;
using CmdContainer = std::list<CmdPtr>;

// Interface pour les commandes
class CommandAbs {
public:
    virtual ~CommandAbs() = default;
    virtual void execute() = 0;
    virtual void cancel() = 0;
};
```

### Classe Invoker (Singleton)

L'Invoker gère l'exécution des commandes avec undo/redo. Il est **un Singleton** pour garantir une seule instance globale.

```cpp
class Invoker {
public:
    virtual ~Invoker() = default;
    
    // Point d'accès au Singleton
    static Invoker* getInstance() {
        if (!m_instance) {
            m_instance.reset(new Invoker());
        }
        return m_instance.get();
    }
    
    // Exécute une commande et l'empile pour undo
    virtual void execute(CmdPtr& cmd) {
        cmd->execute();
        m_cmdDone.push_back(cmd);
        m_cmdUndone.clear();  // Nouvelle action annule redo
    }
    
    // Annule la dernière commande
    virtual void undo() {
        if (!m_cmdDone.empty()) {
            CmdPtr cmd = m_cmdDone.back();
            m_cmdDone.pop_back();
            cmd->cancel();
            m_cmdUndone.push_back(cmd);
        }
    }
    
    // Refait une commande annulée
    virtual void redo() {
        if (!m_cmdUndone.empty()) {
            CmdPtr cmd = m_cmdUndone.back();
            m_cmdUndone.pop_back();
            cmd->execute();
            m_cmdDone.push_back(cmd);
        }
    }
    
    size_t undoCount() const { return m_cmdDone.size(); }
    size_t redoCount() const { return m_cmdUndone.size(); }

protected:
    Invoker() = default;
    static std::unique_ptr<Invoker> m_instance;
    CmdContainer m_cmdDone;    // Commandes exécutées
    CmdContainer m_cmdUndone;  // Commandes annulées
};

std::unique_ptr<Invoker> Invoker::m_instance = nullptr;
```

### Classe InvokerRegistry (Singleton Registry)

Une autre classe Singleton qui gère un **registre global** d'Invokers. Elle permet d'enregistrer et retrouver des Invokers par nom.

**Note importante :** C'est une classe **Singleton différente** de Invoker — le Registry lui-même est unique, et il gère plusieurs Invokers.

```cpp
class InvokerRegistry {
public:
    virtual ~InvokerRegistry() = default;
    
    // Point d'accès au Registry Singleton
    static InvokerRegistry* getInstance() {
        if (!m_instance) {
            m_instance.reset(new InvokerRegistry());
        }
        return m_instance.get();
    }
    
    // Enregistre un Invoker avec un nom
    virtual void registerInstance(std::string name, Invoker* instance) {
        m_registry[name] = instance;
    }
    
    // Retrouve un Invoker par son nom
    virtual Invoker* lookupInstance(std::string name) {
        auto it = m_registry.find(name);
        return (it != m_registry.end()) ? it->second : nullptr;
    }
    
    // Liste tous les Invokers enregistrés
    virtual void listInstances(std::ostream& o) {
        o << "Invokers enregistrés:" << std::endl;
        for (const auto& pair : m_registry) {
            o << "  • " << pair.first 
              << " (undo=" << pair.second->undoCount()
              << ", redo=" << pair.second->redoCount() << ")" << std::endl;
        }
    }

private:
    InvokerRegistry() = default;
    static std::unique_ptr<InvokerRegistry> m_instance;
    std::map<std::string, Invoker*> m_registry;
};

std::unique_ptr<InvokerRegistry> InvokerRegistry::m_instance = nullptr;
```

### Exemple de Commande

```cpp
class TraceCommand : public CommandAbs {
public:
    TraceCommand(std::ostream& os, std::string msg)
        : m_ostream(os), m_logMsg(std::move(msg)) {}
    
    void execute() override {
        m_ostream << "▶ " << m_logMsg << std::endl;
    }
    
    void cancel() override {
        m_ostream << "◀ " << m_logMsg << std::endl;
    }

private:
    std::ostream& m_ostream;
    std::string m_logMsg;
};

} // namespace PolyIcone3D
```

### Utilisation

```cpp
using namespace PolyIcone3D;

int main() {
    std::cout << "╔════════════════════════════════════════════╗" << std::endl;
    std::cout << "║     SINGLETON - INVOKER & REGISTRY         ║" << std::endl;
    std::cout << "╚════════════════════════════════════════════╝" << std::endl;
    
    // Obtenir l'Invoker Singleton
    std::cout << "\n=== Invoker Singleton ===" << std::endl;
    Invoker* invoker = Invoker::getInstance();
    
    // Exécuter des commandes
    CmdPtr cmd1 = std::make_shared<TraceCommand>(std::cout, "Dessiner cercle");
    CmdPtr cmd2 = std::make_shared<TraceCommand>(std::cout, "Dessiner carré");
    
    invoker->execute(cmd1);
    invoker->execute(cmd2);
    
    // Undo
    std::cout << "\n--- Undo ---" << std::endl;
    invoker->undo();
    
    // Vérifier que c'est bien le même Singleton
    Invoker* invoker2 = Invoker::getInstance();
    std::cout << "\nMême instance? " 
              << (invoker == invoker2 ? "OUI ✓" : "NON ✗") << std::endl;
    
    // Utiliser le Registry
    std::cout << "\n=== InvokerRegistry Singleton ===" << std::endl;
    InvokerRegistry* registry = InvokerRegistry::getInstance();
    
    // Enregistrer l'Invoker
    registry->registerInstance("main-invoker", invoker);
    
    // Lister
    registry->listInstances(std::cout);
    
    // Retrouver
    Invoker* found = registry->lookupInstance("main-invoker");
    std::cout << "\nLookup 'main-invoker': " 
              << (found != nullptr ? "Trouvé ✓" : "Non trouvé ✗") << std::endl;
    
    return 0;
}
```

### Sortie Attendue

```
╔════════════════════════════════════════════╗
║     SINGLETON - INVOKER & REGISTRY         ║
╚════════════════════════════════════════════╝

=== Invoker Singleton ===
▶ Dessiner cercle
▶ Dessiner carré

--- Undo ---
◀ Dessiner carré

Même instance? OUI ✓

=== InvokerRegistry Singleton ===
Invokers enregistrés:
  • main-invoker (undo=1, redo=1)

Lookup 'main-invoker': Trouvé ✓
```

---

## 📊 Diagramme UML Complet

```
┌─────────────────────────────────────────────────────────────────────────┐
│                  SINGLETON: INVOKER & REGISTRY                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────────────────────────────┐                                  │
│   │          Invoker                 │                                  │
│   ├──────────────────────────────────┤                                  │
│   │ -m_instance: unique_ptr<Invoker> │                                  │
│   │ -m_cmdDone: CmdContainer         │                                  │
│   │ -m_cmdUndone: CmdContainer       │                                  │
│   ├──────────────────────────────────┤                                  │
│   │ #Invoker()                       │                                  │
│   │ +getInstance(): Invoker*         │                                  │
│   │ +execute(cmd: CmdPtr&): void     │                                  │
│   │ +undo(): void                    │                                  │
│   │ +redo(): void                    │                                  │
│   └──────────────────────────────────┘                                  │
│                   △                                                    │
│                    │ gère                                               │
│                    │                                                    │
│   ┌────────────────┴──────────────────────┐                             │
│   │    InvokerRegistry                    │                             │
│   ├───────────────────────────────────────┤                             │
│   │ -m_instance: unique_ptr<Registry>     │                             │
│   │ -m_registry: map<string, Invoker*>    │                             │
│   ├───────────────────────────────────────┤                             │
│   │ -InvokerRegistry()                    │                             │
│   │ +getInstance(): InvokerRegistry*      │                             │
│   │ +registerInstance(name, instance)     │                             │
│   │ +lookupInstance(name): Invoker*       │                             │
│   │ +listInstances(ostream&): void        │                             │
│   └───────────────────────────────────────┘                             │
│                                                                         │
│   Chaque classe est un Singleton DIFFÉRENT:                             │
│   • Invoker = 1 seule instance                                          │
│   • InvokerRegistry = 1 seule instance                                  │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 🧪 Singleton et Tests

Le problème principal avec le Singleton est qu'il crée une **dépendance globale** difficile à tester. Voici les solutions recommandées :

### Solution 1: Utiliser une Interface (Recommandé)

```cpp
// Interface découplée
class IInvoker {
public:
    virtual void execute(CmdPtr& cmd) = 0;
    virtual void undo() = 0;
    virtual ~IInvoker() = default;
};

// Code client dépend de l'interface, pas du Singleton
class Editor {
    IInvoker& invoker;  // Injecté, pas récupéré globalement
public:
    Editor(IInvoker& inv) : invoker(inv) {}
    void doAction(CmdPtr cmd) { invoker.execute(cmd); }
};

// Test: injecter un mock
class MockInvoker : public IInvoker {
    std::vector<CmdPtr> executed;
public:
    void execute(CmdPtr& cmd) override { executed.push_back(cmd); }
    void undo() override {}
    size_t executeCount() const { return executed.size(); }
};

// Test:
MockInvoker mock;
Editor editor(mock);
editor.doAction(cmd);
assert(mock.executeCount() == 1);  // Vérifié!
```

### Solution 2: Méthode Reset pour les Tests

```cpp
class Invoker {
public:
    // Pour les tests SEULEMENT
    static void resetForTesting() {
        m_instance.reset();
    }
    
    // ... reste du code ...
};

// Test:
Invoker::resetForTesting();
Invoker* invoker1 = Invoker::getInstance();
invoker1->execute(cmd1);
// ... test ...

Invoker::resetForTesting();
Invoker* invoker2 = Invoker::getInstance();
// Nouvel état propre pour le prochain test
```

---

## ✅ Avantages

| Avantage | Explication |
|----------|-------------|
| **Instance unique garantie** | Impossible d'avoir deux instances accidentellement |
| **Accès global simple** | `Singleton::getInstance()` depuis n'importe où |
| **Ressource partagée** | Parfait pour les ressources limitées (BD, logger, etc.) |
| **Lazy loading** | Création seulement si utilisé (dépend de l'implémentation) |

---

## ❌ Inconvénients

| Inconvénient | Explication |
|--------------|-------------|
| **État global** | Dépendances cachées, difficile à tracer |
| **Tests difficiles** | Ne peut pas isoler facilement (à moins d'injection) |
| **Violation SRP** | Gère création ET logique métier |
| **Thread-safety** | À gérer avec soin selon l'implémentation |

### Préférer l'Injection de Dépendance

```cpp
// ❌ Couplé au Singleton (mauvais)
void maFonction() {
    Logger::getInstance().log("...");
}

// ✅ Découplé, testable (bon)
void maFonction(ILogger& logger) {
    logger.log("...");
}
```

---

## 🎯 Cas d'Utilisation Légitimes

1. **Logger/Logging** — Un seul point de journalisation
2. **Configuration** — Paramètres globaux de l'application
3. **Registry** — Registre nommé d'objets (comme `InvokerRegistry`)
4. **Cache partagé** — Cache accessible globalement
5. **Invoker** — Gestion centralisée des commandes undo/redo
6. **Pool de ressources** — Connexions BD, threads, etc.

---

## 🔗 Patrons Connexes

| Patron | Lien |
|--------|------|
| [Commande](../Comportemental/Commande.md) | L'Invoker (Singleton) gère les commandes |
| [Fabrique Abstraite](./FabriqueAbstraite.md) | Souvent implémentée comme Singleton |
| [État](../Comportemental/Etat.md) | Les objets d'état peuvent être Singletons |
| [Façade](../Structurel/Facade.md) | Peut être implémentée comme Singleton |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Adaptateur](../Structurel/Adaptateur.md)
