# 🏠 Patron Façade (Facade)

> **Patron Structurel** - Fournit une interface unifiée à un ensemble d'interfaces d'un sous-système.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Fournir une **interface unifiée** à un ensemble d'interfaces d'un sous-système. La Façade définit une interface de plus **haut niveau** qui rend le sous-système plus facile à utiliser.

---

## 🎯 Problème Résolu

- Comment simplifier l'utilisation d'un système complexe?
- Comment découpler les clients des détails d'implémentation?
- Comment fournir une API simple pour des cas d'usage courants?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON FAÇADE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌────────────────┐                                             │
│  │     Client     │                                             │
│  └───────┬────────┘                                             │
│          │                                                      │
│          │ utilise                                              │
│          ▼                                                      │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                        FAÇADE                             │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │                                                     │  │  │
│  │  │  +operationSimple() {                               │  │  │
│  │  │      sousSystemeA.operation1();                     │  │  │
│  │  │      sousSystemeB.operation2();                     │  │  │
│  │  │      sousSystemeC.operation3();                     │  │  │
│  │  │  }                                                  │  │  │
│  │  │                                                     │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                          │                                      │
│          ┌───────────────┼───────────────┐                      │
│          │               │               │                      │
│          ▼               ▼               ▼                      │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐             │
│  │SousSystemeA  │ │SousSystemeB  │ │SousSystemeC  │             │
│  ├──────────────┤ ├──────────────┤ ├──────────────┤             │
│  │+operation1() │ │+operation2() │ │+operation3() │             │
│  │+operation4() │ │+operation5() │ │+operation6() │             │
│  └──────────────┘ └──────────────┘ └──────────────┘             │
│                                                                 │
│  Le client n'a pas besoin de connaître les sous-systèmes        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🎬 Analogie: Home Cinema

```
┌─────────────────────────────────────────────────────────────────┐
│                   ANALOGIE: HOME CINEMA                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   SANS FAÇADE (complexe):                                       │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ 1. Allumer l'amplificateur                              │   │
│   │ 2. Configurer l'amplificateur sur DVD                   │   │
│   │ 3. Régler le volume à 5                                 │   │
│   │ 4. Allumer le lecteur DVD                               │   │
│   │ 5. Allumer le projecteur                                │   │
│   │ 6. Configurer projecteur en 16:9                        │   │
│   │ 7. Baisser les lumières                                 │   │
│   │ 8. Baisser l'écran                                      │   │
│   │ 9. Insérer le DVD                                       │   │
│   │ 10. Appuyer sur Play                                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   AVEC FAÇADE (simple):                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ homeCinema.regarderFilm("Matrix")                       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   La façade encapsule toute la complexité!                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Sous-systèmes Complexes

```cpp
#include <iostream>
#include <string>
#include <memory>

// ═══════════════════════════════════════
// SOUS-SYSTÈMES (classes complexes)
// ═══════════════════════════════════════

class Amplificateur {
public:
    void allumer() {
        std::cout << "🔊 Amplificateur allumé" << std::endl;
    }
    void eteindre() {
        std::cout << "🔇 Amplificateur éteint" << std::endl;
    }
    void setVolume(int niveau) {
        std::cout << "🔊 Volume réglé à " << niveau << std::endl;
    }
    void setSource(const std::string& source) {
        std::cout << "🔊 Source: " << source << std::endl;
    }
    void setSurroundSound() {
        std::cout << "🔊 Mode surround activé" << std::endl;
    }
};

class LecteurDVD {
public:
    void allumer() {
        std::cout << "📀 Lecteur DVD allumé" << std::endl;
    }
    void eteindre() {
        std::cout << "📀 Lecteur DVD éteint" << std::endl;
    }
    void charger(const std::string& film) {
        std::cout << "📀 Chargement: " << film << std::endl;
    }
    void ejecter() {
        std::cout << "📀 DVD éjecté" << std::endl;
    }
    void play() {
        std::cout << "▶️  Lecture en cours..." << std::endl;
    }
    void pause() {
        std::cout << "⏸️  Pause" << std::endl;
    }
    void stop() {
        std::cout << "⏹️  Arrêt" << std::endl;
    }
};

