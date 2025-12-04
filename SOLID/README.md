# 🎯 Principes SOLID

> Les 5 principes fondamentaux de la conception orientée objet

[⬅️ Retour à l'Index](../INDEX.md)

---

## 📖 Introduction

**SOLID** est un acronyme représentant 5 principes de conception qui rendent le code:
- Plus **compréhensible**
- Plus **flexible**
- Plus **maintenable**

Ces principes ont été popularisés par **Robert C. Martin** (Uncle Bob).

---

## 🔤 L'Acronyme SOLID

| Lettre | Principe | En bref |
|--------|----------|---------|
| **S** | [Single Responsibility](#s---single-responsibility-principle-srp) | Une classe = une responsabilité |
| **O** | [Open/Closed](#o---openclosed-principle-ocp) | Ouvert à l'extension, fermé à la modification |
| **L** | [Liskov Substitution](#l---liskov-substitution-principle-lsp) | Sous-types substituables |
| **I** | [Interface Segregation](#i---interface-segregation-principle-isp) | Interfaces spécifiques |
| **D** | [Dependency Inversion](#d---dependency-inversion-principle-dip) | Dépendre des abstractions |

---

## S - Single Responsibility Principle (SRP)

### 📋 Définition

> **"Une classe ne devrait avoir qu'une seule raison de changer."**

Une classe doit avoir **une seule responsabilité**, c'est-à-dire être responsable d'un seul aspect du système.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│              SINGLE RESPONSIBILITY PRINCIPLE                │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLE SRP                                              │
│   ┌──────────────────────────────┐                          │
│   │         Employe              │                          │
│   ├──────────────────────────────┤  3 raisons de changer:   │
│   │ +calculerPaie()              │◄─ Logique de paie        │
│   │ +sauvegarder()               │◄─ Persistence            │
│   │ +genererRapport()            │◄─ Format du rapport      │
│   └──────────────────────────────┘                          │
│                                                             │
│   ✅ RESPECTE SRP                                           │
│   ┌────────────────┐  ┌────────────────┐  ┌────────────────┐│
│   │    Employe     │  │CalculateurPaie │  │EmployeRepos.   ││
│   ├────────────────┤  ├────────────────┤  ├────────────────┤│
│   │ -nom           │  │+calculer()     │  │+sauvegarder()  ││
│   │ -salaire       │  └────────────────┘  │+charger()      ││
│   └────────────────┘                      └────────────────┘│
│          │                                                  │
│          │         ┌─────────────────┐                      │
│          └────────►│GenerateurRapport│                      │
│                    ├─────────────────┤                      │
│                    │+generer()       │                      │
│                    └─────────────────┘                      │
│                                                             │
│   Chaque classe = 1 responsabilité = 1 raison de changer    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ VIOLE SRP: Multiple responsabilités
class Facture {
private:
    std::vector<LigneFacture> lignes;
    
public:
    // Responsabilité 1: Calculs
    double calculerTotal() { /* ... */ }
    
    // Responsabilité 2: Persistence
    void sauvegarderEnBD() { /* ... */ }
    
    // Responsabilité 3: Présentation
    void imprimerPDF() { /* ... */ }
    void envoyerEmail() { /* ... */ }
};

// ✅ RESPECTE SRP: Séparation des responsabilités
class Facture {
private:
    std::vector<LigneFacture> lignes;
    
public:
    double calculerTotal() const {
        double total = 0;
        for (const auto& ligne : lignes) {
            total += ligne.getSousTotal();
        }
        return total;
    }
    
    const std::vector<LigneFacture>& getLignes() const { 
        return lignes; 
    }
};

class FactureRepository {
public:
    void sauvegarder(const Facture& facture) {
        // Logique de persistence
    }
    
    Facture charger(int id) {
        // Logique de chargement
    }
};

class FacturePrinter {
public:
    void imprimerPDF(const Facture& facture) {
        // Logique d'impression PDF
    }
    
    void imprimerHTML(const Facture& facture) {
        // Logique d'impression HTML
    }
};

class FactureNotifier {
public:
    void envoyerEmail(const Facture& facture, const std::string& dest) {
        // Logique d'envoi email
    }
};
```

### ✅ Avantages
- Classes plus petites et focalisées
- Plus facile à tester
- Moins de conflits lors des merges
- Meilleure réutilisation

### ❌ Signaux d'alerte
- Classe avec beaucoup de méthodes
- Méthodes qui ne sont pas liées
- Difficile de nommer la classe de façon précise
- Plusieurs développeurs modifient souvent la même classe

---

## O - Open/Closed Principle (OCP)

### 📋 Définition

> **"Les entités logicielles doivent être ouvertes à l'extension, mais fermées à la modification."**

On doit pouvoir **ajouter** des fonctionnalités sans **modifier** le code existant.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│                 OPEN/CLOSED PRINCIPLE                       │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLE OCP (modifier pour étendre)                      │
│   ┌──────────────────────────────────┐                      │
│   │      CalculateurAire             │                      │
│   ├──────────────────────────────────┤                      │
│   │ +calculer(formes[]) {            │                      │
│   │   for (forme : formes) {         │                      │
│   │     if (forme.type == "cercle")  │◄─ Modifier pour      │
│   │       ...                        │   chaque nouvelle    │
│   │     else if (type == "carre")    │   forme!             │
│   │       ...                        │                      │
│   │   }                              │                      │
│   │ }                                │                      │
│   └──────────────────────────────────┘                      │
│                                                             │
│   ✅ RESPECTE OCP (étendre sans modifier)                   │
│                                                             │
│                  ┌──────────────┐                           │
│                  │ «interface»  │                           │
│                  │    Forme     │◄─── FERMÉ à modification  │
│                  ├──────────────┤                           │
│                  │ +getAire()   │                           │
│                  └──────┬───────┘                           │
│           ┌─────────────┼─────────────┐                     │
│           │             │             │                     │
│      ┌────▼─────┐   ┌────▼─────┐   ┌────▼─────┐             │
│      │ Cercle   │   │  Carré   │   │Triangle  │ ◄─ OUVERT   │
│      ├──────────┤   ├──────────┤   ├──────────┤ à extension │
│      │+getAire()│   │+getAire()│   │+getAire()│             │
│      └──────────┘   └──────────┘   └──────────┘             │
│                                                             │
│   Nouvelle forme? → Ajouter classe, PAS modifier!           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ VIOLE OCP: Doit modifier pour ajouter un nouveau type
class CalculateurRemise {
public:
    double calculer(const std::string& typeClient, double montant) {
        if (typeClient == "regulier") {
            return montant * 0.05;  // 5%
        } else if (typeClient == "premium") {
            return montant * 0.10;  // 10%
        } else if (typeClient == "vip") {
            return montant * 0.20;  // 20%
        }
        // Nouveau type = modifier cette méthode!
        return 0;
    }
};

// ✅ RESPECTE OCP: Extension par polymorphisme
class IRemise {
public:
    virtual double calculer(double montant) const = 0;
    virtual ~IRemise() = default;
};

class RemiseRegulier : public IRemise {
public:
    double calculer(double montant) const override {
        return montant * 0.05;
    }
};

class RemisePremium : public IRemise {
public:
    double calculer(double montant) const override {
        return montant * 0.10;
    }
};

class RemiseVIP : public IRemise {
public:
    double calculer(double montant) const override {
        return montant * 0.20;
    }
};

// Nouveau type de remise? Créer une nouvelle classe!
class RemiseEtudiant : public IRemise {
public:
    double calculer(double montant) const override {
        return montant * 0.15;
    }
};

// Calculateur ne change JAMAIS
class CalculateurPrix {
public:
    double calculerPrixFinal(double montant, const IRemise& remise) {
        return montant - remise.calculer(montant);
    }
};
```

### ✅ Techniques pour respecter OCP
- **Polymorphisme** - Classes abstraites et interfaces
- **Patron Stratégie** - Encapsuler les algorithmes
- **Patron Décorateur** - Ajouter des comportements
- **Composition** - Préférer à l'héritage

### ❌ Signaux d'alerte
- Beaucoup de `if/else` ou `switch` sur des types
- Ajout de fonctionnalité = modification de code existant
- Peur de casser quelque chose en ajoutant du code

---

## L - Liskov Substitution Principle (LSP)

### 📋 Définition

> **"Les objets d'une classe dérivée doivent pouvoir remplacer les objets de la classe de base sans altérer le comportement du programme."**

Si `S` est un sous-type de `T`, alors les objets de type `T` peuvent être remplacés par des objets de type `S` sans briser le programme.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│              LISKOV SUBSTITUTION PRINCIPLE                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLE LSP: Carré hérite de Rectangle                   │
│                                                             │
│   ┌──────────────┐                                          │
│   │  Rectangle   │    Client utilise Rectangle              │
│   ├──────────────┤    rect.setLargeur(5)                    │
│   │ -largeur     │    rect.setHauteur(10)                   │
│   │ -hauteur     │    assert(rect.getAire() == 50)  SUCCÈS! │
│   ├──────────────┤                                          │
│   │+setLargeur() │                                          │
│   │+setHauteur() │    Mais si rect est un Carré?            │
│   │+getAire()    │    carre.setLargeur(5)  → 5x5            │
│   └──────┬───────┘    carre.setHauteur(10) → 10x10          │
│          │            assert(aire == 50)  ÉCHEC!            │
│          │                                                  │
│     ┌────▼────┐                                             │
│     │  Carré  │  ◄── Carré casse le contrat!                │
│     ├─────────┤      setLargeur modifie aussi hauteur       │
│     │+setLar..│                                             │
│     │+setHau..│                                             │
│     └─────────┘                                             │
│                                                             │
│   ✅ RESPECTE LSP: Hiérarchie correcte                      │
│                                                             │
│                  ┌──────────────┐                           │
│                  │ «interface»  │                           │
│                  │    Forme     │                           │
│                  ├──────────────┤                           │
│                  │ +getAire()   │                           │
│                  └──────┬───────┘                           │
│           ┌─────────────┼─────────────┐                     │
│      ┌────▼─────┐              ┌──────▼─────┐               │
│      │Rectangle │              │   Carré    │               │
│      ├──────────┤              ├────────────┤               │
│      │-largeur  │              │ -cote      │               │
│      │-hauteur  │              ├────────────┤               │
│      └──────────┘              │+getAire()  │               │
│                                └────────────┘               │
│                                                             │
│   Pas d'héritage Carré→Rectangle, interfaces séparées       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ VIOLE LSP: L'oiseau qui ne vole pas
class Oiseau {
public:
    virtual void voler() {
        std::cout << "Je vole!" << std::endl;
    }
};

class Pingouin : public Oiseau {
public:
    void voler() override {
        throw std::runtime_error("Je ne peux pas voler!");
        // VIOLATION! Le client s'attend à ce que voler() fonctionne
    }
};

void faireMigrer(Oiseau& oiseau) {
    oiseau.voler();  // BOOM si c'est un Pingouin!
}

// ✅ RESPECTE LSP: Hiérarchie correcte
class Oiseau {
public:
    virtual void manger() = 0;
    virtual ~Oiseau() = default;
};

class OiseauVolant : public Oiseau {
public:
    virtual void voler() = 0;
};

class OiseauNonVolant : public Oiseau {
    // Pas de méthode voler()
};

class Aigle : public OiseauVolant {
public:
    void manger() override { std::cout << "Mange de la viande" << std::endl; }
    void voler() override { std::cout << "Plane majestueusement" << std::endl; }
};

class Pingouin : public OiseauNonVolant {
public:
    void manger() override { std::cout << "Mange du poisson" << std::endl; }
    void nager() { std::cout << "Nage rapidement" << std::endl; }
};

// Migration ne concerne que les oiseaux volants
void faireMigrer(OiseauVolant& oiseau) {
    oiseau.voler();  // Garanti de fonctionner!
}
```

### 📏 Règles du LSP

| Règle | Description |
|-------|-------------|
| **Préconditions** | Ne peuvent pas être renforcées dans le sous-type |
| **Postconditions** | Ne peuvent pas être affaiblies dans le sous-type |
| **Invariants** | Doivent être préservés |
| **Contrainte historique** | Les sous-types ne peuvent pas ajouter de méthodes qui modifient l'état d'une manière interdite par le type de base |

### ❌ Signaux d'alerte
- Méthode override qui lance une exception non prévue
- Méthode override qui ne fait rien
- Vérification du type concret avant appel (`instanceof`, `dynamic_cast`)
- Classes dérivées qui "cassent" les attentes

---

## I - Interface Segregation Principle (ISP)

### 📋 Définition

> **"Les clients ne devraient pas être forcés de dépendre d'interfaces qu'ils n'utilisent pas."**

Préférer plusieurs **interfaces spécifiques** plutôt qu'une seule interface générale.

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│             INTERFACE SEGREGATION PRINCIPLE                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLE ISP: Interface trop grosse                       │
│                                                             │
│   ┌────────────────────────┐                                │
│   │    «interface»         │                                │
│   │       Machine          │                                │
│   ├────────────────────────┤                                │
│   │ +imprimer()            │                                │
│   │ +scanner()             │                                │
│   │ +faxer()               │                                │
│   │ +agrafer()             │                                │
│   └───────────┬────────────┘                                │
│               │                                             │
│        ┌──────┴──────┐                                      │
│        │             │                                      │
│   ┌────▼────┐   ┌────▼─────────┐                            │
│   │Imprimante│  │ImprimanteSimple│                          │
│   │MultiFunc│   ├──────────────┤                            │
│   └─────────┘   │+scanner()    │ ◄── Doit implémenter       │
│                 │ { VIDE! }    │     des méthodes inutiles! │
│                 │+faxer()      │                            │
│                 │ { VIDE! }    │                            │
│                 └──────────────┘                            │
│                                                             │
│   ✅ RESPECTE ISP: Interfaces ségrégées                     │
│                                                             │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│   │«interface» │  │«interface» │  │«interface» │            │
│   │ Imprimante │  │  Scanner   │  │    Fax     │            │
│   ├────────────┤  ├────────────┤  ├────────────┤            │
│   │+imprimer() │  │+scanner()  │  │+faxer()    │            │
│   └─────┬──────┘  └─────┬──────┘  └─────┬──────┘            │
│         │               │               │                   │
│         └───────────────┼───────────────┘                   │
│                         │                                   │
│                    ┌────▼─────┐                             │
│                    │MultiFunc │ Implémente seulement        │
│                    │          │ ce dont elle a besoin       │
│                    └──────────┘                             │
│                                                             │
│   ┌────────────────┐                                        │
│   │ImprimanteSimple│ N'implémente que Imprimante            │
│   └────────────────┘                                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ VIOLE ISP: Interface trop large
class ITravailleur {
public:
    virtual void travailler() = 0;
    virtual void manger() = 0;
    virtual void dormir() = 0;
};

class Humain : public ITravailleur {
public:
    void travailler() override { /* ... */ }
    void manger() override { /* ... */ }
    void dormir() override { /* ... */ }
};

class Robot : public ITravailleur {
public:
    void travailler() override { /* ... */ }
    void manger() override { /* PROBLÈME: Robot ne mange pas! */ }
    void dormir() override { /* PROBLÈME: Robot ne dort pas! */ }
};

// ✅ RESPECTE ISP: Interfaces ségrégées
class IExecutable {
public:
    virtual void travailler() = 0;
    virtual ~IExecutable() = default;
};

class IMangeable {
public:
    virtual void manger() = 0;
    virtual ~IMangeable() = default;
};

class IDormable {
public:
    virtual void dormir() = 0;
    virtual ~IDormable() = default;
};

// Humain implémente tout
class Humain : public IExecutable, public IMangeable, public IDormable {
public:
    void travailler() override { std::cout << "Travaille..." << std::endl; }
    void manger() override { std::cout << "Mange..." << std::endl; }
    void dormir() override { std::cout << "Dort..." << std::endl; }
};

// Robot n'implémente que ce qui est pertinent
class Robot : public IExecutable {
public:
    void travailler() override { std::cout << "Travaille 24/7..." << std::endl; }
    // Pas de manger() ni dormir()!
};

// Le gestionnaire de travail n'a besoin que de IExecutable
class GestionnaireTravail {
public:
    void assignerTache(IExecutable& travailleur) {
        travailleur.travailler();
    }
};
```

### ✅ Avantages
- Classes implémentent seulement ce qu'elles utilisent
- Interfaces plus stables (moins de raisons de changer)
- Meilleure lisibilité des dépendances
- Facilite les tests (mocks plus simples)

### ❌ Signaux d'alerte
- Méthodes d'interface qui lancent `NotImplementedException`
- Méthodes d'interface vides
- Clients qui n'utilisent qu'une partie de l'interface
- Interfaces avec beaucoup de méthodes (> 5)

---

## D - Dependency Inversion Principle (DIP)

### 📋 Définition

> 1. **"Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux doivent dépendre d'abstractions."**
> 2. **"Les abstractions ne doivent pas dépendre des détails. Les détails doivent dépendre des abstractions."**

### 📊 Schéma

```
┌─────────────────────────────────────────────────────────────┐
│              DEPENDENCY INVERSION PRINCIPLE                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ❌ VIOLE DIP: Haut niveau dépend de bas niveau            │
│                                                             │
│   ┌─────────────────┐                                       │
│   │  ServiceVente   │ ◄── Module HAUT niveau                │
│   └────────┬────────┘                                       │
│            │ dépend directement                             │
│            ▼                                                │
│   ┌─────────────────┐                                       │
│   │ MySQLDatabase   │ ◄── Module BAS niveau                 │
│   └─────────────────┘                                       │
│                                                             │
│   Problème: Changer de BD = modifier ServiceVente           │
│                                                             │
│   ✅ RESPECTE DIP: Tous dépendent d'abstractions            │
│                                                             │
│   ┌─────────────────┐                                       │
│   │  ServiceVente   │ ◄── Module HAUT niveau                │
│   └────────┬────────┘                                       │
│            │ dépend de                                      │
│            ▼                                                │
│   ┌─────────────────┐                                       │
│   │  «interface»    │ ◄── ABSTRACTION (possédée par         │
│   │   IDatabase     │     le haut niveau)                   │
│   └────────┬────────┘                                       │
│            ▲ implémente                                     │
│            │                                                │
│   ┌────────┴────────┐                                       │
│   │                 │                                       │
│   ▼                 ▼                                       │
│ ┌──────────┐  ┌──────────┐                                  │
│ │  MySQL   │  │ MongoDB  │ ◄── Modules BAS niveau           │
│ │ Database │  │ Database │     dépendent de l'abstraction   │
│ └──────────┘  └──────────┘                                  │
│                                                             │
│   La flèche de dépendance est INVERSÉE!                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 💻 Exemple de Code

```cpp
// ❌ VIOLE DIP: Couplage direct au module de bas niveau
class MySQLDatabase {
public:
    void sauvegarder(const std::string& data) {
        std::cout << "Sauvegarde MySQL: " << data << std::endl;
    }
};

class ServiceCommande {
private:
    MySQLDatabase* db;  // Dépendance CONCRÈTE!
    
public:
    ServiceCommande() {
        db = new MySQLDatabase();  // Création directe!
    }
    
    void creerCommande(const std::string& details) {
        // Logique métier...
        db->sauvegarder(details);
    }
};

// ✅ RESPECTE DIP: Dépend d'une abstraction
class IDatabase {
public:
    virtual void sauvegarder(const std::string& data) = 0;
    virtual std::string charger(int id) = 0;
    virtual ~IDatabase() = default;
};

class MySQLDatabase : public IDatabase {
public:
    void sauvegarder(const std::string& data) override {
        std::cout << "Sauvegarde MySQL: " << data << std::endl;
    }
    std::string charger(int id) override { return ""; }
};

class MongoDatabase : public IDatabase {
public:
    void sauvegarder(const std::string& data) override {
        std::cout << "Sauvegarde MongoDB: " << data << std::endl;
    }
    std::string charger(int id) override { return ""; }
};

class ServiceCommande {
private:
    IDatabase* db;  // Dépendance ABSTRAITE!
    
public:
    // Injection de dépendance via constructeur
    ServiceCommande(IDatabase* database) : db(database) {}
    
    void creerCommande(const std::string& details) {
        // Logique métier...
        db->sauvegarder(details);  // Fonctionne avec n'importe quelle BD
    }
};

// Utilisation avec injection de dépendance
int main() {
    // Production: MySQL
    MySQLDatabase mysqlDb;
    ServiceCommande serviceMySQL(&mysqlDb);
    
    // Ou MongoDB
    MongoDatabase mongoDb;
    ServiceCommande serviceMongo(&mongoDb);
    
    // Ou Mock pour tests!
    // MockDatabase mockDb;
    // ServiceCommande serviceTest(&mockDb);
}
```

### 📋 Techniques d'Injection de Dépendance

```cpp
class Service {
    IDatabase* db;
    
public:
    // 1. Injection par constructeur (recommandé)
    Service(IDatabase* database) : db(database) {}
    
    // 2. Injection par setter
    void setDatabase(IDatabase* database) { db = database; }
    
    // 3. Injection par méthode
    void traiter(IDatabase* database) {
        database->sauvegarder("...");
    }
};
```

### ✅ Avantages
- Facilite les tests unitaires (injection de mocks)
- Découplage entre modules
- Flexibilité pour changer d'implémentation
- Code plus modulaire

### ❌ Signaux d'alerte
- `new` dans les constructeurs de classes de haut niveau
- Classes qui créent leurs propres dépendances
- Imports/includes de classes concrètes de bas niveau
- Difficile à tester sans base de données réelle

---

## 🔗 Relations entre Principes SOLID

```
┌─────────────────────────────────────────────────────────────┐
│                   RELATIONS SOLID                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                        ┌─────┐                              │
│                        │ SRP │                              │
│                        └──┬──┘                              │
│                           │ permet                          │
│                           ▼                                 │
│   ┌─────┐           ┌─────────┐           ┌─────┐           │
│   │ ISP │◄──────────│   OCP   │──────────►│ LSP │           │
│   └──┬──┘  petites  └────┬────┘ substituer└──┬──┘           │
│      │   interfaces      │  sans modifier    │              │
│      │                   │                   │              │
│      └───────────────────┼───────────────────┘              │
│                          │                                  │
│                          │ utilise                          │
│                          ▼                                  │
│                      ┌─────┐                                │
│                      │ DIP │                                │
│                      └─────┘                                │
│                                                             │
│   SRP → Classes focalisées permettent OCP                   │
│   ISP → Petites interfaces facilitent LSP                   │
│   OCP → Nécessite abstractions (DIP)                        │
│   LSP → Garantit que OCP fonctionne                         │
│   DIP → Fondation pour tous les autres                      │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 📚 Résumé

| Principe | Problème résolu | Solution |
|----------|-----------------|----------|
| **SRP** | Classes qui font trop | 1 classe = 1 responsabilité |
| **OCP** | Modifier pour étendre | Étendre sans modifier |
| **LSP** | Sous-types incompatibles | Substitution transparente |
| **ISP** | Interfaces trop larges | Interfaces spécifiques |
| **DIP** | Dépendances concrètes | Dépendre des abstractions |

---

## 💡 Conseils Pratiques

1. **Ne pas sur-engineerer** - Appliquer SOLID quand c'est nécessaire
2. **Commencer par SRP** - C'est la base des autres principes
3. **Identifier les points de variation** - C'est là qu'OCP est utile
4. **Tester, tester, tester** - Les tests révèlent les violations
5. **Refactorer progressivement** - Pas besoin de tout refaire d'un coup

---

[⬅️ Retour à l'Index](../INDEX.md) | [➡️ Patrons GRASP](../GRASP/README.md)
