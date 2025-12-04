# 🎛️ Patron Médiateur (Mediator)

> **Patron Comportemental** - Définit un objet qui encapsule les interactions entre un ensemble d'objets.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Définir un objet qui **encapsule les interactions** entre un ensemble d'objets. Le Médiateur favorise un **couplage faible** en évitant que les objets ne se réfèrent explicitement les uns aux autres.

---

## 🎯 Problème Résolu

- Comment réduire le couplage entre objets qui communiquent?
- Comment centraliser la logique de communication complexe?
- Comment modifier les interactions sans toucher aux composants?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON MÉDIATEUR                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│            ┌──────────────────────┐                             │
│            │     «interface»      │                             │
│            │      IMediateur      │                             │
│            ├──────────────────────┤                             │
│            │ +notifier(emetteur,  │                             │
│            │           evenement) │                             │
│            └──────────┬───────────┘                             │
│                       │                                         │
│                       │                                         │
│            ┌──────────▼───────────┐                             │
│            │  MediateurConcret    │                             │
│            ├──────────────────────┤                             │
│            │ -composantA          │                             │
│            │ -composantB          │                             │
│            │ -composantC          │                             │
│            ├──────────────────────┤                             │
│            │ +notifier() {        │                             │
│            │   // coordonne       │                             │
│            │   // interactions    │                             │
│            │ }                    │                             │
│            └──────────────────────┘                             │
│                  ▲    ▲    ▲                                    │
│                  │    │    │                                    │
│          ┌───────┘    │    └────────┐                           │
│          │            │             │                           │
│    ┌─────┴─────┐ ┌────┴─────┐ ┌─────┴─────┐                     │
│    │ComposantA │ │ComposantB│ │ComposantC │                     │
│    ├───────────┤ ├──────────┤ ├───────────┤                     │
│    │-mediateur │ │-mediateur│ │-mediateur │                     │
│    ├───────────┤ ├──────────┤ ├───────────┤                     │
│    │ action() {│ │ action() │ │ action()  │                     │
│    │  mediateur│ │          │ │           │                     │
│    │  .notifier│ │          │ │           │                     │
│    │  (this)   │ │          │ │           │                     │
│    │ }         │ │          │ │           │                     │
│    └───────────┘ └──────────┘ └───────────┘                     │
│                                                                 │
│   Composants ne se connaissent PAS entre eux                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Communication Centralisée

```
┌─────────────────────────────────────────────────────────────────┐
│              SANS vs AVEC MÉDIATEUR                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ❌ SANS MÉDIATEUR:                                            │
│                                                                 │
│       ┌───┐     ┌───┐                                           │
│       │ A │◄───►│ B │     Chaque composant connaît              │
│       └───┘     └───┘     tous les autres                       │
│         ▲ ╲   ╱ ▲         = N*(N-1) connexions                  │
│         │  ╲ ╱  │                                               │
│         │   ╳   │                                               │
│         │  ╱ ╲  │                                               │
│         ▼ ╱   ╲ ▼                                               │
│       ┌───┐     ┌───┐                                           │
│       │ C │◄───►│ D │                                           │
│       └───┘     └───┘                                           │
│                                                                 │
│   ✅ AVEC MÉDIATEUR:                                            │
│                                                                 │
│       ┌───┐     ┌───┐                                           │
│       │ A │     │ B │     Chaque composant ne                   │
│       └─┬─┘     └─┬─┘     connaît que le médiateur              │
│         │         │       = N connexions                        │
│         └────┬────┘                                             │
│              ▼                                                  │
│         ┌─────────┐                                             │
│         │MEDIATEUR│                                             │
│         └─────────┘                                             │
│              ▲                                                  │
│         ┌────┴────┐                                             │
│         │         │                                             │
│       ┌─┴─┐     ┌─┴─┐                                           │
│       │ C │     │ D │                                           │
│       └───┘     └───┘                                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Médiateur

```cpp
#include <iostream>
#include <string>
#include <memory>

// Forward declarations
class Composant;

// Interface Médiateur
class IMediateur {
public:
    virtual void notifier(Composant* emetteur, 
                          const std::string& evenement) = 0;
    virtual ~IMediateur() = default;
};
```

### Classe de Base Composant

```cpp
// Classe de base pour les composants
class Composant {
protected:
    IMediateur* mediateur;
    
public:
    Composant(IMediateur* m = nullptr) : mediateur(m) {}
    
    void setMediateur(IMediateur* m) {
        mediateur = m;
    }
    
    virtual ~Composant() = default;
};
```

### Composants Concrets (UI)

```cpp
// Composant: Champ de texte
class ChampTexte : public Composant {
private:
    std::string texte;
    bool estVide = true;
    
public:
    using Composant::Composant;
    
    void setTexte(const std::string& t) {
        texte = t;
        estVide = t.empty();
        std::cout << "📝 ChampTexte: \"" << texte << "\"" << std::endl;
        if (mediateur) {
            mediateur->notifier(this, "texte_change");
        }
    }
    
    std::string getTexte() const { return texte; }
    bool vide() const { return estVide; }
};