class Projecteur {
public:
    void allumer() {
        std::cout << "📽️  Projecteur allumé" << std::endl;
    }
    void eteindre() {
        std::cout << "📽️  Projecteur éteint" << std::endl;
    }
    void setInput(const std::string& input) {
        std::cout << "📽️  Input: " << input << std::endl;
    }
    void setMode169() {
        std::cout << "📽️  Mode 16:9 activé" << std::endl;
    }
    void setMode43() {
        std::cout << "📽️  Mode 4:3 activé" << std::endl;
    }
};

class Ecran {
public:
    void descendre() {
        std::cout << "🖼️  Écran descendu" << std::endl;
    }
    void monter() {
        std::cout << "🖼️  Écran remonté" << std::endl;
    }
};

class Lumieres {
public:
    void allumer() {
        std::cout << "💡 Lumières allumées" << std::endl;
    }
    void eteindre() {
        std::cout << "💡 Lumières éteintes" << std::endl;
    }
    void tamiser(int niveau) {
        std::cout << "💡 Lumières tamisées à " << niveau << "%" << std::endl;
    }
};

class MachinePopcorn {
public:
    void allumer() {
        std::cout << "🍿 Machine à popcorn allumée" << std::endl;
    }
    void eteindre() {
        std::cout << "🍿 Machine à popcorn éteinte" << std::endl;
    }
    void faire() {
        std::cout << "🍿 Préparation du popcorn..." << std::endl;
    }
};
```

### Façade

```cpp
// ═══════════════════════════════════════
// FAÇADE
// ═══════════════════════════════════════

class HomeCinemaFacade {
private:
    std::unique_ptr<Amplificateur> ampli;
    std::unique_ptr<LecteurDVD> dvd;
    std::unique_ptr<Projecteur> projecteur;
    std::unique_ptr<Ecran> ecran;
    std::unique_ptr<Lumieres> lumieres;
    std::unique_ptr<MachinePopcorn> popcorn;
    
public:
    HomeCinemaFacade() {
        ampli = std::make_unique<Amplificateur>();
        dvd = std::make_unique<LecteurDVD>();
        projecteur = std::make_unique<Projecteur>();
        ecran = std::make_unique<Ecran>();
        lumieres = std::make_unique<Lumieres>();
        popcorn = std::make_unique<MachinePopcorn>();
    }
    
    // ═══ MÉTHODE SIMPLIFIÉE: Regarder un film ═══
    void regarderFilm(const std::string& film) {
        std::cout << "\n🎬 Préparation pour regarder: " << film << std::endl;
        std::cout << "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━" << std::endl;
        
        popcorn->allumer();
        popcorn->faire();
        
        lumieres->tamiser(10);
        
        ecran->descendre();
        
        projecteur->allumer();
        projecteur->setInput("DVD");
        projecteur->setMode169();
        
        ampli->allumer();
        ampli->setSource("DVD");
        ampli->setSurroundSound();
        ampli->setVolume(5);
        
        dvd->allumer();
        dvd->charger(film);
        dvd->play();
        
        std::cout << "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━" << std::endl;
        std::cout << "🎬 Bon film!\n" << std::endl;
    }
    
    // ═══ MÉTHODE SIMPLIFIÉE: Fin du film ═══
    void finFilm() {
        std::cout << "\n🎬 Fin de la séance" << std::endl;
        std::cout << "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━" << std::endl;
        
        dvd->stop();
        dvd->ejecter();
        dvd->eteindre();
        
        popcorn->eteindre();
        
        ampli->eteindre();
        
        projecteur->eteindre();
        
        ecran->monter();
        
        lumieres->allumer();
        
        std::cout << "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━" << std::endl;
        std::cout << "🎬 À bientôt!\n" << std::endl;
    }
    
    // ═══ MÉTHODE SIMPLIFIÉE: Pause ═══
    void pause() {
        std::cout << "\n⏸️  Pause" << std::endl;
        dvd->pause();
        lumieres->tamiser(50);
    }
    
