# 🎓 Patrons GRASP

> **G**eneral **R**esponsibility **A**ssignment **S**oftware **P**atterns

[⬅️ Retour à l'Index](../INDEX.md)

---

## 📖 Introduction

Les patrons GRASP sont des **principes fondamentaux** pour attribuer les responsabilités aux classes et objets en conception orientée objet. Ils ont été introduits par **Craig Larman** dans son livre "Applying UML and Patterns".

> 💡 **GRASP = Guide pour savoir QUOI mettre DANS quelle classe**

---

## 🎯 Les 9 Patrons GRASP

| Patron | Objectif Principal |
|--------|-------------------|
| [Expert en Information](#1-expert-en-information-information-expert) | Qui a les données? |
| [Créateur](#2-créateur-creator) | Qui crée quoi? |
| [Contrôleur](#3-contrôleur-controller) | Qui gère les événements système? |
| [Faible Couplage](#4-faible-couplage-low-coupling) | Comment minimiser les dépendances? |
| [Forte Cohésion](#5-forte-cohésion-high-cohesion) | Comment garder les classes focalisées? |
| [Polymorphisme](#6-polymorphisme-polymorphism) | Comment gérer les variantes? |
| [Fabrication Pure](#7-fabrication-pure-pure-fabrication) | Quand créer des classes artificielles? |
| [Indirection](#8-indirection-indirection) | Comment découpler avec un intermédiaire? |
| [Protection des Variations](#9-protection-des-variations-protected-variations) | Comment isoler les changements? |

---

## 1. Expert en Information (Information Expert)

### 🎯 Problème
À quelle classe attribuer une responsabilité?

### ✅ Solution
**Attribuer la responsabilité à la classe qui possède l'information nécessaire pour l'accomplir.**

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                      EXPERT EN INFORMATION                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Question: "Qui calcule le total d'une commande?"          │
│                                                             │
│   ┌─────────────┐         ┌─────────────┐                   │
│   │  Commande   │◄────────│LigneCommande│                   │
│   ├─────────────┤         ├─────────────┤                   │
│   │ -lignes[]   │         │ -produit    │                   │
│   │ -date       │         │ -quantite   │                   │
│   ├─────────────┤         ├─────────────┤                   │
│   │+getTotal()  │         │+getSousTotal()                  │
│   └─────────────┘         └─────────────┘                   │
│         │                        │                          │
│         │    Délègue à           │                          │
│         └────────────────────────┘                          │
│                                                             │
│   Commande connaît ses lignes → Expert pour le total        │
│   LigneCommande connaît quantité/prix → Expert sous-total   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ MAUVAIS: La logique est dans une classe externe
class CalculateurCommande {
public:
    double calculerTotal(Commande& cmd) {
        double total = 0;
        for (auto& ligne : cmd.getLignes()) {
            total += ligne.getProduit().getPrix() * ligne.getQuantite();
        }
        return total;
    }
};

// ✅ BON: Chaque classe calcule ce qu'elle connaît
class LigneCommande {
private:
    Produit* produit;
    int quantite;
public:
    double getSousTotal() const {
        return produit->getPrix() * quantite;  // Expert: connaît produit et quantité
    }
};

class Commande {
private:
    std::vector<LigneCommande> lignes;
public:
    double getTotal() const {
        double total = 0;
        for (const auto& ligne : lignes) {
            total += ligne.getSousTotal();  // Délègue à l'expert
        }
        return total;
    }
};
```

### ✅ Avantages
- Encapsulation maintenue
- Information et comportement ensemble
- Couplage minimal

### ❌ Inconvénients
- Peut créer des classes trop "lourdes"
- Parfois l'expert naturel n'est pas le bon choix (voir Fabrication Pure)

---

## 2. Créateur (Creator)

### 🎯 Problème
Quelle classe doit être responsable de créer une nouvelle instance d'une autre classe?

### ✅ Solution
Attribuer à **B** la responsabilité de créer **A** si:
- B **contient** ou **agrège** A
- B **enregistre** A
- B **utilise étroitement** A
- B **possède les données** pour initialiser A

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                         CRÉATEUR                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Qui crée les LigneCommande?                               │
│                                                             │
│   ┌─────────────┐    contient    ┌─────────────────┐        │
│   │  Commande   │◆──────────────│  LigneCommande  │        │
│   ├─────────────┤                ├─────────────────┤        │
│   │ -lignes[]   │                │ -produit        │        │
│   ├─────────────┤                │ -quantite       │        │
│   │+ajouterLigne│                └─────────────────┘        │
│   │  (p, qty)   │                                           │
│   └─────────────┘                                           │
│         │                                                   │
│         │ crée                                              │
│         ▼                                                   │
│   new LigneCommande(p, qty)                                 │
│                                                             │
│   Commande contient les lignes → Créateur naturel           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ MAUVAIS: Un gestionnaire externe crée tout
class GestionnaireCommandes {
public:
    void ajouterProduit(Commande& cmd, Produit* p, int qty) {
        LigneCommande* ligne = new LigneCommande(p, qty);
        cmd.getLignes().push_back(ligne);
    }
};

// ✅ BON: Commande crée ses propres lignes (elle les contient)
class Commande {
private:
    std::vector<LigneCommande*> lignes;
public:
    void ajouterProduit(Produit* produit, int quantite) {
        // Commande est le créateur naturel car elle contient les lignes
        LigneCommande* ligne = new LigneCommande(produit, quantite);
        lignes.push_back(ligne);
    }
};
```

### ✅ Avantages
- Faible couplage (pas de classe tierce)
- Logique de création proche des données

### ❌ Inconvénients
- Création complexe → utiliser une **Fabrique**
- Si les données d'initialisation viennent d'ailleurs

---

## 3. Contrôleur (Controller)

### 🎯 Problème
Quel objet reçoit et coordonne les événements système?

### ✅ Solution
Attribuer la responsabilité à:
1. **Contrôleur de façade** : représente le système global (ex: `SystèmeJeu`)
2. **Contrôleur de cas d'utilisation** : un par scénario (ex: `GérerCommandeHandler`)

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                       CONTRÔLEUR                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────┐    événement    ┌────────────────┐           │
│   │   UI     │────────────────►│  Contrôleur    │           │
│   │ (Vue)    │                 │                │           │
│   └──────────┘                 ├────────────────┤           │
│                                │+passerCommande()           │
│                                │+annulerCommande()          │
│                                └───────┬────────┘           │
│                                        │                    │
│                          ┌─────────────┼─────────────┐      │
│                          │             │             │      │
│                          ▼             ▼             ▼      │
│                    ┌─────────┐   ┌─────────┐   ┌─────────┐  │
│                    │Commande │   │ Client  │   │Inventaire│ │
│                    └─────────┘   └─────────┘   └─────────┘  │
│                                                             │
│   Le Contrôleur coordonne mais NE FAIT PAS le travail       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ✅ Contrôleur de façade
class SystemeVente {
private:
    Catalogue* catalogue;
    Inventaire* inventaire;
    Caisse* caisse;
    Vente* venteEnCours;
    
public:
    // Reçoit les événements système de l'UI
    void demarrerVente() {
        venteEnCours = new Vente();
    }
    
    void entrerArticle(const std::string& codeBarres, int quantite) {
        // Coordonne sans faire le travail lui-même
        Produit* p = catalogue->trouverProduit(codeBarres);
        venteEnCours->ajouterLigne(p, quantite);
        inventaire->decrementer(codeBarres, quantite);
    }
    
    void terminerVente() {
        double total = venteEnCours->getTotal();
        caisse->afficherTotal(total);
    }
};

// Usage depuis l'UI
class InterfaceVente {
private:
    SystemeVente* controleur;  // Délègue au contrôleur
public:
    void onBoutonScanner(const std::string& code) {
        controleur->entrerArticle(code, 1);
    }
};
```

### ✅ Avantages
- Sépare UI de la logique métier
- Point d'entrée unique et clair
- Facilite les tests (on teste le contrôleur sans UI)

### ❌ Inconvénients
- Peut devenir un "God Object" si mal conçu
- Trop de responsabilités → découper en plusieurs contrôleurs

---

## 4. Faible Couplage (Low Coupling)

### 🎯 Problème
Comment réduire l'impact des changements et favoriser la réutilisation?

### ✅ Solution
**Minimiser les dépendances** entre les classes. Une classe fortement couplée:
- Est difficile à comprendre isolément
- Est difficile à réutiliser
- Est affectée par les changements des autres

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                     FAIBLE COUPLAGE                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ FORT COUPLAGE              ✅ FAIBLE COUPLAGE          │
│                                                             │
│   ┌─────┐                      ┌─────┐                      │
│   │  A  │◄─────┐               │  A  │                      │
│   └──┬──┘      │               └──┬──┘                      │
│      │         │                  │                         │
│      ▼         │                  ▼                         │
│   ┌─────┐      │               ┌─────┐                      │
│   │  B  │◄─────┤               │  B  │ (via interface)      │
│   └──┬──┘      │               └─────┘                      │
│      │         │                                            │
│      ▼         │                                            │
│   ┌─────┐      │                                            │
│   │  C  │◄─────┘                                            │
│   └──┬──┘                                                   │
│      │                                                      │
│      ▼                                                      │
│   ┌─────┐                                                   │
│   │  D  │                                                   │
│   └─────┘                                                   │
│                                                             │
│   A dépend de tout!             A dépend d'une abstraction  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ FORT COUPLAGE: Commande dépend de classes concrètes
class Commande {
private:
    MySQLDatabase* db;           // Couplage à MySQL
    SMTPEmailService* email;     // Couplage à SMTP
    StripePayment* paiement;     // Couplage à Stripe
public:
    void confirmer() {
        db->sauvegarder(this);
        email->envoyer("Confirmation...");
        paiement->debiter(total);
    }
};

// ✅ FAIBLE COUPLAGE: Dépend d'abstractions
class IBaseDonnees {
public:
    virtual void sauvegarder(Commande* cmd) = 0;
};

class INotification {
public:
    virtual void envoyer(const std::string& msg) = 0;
};

class IPaiement {
public:
    virtual void debiter(double montant) = 0;
};

class Commande {
private:
    IBaseDonnees* db;      // Interface!
    INotification* notif;  // Interface!
    IPaiement* paiement;   // Interface!
public:
    Commande(IBaseDonnees* d, INotification* n, IPaiement* p)
        : db(d), notif(n), paiement(p) {}
        
    void confirmer() {
        db->sauvegarder(this);
        notif->envoyer("Confirmation...");
        paiement->debiter(total);
    }
};
```

### 📏 Types de Couplage (du pire au meilleur)

| Type | Description | Exemple |
|------|-------------|---------|
| Contenu | A modifie les données internes de B | `b.data = 5;` |
| Commun | A et B partagent des données globales | Variables globales |
| Contrôle | A contrôle le comportement de B | Paramètre flag |
| Données | A passe des données à B | Paramètres simples |
| Message | A appelle une méthode de B | `b.faire()` |
| Aucun | A et B sont indépendants | ✅ Idéal |

---

## 5. Forte Cohésion (High Cohesion)

### 🎯 Problème
Comment garder les classes focalisées, compréhensibles et gérables?

### ✅ Solution
Une classe devrait avoir des **responsabilités étroitement liées**. Éviter les classes qui font "trop de choses".

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                     FORTE COHÉSION                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ FAIBLE COHÉSION                                        │
│   ┌─────────────────────────────────────┐                   │
│   │          GodClass                   │                   │
│   ├─────────────────────────────────────┤                   │
│   │ +gererUtilisateur()                 │                   │
│   │ +calculerTaxes()                    │  Fait TOUT!       │
│   │ +envoyerEmail()                     │                   │
│   │ +genererRapport()                   │                   │
│   │ +validerPaiement()                  │                   │
│   └─────────────────────────────────────┘                   │
│                                                             │
│   ✅ FORTE COHÉSION                                         │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│   │Utilisateur   │  │CalculTaxes   │  │EmailService  │      │
│   ├──────────────┤  ├──────────────┤  ├──────────────┤      │
│   │+creer()      │  │+calculerTPS()│  │+envoyer()    │      │
│   │+modifier()   │  │+calculerTVQ()│  │+formater()   │      │
│   │+supprimer()  │  │+appliquer()  │  │+valider()    │      │
│   └──────────────┘  └──────────────┘  └──────────────┘      │
│                                                             │
│   Chaque classe = UNE responsabilité claire                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ FAIBLE COHÉSION: La classe fait tout
class Employe {
private:
    std::string nom;
    double salaire;
    
public:
    // Données employé - OK
    std::string getNom() { return nom; }
    
    // Calcul paie - OK (lié à l'employé)
    double calculerPaie() { return salaire; }
    
    // ❌ Génération PDF - pas lié!
    void genererFichePaiePDF() { /* ... */ }
    
    // ❌ Envoi email - pas lié!
    void envoyerFichePaieParEmail() { /* ... */ }
    
    // ❌ Sauvegarde BD - pas lié!
    void sauvegarderDansBaseDonnees() { /* ... */ }
};

// ✅ FORTE COHÉSION: Chaque classe a une responsabilité
class Employe {
private:
    std::string nom;
    double salaire;
public:
    std::string getNom() const { return nom; }
    double getSalaire() const { return salaire; }
};

class CalculateurPaie {
public:
    double calculer(const Employe& emp) {
        return emp.getSalaire();  // + déductions, bonus, etc.
    }
};

class GenerateurFichePaie {
public:
    void genererPDF(const Employe& emp, double montant) { /* ... */ }
};

class ServiceNotification {
public:
    void envoyerParEmail(const Employe& emp, const std::string& fichier) { /* ... */ }
};
```

### 📏 Mesurer la Cohésion

| Niveau | Description |
|--------|-------------|
| **Fonctionnelle** | Tous les éléments contribuent à UNE tâche ✅ |
| **Séquentielle** | Sortie d'un élément = entrée du suivant |
| **Communicationnelle** | Opèrent sur les mêmes données |
| **Temporelle** | Exécutés au même moment |
| **Coïncidentale** | Aucun lien - juste regroupés ❌ |

---

## 6. Polymorphisme (Polymorphism)

### 🎯 Problème
Comment gérer les comportements qui varient selon le type?

### ✅ Solution
Utiliser des **méthodes polymorphes** au lieu de conditionnelles (if/switch sur le type).

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                      POLYMORPHISME                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ AVEC IF/SWITCH                                         │
│   ┌─────────────────────────────────────┐                   │
│   │ if (type == "cercle")               │                   │
│   │     dessinerCercle();               │                   │
│   │ else if (type == "carre")           │                   │
│   │     dessinerCarre();                │                   │
│   │ else if (type == "triangle")        │                   │
│   │     dessinerTriangle();             │  Difficile à      │
│   │ // Ajouter nouvelle forme = modifier│  maintenir!       │
│   └─────────────────────────────────────┘                   │
│                                                             │
│   ✅ AVEC POLYMORPHISME                                     │
│                 ┌───────────┐                               │
│                 │«interface»│                               │
│                 │  Forme    │                               │
│                 ├───────────┤                               │
│                 │+dessiner()│                               │
│                 └────┬──────┘                               │
│           ┌──────────┼──────────┐                           │
│           │          │          │                           │
│      ┌────▼───┐ ┌────▼───┐ ┌────▼────┐                      │
│      │ Cercle │ │ Carré  │ │Triangle │                      │
│      ├────────┤ ├────────┤ ├─────────┤                      │
│      │dessiner│ │dessiner│ │dessiner │  Chacun sait         │
│      └────────┘ └────────┘ └─────────┘  comment faire!      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ MAUVAIS: Conditionnelle sur le type
class Paiement {
    std::string type;  // "carte", "paypal", "crypto"
    
    void traiter() {
        if (type == "carte") {
            // logique carte...
        } else if (type == "paypal") {
            // logique paypal...
        } else if (type == "crypto") {
            // logique crypto...
        }
        // Chaque nouveau type = modifier cette méthode!
    }
};

// ✅ BON: Polymorphisme
class IPaiement {
public:
    virtual void traiter() = 0;
    virtual ~IPaiement() = default;
};

class PaiementCarte : public IPaiement {
public:
    void traiter() override {
        std::cout << "Traitement carte bancaire..." << std::endl;
    }
};

class PaiementPayPal : public IPaiement {
public:
    void traiter() override {
        std::cout << "Redirection PayPal..." << std::endl;
    }
};

class PaiementCrypto : public IPaiement {
public:
    void traiter() override {
        std::cout << "Vérification blockchain..." << std::endl;
    }
};

// Utilisation - pas de if/switch!
void processerPaiement(IPaiement* paiement) {
    paiement->traiter();  // Le bon comportement automatiquement
}
```

---

## 7. Fabrication Pure (Pure Fabrication)

### 🎯 Problème
Que faire quand l'Expert crée un couplage élevé ou une faible cohésion?

### ✅ Solution
Créer une **classe artificielle** (qui n'existe pas dans le domaine métier) pour regrouper des responsabilités.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                   FABRICATION PURE                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Problème: Qui sauvegarde une Vente en BD?                 │
│                                                             │
│   ❌ Expert = Vente?                                        │
│   ┌─────────────────┐                                       │
│   │      Vente      │  Vente connaît ses données MAIS       │
│   ├─────────────────┤  ajouter la persistence =             │
│   │ +getTotal()     │  - Faible cohésion                    │
│   │ +sauvegarder()  │◄─── Couplage à la BD!                 │
│   │ +charger()      │                                       │
│   └─────────────────┘                                       │
│                                                             │
│   ✅ Fabrication Pure                                       │
│   ┌─────────────────┐     ┌─────────────────┐               │
│   │      Vente      │     │ VenteRepository │◄── Classe     │
│   ├─────────────────┤     ├─────────────────┤   artificielle│
│   │ +getTotal()     │     │ +sauvegarder()  │               │
│   │ +getLignes()    │     │ +charger()      │               │ 
│   └─────────────────┘     │ +supprimer()    │               │
│          │                └────────┬────────┘               │
│          │ utilise                 │                        │
│          └─────────────────────────┘                        │
│                                                             │
│   Vente = Domaine métier    Repository = Fabrication pure   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// Classe du domaine métier
class Vente {
private:
    std::vector<LigneVente> lignes;
    std::string date;
    
public:
    double getTotal() const { /* ... */ }
    std::vector<LigneVente>& getLignes() { return lignes; }
    std::string getDate() const { return date; }
    // PAS de logique de persistence ici!
};

// ✅ Fabrication Pure: Classe artificielle pour la persistence
class VenteRepository {
private:
    Database* db;
    
public:
    void sauvegarder(const Vente& vente) {
        std::string sql = "INSERT INTO ventes...";
        db->executer(sql);
    }
    
    Vente charger(int id) {
        std::string sql = "SELECT * FROM ventes WHERE id = " + std::to_string(id);
        // ...
    }
    
    void supprimer(int id) { /* ... */ }
};
```

### 📋 Exemples courants de Fabrications Pures
- **Repository** - Persistence des données
- **Service** - Logique métier complexe
- **Factory** - Création d'objets
- **Controller** - Coordination des événements
- **Adapter** - Conversion d'interfaces

---

## 8. Indirection (Indirection)

### 🎯 Problème
Comment découpler deux classes pour favoriser la réutilisation?

### ✅ Solution
Introduire un **objet intermédiaire** qui assure la médiation.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                      INDIRECTION                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ SANS INDIRECTION           ✅ AVEC INDIRECTION         │
│                                                             │
│   ┌─────────┐              ┌─────────┐                      │
│   │ Client  │              │ Client  │                      │
│   └────┬────┘              └────┬────┘                      │
│        │                        │                           │
│        │ couplage direct        │ couplage faible           │
│        │                        ▼                           │
│        │                   ┌─────────────┐                  │
│        │                   │ Adaptateur  │ ◄── Intermédiaire│
│        │                   └──────┬──────┘                  │
│        │                          │                         │
│        ▼                          ▼                         │
│   ┌─────────┐              ┌─────────────┐                  │
│   │ Service │              │ServiceExterne│                 │
│   │ Externe │              └─────────────┘                  │
│   └─────────┘                                               │
│                                                             │
│   "Tout problème peut être résolu par une indirection       │
│    supplémentaire" (sauf trop d'indirection!)               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// Service externe avec une interface complexe
class ServicePaiementExterne {
public:
    void initTransaction(const std::string& merchantId) { /* ... */ }
    void setAmount(int centimes) { /* ... */ }
    void setCardData(const std::string& encrypted) { /* ... */ }
    bool processSync() { /* ... */ return true; }
};

// ✅ Indirection: Adaptateur qui simplifie l'interface
class AdaptateurPaiement {
private:
    ServicePaiementExterne* service;
    std::string merchantId;
    
public:
    AdaptateurPaiement(const std::string& id) : merchantId(id) {
        service = new ServicePaiementExterne();
    }
    
    // Interface simplifiée pour notre application
    bool payer(double montant, const std::string& carteCryptee) {
        service->initTransaction(merchantId);
        service->setAmount(static_cast<int>(montant * 100));
        service->setCardData(carteCryptee);
        return service->processSync();
    }
};

// Client utilise l'adaptateur (indirection)
class Caisse {
private:
    AdaptateurPaiement* paiement;
    
public:
    void finaliserVente(double total, const std::string& carte) {
        if (paiement->payer(total, carte)) {
            std::cout << "Paiement accepté!" << std::endl;
        }
    }
};
```

---

## 9. Protection des Variations (Protected Variations)

### 🎯 Problème
Comment concevoir des systèmes stables face aux changements prévisibles?

### ✅ Solution
Identifier les **points de variation** et créer une **interface stable** autour.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                 PROTECTION DES VARIATIONS                   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Point de variation: Calcul des taxes (varie par région)   │
│                                                             │
│                    ┌───────────────────┐                    │
│                    │   «interface»     │ ◄── Interface      │
│                    │ ICalculateurTaxe  │     stable         │
│                    ├───────────────────┤                    │
│                    │ +calculer(montant)│                    │
│                    └─────────┬─────────┘                    │
│              ┌───────────────┼───────────────┐              │
│              │               │               │              │
│         ┌────▼────┐    ┌─────▼────┐    ┌─────▼────┐         │
│         │TaxeCanada│    │ TaxeUSA │    │ TaxeEU  │          │
│         ├─────────┤    ├──────────┤    ├─────────┤          │
│         │calculer()│   │calculer()│    │calculer()│         │
│         │TPS+TVQ   │   │ Sales Tax│    │   TVA   │          │
│         └─────────┘    └──────────┘    └─────────┘          │
│                                                             │
│   Nouvelle région? → Ajouter une classe, pas modifier!      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ✅ Interface stable (point de protection)
class ICalculateurTaxe {
public:
    virtual double calculer(double montant) const = 0;
    virtual ~ICalculateurTaxe() = default;
};

// Implémentations concrètes (points de variation)
class TaxeQuebec : public ICalculateurTaxe {
public:
    double calculer(double montant) const override {
        double tps = montant * 0.05;   // 5% TPS
        double tvq = montant * 0.09975; // 9.975% TVQ
        return tps + tvq;
    }
};

class TaxeOntario : public ICalculateurTaxe {
public:
    double calculer(double montant) const override {
        return montant * 0.13;  // 13% HST
    }
};

class TaxeAlberta : public ICalculateurTaxe {
public:
    double calculer(double montant) const override {
        return montant * 0.05;  // 5% GST seulement
    }
};

// Client protégé des variations
class Facture {
private:
    ICalculateurTaxe* calculateur;  // Dépend de l'interface
    double montant;
    
public:
    Facture(double m, ICalculateurTaxe* calc) 
        : montant(m), calculateur(calc) {}
    
    double getTotal() const {
        return montant + calculateur->calculer(montant);
    }
};
```

### 📋 Mécanismes de Protection
- Interfaces et classes abstraites
- Polymorphisme
- Standards et protocoles
- Encapsulation des données
- Injection de dépendances

---

## 🔗 Relations entre Patrons GRASP

```
┌─────────────────────────────────────────────────────────────┐
│                    RELATIONS GRASP                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Faible Couplage ◄──────────────────► Forte Cohésion       │
│         ▲                                    ▲              │
│         │ supporte                           │ supporte     │
│         │                                    │              │
│   ┌─────┴─────┐                        ┌─────┴─────┐        │
│   │Indirection│                        │Fabrication│        │
│   │           │                        │   Pure    │        │
│   └───────────┘                        └───────────┘        │
│         ▲                                    ▲              │
│         │                                    │              │
│         └──────── Protection des ────────────┘              │
│                    Variations                               │
│                        ▲                                    │
│                        │ utilise                            │
│                        │                                    │
│                  Polymorphisme                              │
│                                                             │
│   Expert ────────► base de Créateur et Contrôleur           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Résumé

| Patron | Question Clé | Réponse |
|--------|-------------|---------|
| Expert | Qui fait quoi? | Celui qui a l'info |
| Créateur | Qui crée quoi? | Celui qui contient/utilise |
| Contrôleur | Qui reçoit les événements? | Façade ou UC handler |
| Faible Couplage | Comment minimiser dépendances? | Interfaces, abstractions |
| Forte Cohésion | Comment rester focalisé? | Une responsabilité/classe |
| Polymorphisme | Comment gérer les variantes? | Méthodes virtuelles |
| Fabrication Pure | Si Expert pas idéal? | Classe artificielle |
| Indirection | Comment découpler? | Objet intermédiaire |
| Protection Variations | Comment isoler changements? | Interface stable |

---

[⬅️ Retour à l'Index](../INDEX.md) | [➡️ Principes SOLID](../SOLID/README.md)