// Composant: Bouton
class Bouton : public Composant {
private:
    std::string label;
    bool actif = true;
    
public:
    Bouton(const std::string& l, IMediateur* m = nullptr) 
        : Composant(m), label(l) {}
    
    void cliquer() {
        if (actif) {
            std::cout << "🔘 Bouton \"" << label << "\" cliqué" << std::endl;
            if (mediateur) {
                mediateur->notifier(this, "clic");
            }
        } else {
            std::cout << "🔘 Bouton \"" << label << "\" désactivé" << std::endl;
        }
    }
    
    void setActif(bool a) {
        actif = a;
        std::cout << "   → Bouton \"" << label << "\" " 
                  << (actif ? "activé ✓" : "désactivé ✗") << std::endl;
    }
    
    bool estActif() const { return actif; }
    std::string getLabel() const { return label; }
};

// Composant: Case à cocher
class CaseACocher : public Composant {
private:
    std::string label;
    bool cochee = false;
    
public:
    CaseACocher(const std::string& l, IMediateur* m = nullptr)
        : Composant(m), label(l) {}
    
    void basculer() {
        cochee = !cochee;
        std::cout << "☑️  Case \"" << label << "\": " 
                  << (cochee ? "[✓]" : "[ ]") << std::endl;
        if (mediateur) {
            mediateur->notifier(this, "case_change");
        }
    }
    
    bool estCochee() const { return cochee; }
};

// Composant: Liste déroulante
class ListeDeroulante : public Composant {
private:
    std::vector<std::string> options;
    int selectionIndex = -1;
    
public:
    using Composant::Composant;
    
    void ajouterOption(const std::string& opt) {
        options.push_back(opt);
    }
    
    void selectionner(int index) {
        if (index >= 0 && index < options.size()) {
            selectionIndex = index;
            std::cout << "📋 Liste: sélection \"" 
                      << options[index] << "\"" << std::endl;
            if (mediateur) {
                mediateur->notifier(this, "selection_change");
            }
        }
    }
    
    std::string getSelection() const {
        if (selectionIndex >= 0) return options[selectionIndex];
        return "";
    }
    
    bool aSelection() const { return selectionIndex >= 0; }
};
```

### Médiateur Concret

```cpp
// Médiateur Concret: Dialogue d'inscription
class MediateurInscription : public IMediateur {
private:
    std::shared_ptr<ChampTexte> champNom;
    std::shared_ptr<ChampTexte> champEmail;
    std::shared_ptr<CaseACocher> caseConditions;
    std::shared_ptr<ListeDeroulante> listePays;
    std::shared_ptr<Bouton> boutonValider;
    std::shared_ptr<Bouton> boutonAnnuler;
    
public:
    void setComposants(
        std::shared_ptr<ChampTexte> nom,
        std::shared_ptr<ChampTexte> email,
        std::shared_ptr<CaseACocher> conditions,
        std::shared_ptr<ListeDeroulante> pays,
        std::shared_ptr<Bouton> valider,
        std::shared_ptr<Bouton> annuler
    ) {
        champNom = nom;
        champEmail = email;
        caseConditions = conditions;
        listePays = pays;
        boutonValider = valider;
        boutonAnnuler = annuler;
        
        // Connecter les composants au médiateur
        champNom->setMediateur(this);
        champEmail->setMediateur(this);
        caseConditions->setMediateur(this);
        listePays->setMediateur(this);
        boutonValider->setMediateur(this);
        boutonAnnuler->setMediateur(this);
    }
    
    void notifier(Composant* emetteur, 
                  const std::string& evenement) override {
        
        std::cout << "   [Médiateur reçoit: " << evenement << "]" << std::endl;
        
        // Logique de coordination centralisée
        if (evenement == "texte_change" || 
            evenement == "case_change" || 
            evenement == "selection_change") {
            
            // Vérifier si le formulaire est valide
            bool formulaireValide = 
                !champNom->vide() && 
                !champEmail->vide() && 
                caseConditions->estCochee() &&
                listePays->aSelection();
            
            boutonValider->setActif(formulaireValide);
        }
        else if (evenement == "clic") {
            Bouton* bouton = dynamic_cast<Bouton*>(emetteur);
            if (bouton) {
                if (bouton->getLabel() == "Valider") {
                    std::cout << "\n✅ INSCRIPTION VALIDÉE!" << std::endl;
                    std::cout << "   Nom: " << champNom->getTexte() << std::endl;
                    std::cout << "   Email: " << champEmail->getTexte() << std::endl;
                    std::cout << "   Pays: " << listePays->getSelection() << std::endl;
                }
                else if (bouton->getLabel() == "Annuler") {
                    std::cout << "\n❌ Inscription annulée" << std::endl;
                }
            }
        }
    }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔════════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON MÉDIATEUR - DEMO           ║" << std::endl;
    std::cout << "╚════════════════════════════════════════╝" << std::endl;
    
    // Créer le médiateur
    auto mediateur = std::make_shared<MediateurInscription>();
    