    // ═══ MÉTHODE SIMPLIFIÉE: Reprendre ═══
    void reprendre() {
        std::cout << "\n▶️  Reprise" << std::endl;
        lumieres->tamiser(10);
        dvd->play();
    }
    
    // Accès direct aux sous-systèmes si nécessaire
    Amplificateur* getAmplificateur() { return ampli.get(); }
    LecteurDVD* getLecteurDVD() { return dvd.get(); }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON FAÇADE - DEMO             ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Créer la façade
    HomeCinemaFacade homeCinema;
    
    // Le client utilise une interface simple
    homeCinema.regarderFilm("The Matrix");
    
    // Pause pour aller chercher une boisson
    homeCinema.pause();
    
    // Reprendre
    homeCinema.reprendre();
    
    // Fin du film
    homeCinema.finFilm();
    
    // Accès direct si besoin avancé
    std::cout << "\n=== Accès direct au sous-système ===" << std::endl;
    homeCinema.getAmplificateur()->setVolume(8);
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON FAÇADE - DEMO             ║
╚═══════════════════════════════════════╝

🎬 Préparation pour regarder: The Matrix
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🍿 Machine à popcorn allumée
🍿 Préparation du popcorn...
💡 Lumières tamisées à 10%
🖼️  Écran descendu
📽️  Projecteur allumé
📽️  Input: DVD
📽️  Mode 16:9 activé
🔊 Amplificateur allumé
🔊 Source: DVD
🔊 Mode surround activé
🔊 Volume réglé à 5
📀 Lecteur DVD allumé
📀 Chargement: The Matrix
▶️  Lecture en cours...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎬 Bon film!


⏸️  Pause
⏸️  Pause
💡 Lumières tamisées à 50%

▶️  Reprise
💡 Lumières tamisées à 10%
▶️  Lecture en cours...

🎬 Fin de la séance
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏹️  Arrêt
📀 DVD éjecté
📀 Lecteur DVD éteint
🍿 Machine à popcorn éteinte
🔇 Amplificateur éteint
📽️  Projecteur éteint
🖼️  Écran remonté
💡 Lumières allumées
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🎬 À bientôt!


=== Accès direct au sous-système ===
🔊 Volume réglé à 8
```

---

## 🖥️ Autre Exemple: Compilation

```cpp
// Façade pour un système de compilation
class CompilationFacade {
    Preprocesseur preprocesseur;
    Compilateur compilateur;
    Assembleur assembleur;
    Linker linker;
    
public:
    // Interface simple
    bool compiler(const std::string& fichierSource) {
        std::cout << "Compilation de " << fichierSource << std::endl;
        
        auto pretraite = preprocesseur.traiter(fichierSource);
        if (!pretraite) return false;
        
        auto objetCode = compilateur.compiler(*pretraite);
        if (!objetCode) return false;
        
        auto assemblee = assembleur.assembler(*objetCode);
        if (!assemblee) return false;
        
        return linker.lier(*assemblee, "output.exe");
    }
};

// Utilisation
CompilationFacade facade;
facade.compiler("main.cpp");  // Une seule ligne!
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Simplicité** | Interface simple pour système complexe |
| **Découplage** | Client indépendant des sous-systèmes |
| **Layering** | Organisation en couches |
| **Flexibilité** | Accès direct toujours possible |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **God Object** | Peut devenir trop gros |
| **Abstraction forcée** | Pas toujours nécessaire |
| **Maintenance** | Doit évoluer avec les sous-systèmes |

---

## 🎯 Cas d'Utilisation

1. **API publiques** - Simplifier une bibliothèque
2. **Legacy** - Envelopper du vieux code
3. **Couches** - Interface entre couches applicatives
4. **Frameworks** - Points d'entrée simplifiés

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Adaptateur](./Adaptateur.md) | Convertit interface vs simplifie |
| [Médiateur](../Comportemental/Mediateur.md) | Abstrait communications |
| [Singleton](../Creationnel/Singleton.md) | La façade est souvent unique |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Proxy](./Proxy.md)
