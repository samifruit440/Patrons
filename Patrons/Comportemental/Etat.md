# 🔀 Patron État (State)

> **Patron Comportemental** - Permet à un objet de modifier son comportement lorsque son état interne change.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Permettre à un objet de **modifier son comportement** lorsque son **état interne change**. L'objet semblera avoir changé de classe.

---

## 🎯 Problème Résolu

- Comment éviter les gros blocs `switch/case` ou `if/else` pour gérer les états?
- Comment rendre les transitions d'état explicites et organisées?
- Comment ajouter de nouveaux états sans modifier le code existant?

---

## 📊 Diagramme UML

```
┌──────────────────────────────────────────────────────────────────┐
│                        PATRON ÉTAT                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────────────────┐         ┌──────────────────────┐       │
│   │      Contexte       │         │     «interface»      │       │
│   ├─────────────────────┤         │        IEtat         │       │
│   │ -etat: IEtat        │────────►├──────────────────────┤       │
│   ├─────────────────────┤         │ +gerer(contexte)     │       │
│   │ +setEtat(e: IEtat)  │         └──────────┬───────────┘       │
│   │ +requete()          │                    │                   │
│   └─────────────────────┘                    │                   │
│                                              │                   │
│   requete() {                   ┌────────────┼─────────────┐     │
│       etat.gerer(this);         │            │             │     │
│   }                             │            │             │     │
│                        ┌────────▼───┐  ┌─────▼──────┐ ┌────▼────┐│
│                        │  EtatA     │  │   EtatB    │ │  EtatC  ││
│                        ├────────────┤  ├────────────┤ ├─────────┤│
│                        │ +gerer() { │  │ +gerer()   │ │+gerer() ││
│                        │  // action │  │  // action │ │         ││
│                        │  // puis   │  │  // puis   │ │         ││
│                        │  // changer│  │  // changer│ │         ││
│                        │  // état   │  │  // état   │ │         ││
│                        │ }          │  │ }          │ │         ││
│                        └────────────┘  └────────────┘ └─────────┘│
│                                                                  │
│   Chaque état encapsule le comportement et les transitions       │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Machine à États

```
┌─────────────────────────────────────────────────────────────────┐
│                    EXEMPLE: DISTRIBUTEUR                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│              ┌──────────────────┐                               │
│              │  SansPièce       │◄──────────────────────┐       │
│              │  (état initial)  │                       │       │
│              └────────┬─────────┘                       │       │
│                       │                                 │       │
│            insérerPièce()                     éjecter() │       │
│                       │                                 │       │
│                       ▼                                 │       │
│              ┌──────────────────┐                       │       │
│              │   AvecPièce      │───────────────────────┘       │
│              └────────┬─────────┘                               │
│                       │                                         │
│            tournerManivelle()                                   │
│                       │                                         │
│                       ▼                                         │
│              ┌──────────────────┐                               │
│              │    Vendu         │                               │
│              └────────┬─────────┘                               │
│                       │                                         │
│              distribuer()                                       │
│                       │                                         │
│           ┌───────────┴───────────┐                             │
│           │                       │                             │
│           ▼                       ▼                             │
│    ┌─────────────┐         ┌─────────────┐                      │
│    │  SansPièce  │         │   Épuisé    │                      │
│    │ (si stock>0)│         │ (si stock=0)│                      │
│    └─────────────┘         └─────────────┘                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface État

```cpp
#include <iostream>
#include <memory>
#include <string>

// Forward declaration
class Distributeur;

// Interface État
class IEtat {
public:
    virtual void insererPiece(Distributeur* d) = 0;
    virtual void ejecterPiece(Distributeur* d) = 0;
    virtual void tournerManivelle(Distributeur* d) = 0;
    virtual void distribuer(Distributeur* d) = 0;
    virtual std::string getNom() const = 0;
    virtual ~IEtat() = default;
};
```

### Contexte (Distributeur)

```cpp
// Contexte: Distributeur automatique
class Distributeur {
private:
    std::shared_ptr<IEtat> etat;
    int nombreBonbons;
    
public:
    // Déclaration des états (définis plus bas)
    static std::shared_ptr<IEtat> etatSansPiece;
    static std::shared_ptr<IEtat> etatAvecPiece;
    static std::shared_ptr<IEtat> etatVendu;
    static std::shared_ptr<IEtat> etatEpuise;
    
    Distributeur(int stock) : nombreBonbons(stock) {
        // État initial selon le stock
        if (stock > 0) {
            etat = etatSansPiece;
        } else {
            etat = etatEpuise;
        }
    }
    
    void setEtat(std::shared_ptr<IEtat> nouvelEtat) {
        std::cout << "   → Transition: " << etat->getNom() 
                  << " → " << nouvelEtat->getNom() << std::endl;
        etat = nouvelEtat;
    }
    
    std::shared_ptr<IEtat> getEtat() const { return etat; }
    
    // Actions déléguées à l'état courant
    void insererPiece() {
        std::cout << "\n💰 Action: Insérer pièce" << std::endl;
        etat->insererPiece(this);
    }
    
    void ejecterPiece() {
        std::cout << "\n↩️  Action: Éjecter pièce" << std::endl;
        etat->ejecterPiece(this);
    }
    
    void tournerManivelle() {
        std::cout << "\n🔄 Action: Tourner manivelle" << std::endl;
        etat->tournerManivelle(this);
        etat->distribuer(this);  // Distribution automatique
    }
    
    // Gestion du stock
    void libererBonbon() {
        if (nombreBonbons > 0) {
            nombreBonbons--;
            std::cout << "   🍬 Bonbon distribué! Stock restant: " 
                      << nombreBonbons << std::endl;
        }
    }
    
    int getStock() const { return nombreBonbons; }
    
    void afficherEtat() const {
        std::cout << "📊 État: " << etat->getNom() 
                  << " | Stock: " << nombreBonbons << std::endl;
    }
};
```