    // Créer les composants
    auto champNom = std::make_shared<ChampTexte>();
    auto champEmail = std::make_shared<ChampTexte>();
    auto caseConditions = std::make_shared<CaseACocher>("J'accepte les conditions");
    auto listePays = std::make_shared<ListeDeroulante>();
    auto boutonValider = std::make_shared<Bouton>("Valider");
    auto boutonAnnuler = std::make_shared<Bouton>("Annuler");
    
    // Configurer la liste
    listePays->ajouterOption("Canada");
    listePays->ajouterOption("France");
    listePays->ajouterOption("Belgique");
    
    // Connecter au médiateur
    mediateur->setComposants(
        champNom, champEmail, caseConditions, 
        listePays, boutonValider, boutonAnnuler
    );
    
    // Initialement, le bouton Valider est désactivé
    boutonValider->setActif(false);
    
    std::cout << "\n=== Simulation d'interaction utilisateur ===" << std::endl;
    
    // L'utilisateur remplit le formulaire
    std::cout << "\n--- Étape 1: Entrer le nom ---" << std::endl;
    champNom->setTexte("Jean Dupont");
    
    std::cout << "\n--- Étape 2: Entrer l'email ---" << std::endl;
    champEmail->setTexte("jean@email.com");
    
    std::cout << "\n--- Étape 3: Sélectionner le pays ---" << std::endl;
    listePays->selectionner(1);  // France
    
    std::cout << "\n--- Étape 4: Tenter de valider (sans conditions) ---" << std::endl;
    boutonValider->cliquer();  // Désactivé!
    
    std::cout << "\n--- Étape 5: Accepter les conditions ---" << std::endl;
    caseConditions->basculer();
    
    std::cout << "\n--- Étape 6: Valider l'inscription ---" << std::endl;
    boutonValider->cliquer();  // Maintenant actif!
    
    return 0;
}
```

### Sortie

```
╔════════════════════════════════════════╗
║      PATRON MÉDIATEUR - DEMO           ║
╚════════════════════════════════════════╝
   → Bouton "Valider" désactivé ✗

=== Simulation d'interaction utilisateur ===

--- Étape 1: Entrer le nom ---
📝 ChampTexte: "Jean Dupont"
   [Médiateur reçoit: texte_change]
   → Bouton "Valider" désactivé ✗

--- Étape 2: Entrer l'email ---
📝 ChampTexte: "jean@email.com"
   [Médiateur reçoit: texte_change]
   → Bouton "Valider" désactivé ✗

--- Étape 3: Sélectionner le pays ---
📋 Liste: sélection "France"
   [Médiateur reçoit: selection_change]
   → Bouton "Valider" désactivé ✗

--- Étape 4: Tenter de valider (sans conditions) ---
🔘 Bouton "Valider" désactivé

--- Étape 5: Accepter les conditions ---
☑️  Case "J'accepte les conditions": [✓]
   [Médiateur reçoit: case_change]
   → Bouton "Valider" activé ✓

--- Étape 6: Valider l'inscription ---
🔘 Bouton "Valider" cliqué
   [Médiateur reçoit: clic]

✅ INSCRIPTION VALIDÉE!
   Nom: Jean Dupont
   Email: jean@email.com
   Pays: France
```

---

## 🎮 Autre Exemple: Chat Room

```cpp
// Médiateur pour une salle de chat
class ISalonChat {
public:
    virtual void envoyerMessage(const std::string& message, 
                                 Utilisateur* expediteur) = 0;
    virtual void ajouterUtilisateur(Utilisateur* u) = 0;
    virtual ~ISalonChat() = default;
};

class SalonChat : public ISalonChat {
private:
    std::vector<Utilisateur*> utilisateurs;
    
public:
    void ajouterUtilisateur(Utilisateur* u) override {
        utilisateurs.push_back(u);
        u->setSalon(this);
    }
    
    void envoyerMessage(const std::string& message, 
                        Utilisateur* expediteur) override {
        // Distribuer le message à tous SAUF l'expéditeur
        for (auto u : utilisateurs) {
            if (u != expediteur) {
                u->recevoir(message, expediteur->getNom());
            }
        }
    }
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Couplage faible** | Composants ne se connaissent pas |
| **Centralisation** | Logique en un seul endroit |
| **Réutilisation** | Composants réutilisables |
| **Maintenance** | Facile de modifier les interactions |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **God Object** | Le médiateur peut devenir trop complexe |
| **Single Point** | Point unique de défaillance |
| **Performance** | Indirection supplémentaire |

---

## 🎯 Cas d'Utilisation

1. **Interfaces graphiques** - Coordination de widgets
2. **Systèmes de chat** - Distribution de messages
3. **Contrôle aérien** - Coordination d'avions
4. **Formulaires** - Validation interdépendante
5. **Systèmes distribués** - Orchestration de services

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Observateur](./Observateur.md) | Alternative pour notification |
| [Façade](../Structurel/Facade.md) | Simplifie aussi mais dans un sens |
| [Commande](./Commande.md) | Peut encapsuler les requêtes |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Stratégie](./Strategie.md)