### États Concrets

```cpp
// État: Sans Pièce
class EtatSansPiece : public IEtat {
public:
    void insererPiece(Distributeur* d) override {
        std::cout << "   ✅ Pièce acceptée" << std::endl;
        d->setEtat(Distributeur::etatAvecPiece);
    }
    
    void ejecterPiece(Distributeur* d) override {
        std::cout << "   ❌ Pas de pièce à éjecter" << std::endl;
    }
    
    void tournerManivelle(Distributeur* d) override {
        std::cout << "   ❌ Insérez d'abord une pièce" << std::endl;
    }
    
    void distribuer(Distributeur* d) override {
        std::cout << "   ❌ Paiement requis" << std::endl;
    }
    
    std::string getNom() const override { return "SansPièce"; }
};

// État: Avec Pièce
class EtatAvecPiece : public IEtat {
public:
    void insererPiece(Distributeur* d) override {
        std::cout << "   ❌ Pièce déjà insérée" << std::endl;
    }
    
    void ejecterPiece(Distributeur* d) override {
        std::cout << "   ✅ Pièce retournée" << std::endl;
        d->setEtat(Distributeur::etatSansPiece);
    }
    
    void tournerManivelle(Distributeur* d) override {
        std::cout << "   ✅ Manivelle tournée..." << std::endl;
        d->setEtat(Distributeur::etatVendu);
    }
    
    void distribuer(Distributeur* d) override {
        std::cout << "   ❌ Tournez la manivelle" << std::endl;
    }
    
    std::string getNom() const override { return "AvecPièce"; }
};

// État: Vendu (en cours de distribution)
class EtatVendu : public IEtat {
public:
    void insererPiece(Distributeur* d) override {
        std::cout << "   ❌ Attendez votre bonbon" << std::endl;
    }
    
    void ejecterPiece(Distributeur* d) override {
        std::cout << "   ❌ Trop tard, bonbon en cours" << std::endl;
    }
    
    void tournerManivelle(Distributeur* d) override {
        std::cout << "   ❌ Déjà en cours" << std::endl;
    }
    
    void distribuer(Distributeur* d) override {
        d->libererBonbon();
        if (d->getStock() > 0) {
            d->setEtat(Distributeur::etatSansPiece);
        } else {
            std::cout << "   ⚠️  Plus de bonbons!" << std::endl;
            d->setEtat(Distributeur::etatEpuise);
        }
    }
    
    std::string getNom() const override { return "Vendu"; }
};

// État: Épuisé
class EtatEpuise : public IEtat {
public:
    void insererPiece(Distributeur* d) override {
        std::cout << "   ❌ Machine vide, pièce refusée" << std::endl;
    }
    
    void ejecterPiece(Distributeur* d) override {
        std::cout << "   ❌ Pas de pièce insérée" << std::endl;
    }
    
    void tournerManivelle(Distributeur* d) override {
        std::cout << "   ❌ Machine vide" << std::endl;
    }
    
    void distribuer(Distributeur* d) override {
        std::cout << "   ❌ Rien à distribuer" << std::endl;
    }
    
    std::string getNom() const override { return "Épuisé"; }
};
```

### Initialisation et Utilisation

```cpp
// Initialisation des états (Singleton)
std::shared_ptr<IEtat> Distributeur::etatSansPiece = 
    std::make_shared<EtatSansPiece>();
std::shared_ptr<IEtat> Distributeur::etatAvecPiece = 
    std::make_shared<EtatAvecPiece>();
std::shared_ptr<IEtat> Distributeur::etatVendu = 
    std::make_shared<EtatVendu>();
std::shared_ptr<IEtat> Distributeur::etatEpuise = 
    std::make_shared<EtatEpuise>();

int main() {
    std::cout << "╔════════════════════════════════════════╗" << std::endl;
    std::cout << "║        PATRON ÉTAT - DEMO              ║" << std::endl;
    std::cout << "╚════════════════════════════════════════╝" << std::endl;
    
    Distributeur distributeur(2);  // 2 bonbons en stock
    distributeur.afficherEtat();
    
    // Scénario 1: Achat normal
    std::cout << "\n=== Achat 1 ===" << std::endl;
    distributeur.insererPiece();
    distributeur.tournerManivelle();
    distributeur.afficherEtat();
    
    // Scénario 2: Éjecter la pièce
    std::cout << "\n=== Tentative avec éjection ===" << std::endl;
    distributeur.insererPiece();
    distributeur.ejecterPiece();
    distributeur.afficherEtat();
    
    // Scénario 3: Dernier bonbon
    std::cout << "\n=== Achat 2 (dernier) ===" << std::endl;
    distributeur.insererPiece();
    distributeur.tournerManivelle();
    distributeur.afficherEtat();
    
    // Scénario 4: Machine vide
    std::cout << "\n=== Tentative sur machine vide ===" << std::endl;
    distributeur.insererPiece();
    
    return 0;
}
```

### Sortie

```
╔════════════════════════════════════════╗
║        PATRON ÉTAT - DEMO              ║
╚════════════════════════════════════════╝
📊 État: SansPièce | Stock: 2

=== Achat 1 ===

💰 Action: Insérer pièce
   ✅ Pièce acceptée
   → Transition: SansPièce → AvecPièce

🔄 Action: Tourner manivelle
   ✅ Manivelle tournée...
   → Transition: AvecPièce → Vendu
   🍬 Bonbon distribué! Stock restant: 1
   → Transition: Vendu → SansPièce
📊 État: SansPièce | Stock: 1

=== Tentative avec éjection ===

💰 Action: Insérer pièce
   ✅ Pièce acceptée
   → Transition: SansPièce → AvecPièce

↩️  Action: Éjecter pièce
   ✅ Pièce retournée
   → Transition: AvecPièce → SansPièce
📊 État: SansPièce | Stock: 1

=== Achat 2 (dernier) ===

💰 Action: Insérer pièce
   ✅ Pièce acceptée
   → Transition: SansPièce → AvecPièce

🔄 Action: Tourner manivelle
   ✅ Manivelle tournée...
   → Transition: AvecPièce → Vendu
   🍬 Bonbon distribué! Stock restant: 0
   ⚠️  Plus de bonbons!
   → Transition: Vendu → Épuisé
📊 État: Épuisé | Stock: 0

=== Tentative sur machine vide ===

💰 Action: Insérer pièce
   ❌ Machine vide, pièce refusée
```

---

## 🆚 État vs Switch/Case

```
┌─────────────────────────────────────────────────────────────────┐
│                    COMPARAISON                                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ❌ SANS PATRON ÉTAT:                                          │
│   ┌─────────────────────────────────────┐                       │
│   │ void insererPiece() {               │                       │
│   │   switch(etat) {                    │                       │
│   │     case SANS_PIECE:                │                       │
│   │       etat = AVEC_PIECE;            │                       │
│   │       break;                        │                       │
│   │     case AVEC_PIECE:                │                       │
│   │       cout << "déjà une pièce";     │                       │
│   │       break;                        │                       │
│   │     case VENDU:                     │                       │
│   │       cout << "attendez";           │                       │
│   │       break;                        │                       │
│   │     case EPUISE:                    │                       │
│   │       cout << "refusée";            │                       │
│   │       break;                        │                       │
│   │   }                                 │                       │
│   │ }                                   │ ← Répété pour chaque  │
│   └─────────────────────────────────────┘   action!             │
│                                                                 │
│   ✅ AVEC PATRON ÉTAT:                                          │
│   ┌─────────────────────────────────────┐                       │
│   │ void insererPiece() {               │                       │
│   │   etat->insererPiece(this);         │ ← Simple délégation   │
│   │ }                                   │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Lisibilité** | Chaque état dans sa propre classe |
| **Extensibilité** | Ajouter un état = ajouter une classe |
| **Principe OCP** | Ouvert à l'extension, fermé à la modification |
| **Transitions explicites** | Les changements d'état sont clairs |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Nombre de classes** | Une classe par état |
| **Couplage** | États connaissent le contexte |
| **Complexité** | Peut être excessif pour peu d'états |

---

## 🎯 Cas d'Utilisation

1. **Distributeurs automatiques** - États de paiement/distribution
2. **Lecteurs média** - Play, Pause, Stop
3. **Connexions réseau** - Connecté, Déconnecté, En attente
4. **Workflows** - Brouillon, En revue, Approuvé, Publié
5. **Jeux** - États du personnage (Idle, Running, Jumping)

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Stratégie](./Strategie.md) | Similaire mais pour algorithmes, pas états |
| [Singleton](../Creationnel/Singleton.md) | États souvent partagés comme singletons |
| [Patron de Méthode](./PatronMethode.md) | Peut définir le squelette des transitions |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Visiteur](./Visiteur.md)
