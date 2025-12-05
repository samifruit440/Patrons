# 📋 Patron de Méthode (Template Method)

> **Patron Comportemental** - Définit le squelette d'un algorithme dans une opération, en déléguant certaines étapes aux sous-classes.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Définir le **squelette d'un algorithme** dans une méthode, en laissant les sous-classes **redéfinir certaines étapes** sans changer la structure globale de l'algorithme.

---

## 🎯 Problème Résolu

- Comment définir un algorithme générique avec des étapes personnalisables?
- Comment éviter la duplication de code entre algorithmes similaires?
- Comment contrôler les points d'extension d'un algorithme?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                    PATRON DE MÉTHODE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│            ┌──────────────────────────────────┐                 │
│            │        «abstract»                │                 │
│            │       ClasseAbstraite            │                 │
│            ├──────────────────────────────────┤                 │
│            │ +methodeTemplate() {             │ ◄── FINALE      │
│            │     etape1();                    │     (non        │
│            │     etape2();        // abstraite│      modifiable)│
│            │     etape3();        // abstraite│                 │
│            │     hook();          // optionnel│                 │
│            │ }                                │                 │
│            │ +etape1() { ... }    // concrète │                 │
│            │ +etape2() = 0;       // abstraite│                 │
│            │ +etape3() = 0;       // abstraite│                 │
│            │ +hook() { }          // hook vide│                 │
│            └───────────────┬──────────────────┘                 │
│                            │                                    │
│              ┌─────────────┴─────────────┐                      │
│              │                           │                      │
│    ┌─────────▼─────────┐       ┌─────────▼─────────┐            │
│    │   ClasseConcreteA │       │   ClasseConcreteB │            │
│    ├───────────────────┤       ├───────────────────┤            │
│    │ +etape2() { ... } │       │ +etape2() { ... } │            │
│    │ +etape3() { ... } │       │ +etape3() { ... } │            │
│    │ +hook() { ... }   │       │ // hook non redéf.│            │
│    └───────────────────┘       └───────────────────┘            │
│                                                                 │
│   Les sous-classes redéfinissent les étapes, PAS l'algorithme   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Types de Méthodes

```
┌─────────────────────────────────────────────────────────────────┐
│               TYPES DE MÉTHODES DANS LE TEMPLATE                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. MÉTHODE TEMPLATE (finale)                                  │
│      ┌─────────────────────────────┐                            │
│      │ algorithme() {              │ Ne peut pas être           │
│      │     etape1();               │ redéfinie par les          │
│      │     etape2();               │ sous-classes               │
│      │     etape3();               │                            │
│      │ }                           │                            │
│      └─────────────────────────────┘                            │
│                                                                 │
│   2. MÉTHODES ABSTRAITES (obligatoires)                         │
│      ┌─────────────────────────────┐                            │
│      │ virtual etape2() = 0;       │ DOIVENT être               │
│      │ virtual etape3() = 0;       │ implémentées               │
│      └─────────────────────────────┘                            │
│                                                                 │
│   3. MÉTHODES CONCRÈTES (optionnelles)                          │
│      ┌─────────────────────────────┐                            │
│      │ etape1() { ... }            │ CAN être redéfinies        │
│      └─────────────────────────────┘ mais ont une impl. défaut  │
│                                                                 │
│   4. HOOKS (points d'extension)                                 │
│      ┌─────────────────────────────┐                            │
│      │ hook() { }  // vide         │ Optionnels, permettent     │
│      └─────────────────────────────┘ d'ajouter du comportement  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Classe Abstraite avec Template

```cpp
#include <iostream>
#include <string>
#include <fstream>

// Classe abstraite définissant le template
class AnalyseurDonnees {
public:
    // MÉTHODE TEMPLATE - définit l'algorithme
    // Le "final" empêche les sous-classes de la modifier
    void analyser() {
        ouvrirFichier();
        lireDonnees();
        traiterDonnees();    // Abstraite - DOIT être implémentée
        analyserResultats(); // Abstraite - DOIT être implémentée
        envoyerRapport();    // Hook - peut être redéfinie
        fermerFichier();
    }
    
    virtual ~AnalyseurDonnees() = default;
    
protected:
    std::string donnees;
    
    // Étapes concrètes (implémentation par défaut)
    virtual void ouvrirFichier() {
        std::cout << "📂 Ouverture du fichier de données..." << std::endl;
    }
    
    virtual void lireDonnees() {
        std::cout << "📖 Lecture des données brutes..." << std::endl;
    }
    
    virtual void fermerFichier() {
        std::cout << "📁 Fermeture du fichier." << std::endl;
    }
    
    // Étapes abstraites (DOIVENT être implémentées)
    virtual void traiterDonnees() = 0;
    virtual void analyserResultats() = 0;
    
    // Hook - comportement optionnel par défaut
    virtual void envoyerRapport() {
        // Par défaut: ne fait rien
    }
};
```

### Classes Concrètes

```cpp
// Analyseur CSV
class AnalyseurCSV : public AnalyseurDonnees {
protected:
    void lireDonnees() override {
        std::cout << "📖 Parsing du fichier CSV (séparateur: ,)..." << std::endl;
        donnees = "col1,col2,col3\n1,2,3\n4,5,6";
    }
    
    void traiterDonnees() override {
        std::cout << "⚙️  Traitement CSV: séparation par virgules..." << std::endl;
        std::cout << "   → 3 colonnes détectées" << std::endl;
        std::cout << "   → 2 lignes de données" << std::endl;
    }
    
    void analyserResultats() override {
        std::cout << "📊 Analyse CSV: calcul des statistiques par colonne..." << std::endl;
        std::cout << "   → Moyenne col1: 2.5" << std::endl;
    }
};

// Analyseur JSON
class AnalyseurJSON : public AnalyseurDonnees {
protected:
    void lireDonnees() override {
        std::cout << "📖 Parsing du fichier JSON..." << std::endl;
        donnees = R"({"items": [{"id": 1}, {"id": 2}]})";
    }
    
    void traiterDonnees() override {
        std::cout << "⚙️  Traitement JSON: parsing de l'arbre..." << std::endl;
        std::cout << "   → 1 objet racine détecté" << std::endl;
        std::cout << "   → 2 items dans le tableau" << std::endl;
    }
    
    void analyserResultats() override {
        std::cout << "📊 Analyse JSON: extraction des IDs..." << std::endl;
        std::cout << "   → IDs trouvés: [1, 2]" << std::endl;
    }
    
    // Redéfinition du hook
    void envoyerRapport() override {
        std::cout << "📧 Envoi du rapport JSON par email..." << std::endl;
    }
};

// Analyseur XML
class AnalyseurXML : public AnalyseurDonnees {
private:
    bool validationSchema = true;
    
protected:
    void lireDonnees() override {
        std::cout << "📖 Parsing du fichier XML avec validation XSD..." << std::endl;
        if (validationSchema) {
            std::cout << "   → Validation du schéma: OK" << std::endl;
        }
        donnees = "<root><item id='1'/><item id='2'/></root>";
    }
    
    void traiterDonnees() override {
        std::cout << "⚙️  Traitement XML: construction du DOM..." << std::endl;
        std::cout << "   → 1 élément racine" << std::endl;
        std::cout << "   → 2 éléments enfants" << std::endl;
    }
    
    void analyserResultats() override {
        std::cout << "📊 Analyse XML: extraction avec XPath..." << std::endl;
        std::cout << "   → Query: //item/@id" << std::endl;
        std::cout << "   → Résultats: ['1', '2']" << std::endl;
    }
    
    void envoyerRapport() override {
        std::cout << "💾 Sauvegarde du rapport XML en base de données..." << std::endl;
    }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║     PATRON DE MÉTHODE - DEMO          ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Analyseur CSV
    std::cout << "\n━━━ Analyse CSV ━━━" << std::endl;
    AnalyseurCSV csv;
    csv.analyser();
    
    // Analyseur JSON
    std::cout << "\n━━━ Analyse JSON ━━━" << std::endl;
    AnalyseurJSON json;
    json.analyser();
    
    // Analyseur XML
    std::cout << "\n━━━ Analyse XML ━━━" << std::endl;
    AnalyseurXML xml;
    xml.analyser();
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║     PATRON DE MÉTHODE - DEMO          ║
╚═══════════════════════════════════════╝

━━━ Analyse CSV ━━━
📂 Ouverture du fichier de données...
📖 Parsing du fichier CSV (séparateur: ,)...
⚙️  Traitement CSV: séparation par virgules...
   → 3 colonnes détectées
   → 2 lignes de données
📊 Analyse CSV: calcul des statistiques par colonne...
   → Moyenne col1: 2.5
📁 Fermeture du fichier.

━━━ Analyse JSON ━━━
📂 Ouverture du fichier de données...
📖 Parsing du fichier JSON...
⚙️  Traitement JSON: parsing de l'arbre...
   → 1 objet racine détecté
   → 2 items dans le tableau
📊 Analyse JSON: extraction des IDs...
   → IDs trouvés: [1, 2]
📧 Envoi du rapport JSON par email...
📁 Fermeture du fichier.

━━━ Analyse XML ━━━
📂 Ouverture du fichier de données...
📖 Parsing du fichier XML avec validation XSD...
   → Validation du schéma: OK
⚙️  Traitement XML: construction du DOM...
   → 1 élément racine
   → 2 éléments enfants
📊 Analyse XML: extraction avec XPath...
   → Query: //item/@id
   → Résultats: ['1', '2']
💾 Sauvegarde du rapport XML en base de données...
📁 Fermeture du fichier.
```

---

## 🎮 Autre Exemple: Jeu de Plateau

```cpp
// Template pour un tour de jeu
class JeuDePlateau {
public:
    // Méthode template
    void jouerPartie() {
        initialiserJeu();
        
        while (!estTermine()) {
            std::cout << "\n--- Tour du joueur " << joueurActuel() << " ---" << std::endl;
            jouerTour();
            changerJoueur();
        }
        
        afficherGagnant();
    }
    
    virtual ~JeuDePlateau() = default;
    
protected:
    // Méthodes abstraites
    virtual void initialiserJeu() = 0;
    virtual void jouerTour() = 0;
    virtual bool estTermine() = 0;
    virtual std::string joueurActuel() = 0;
    virtual void changerJoueur() = 0;
    virtual void afficherGagnant() = 0;
};

class Echecs : public JeuDePlateau {
    int tour = 0;
    std::string joueur = "Blanc";
    
protected:
    void initialiserJeu() override {
        std::cout << "♟️ Placement des pièces d'échecs" << std::endl;
    }
    
    void jouerTour() override {
        std::cout << "Joueur " << joueur << " déplace une pièce" << std::endl;
        tour++;
    }
    
    bool estTermine() override {
        return tour >= 4;  // Simplifié pour la démo
    }
    
    std::string joueurActuel() override { return joueur; }
    
    void changerJoueur() override {
        joueur = (joueur == "Blanc") ? "Noir" : "Blanc";
    }
    
    void afficherGagnant() override {
        std::cout << "🏆 Échec et mat! " << joueur << " gagne!" << std::endl;
    }
};
```

---

## 🪝 Utilisation des Hooks

```cpp
class ProcesseurCommande {
public:
    void traiterCommande() {
        valider();
        
        // Hook: permet de customiser
        if (avantTraitement()) {
            calculerTotal();
            appliquerRemises();
            
            // Hook: vérification optionnelle
            if (verifierStock()) {
                finaliser();
            }
        }
        
        apresTraitement();  // Hook de nettoyage
    }
    
protected:
    virtual void valider() = 0;
    virtual void calculerTotal() = 0;
    virtual void appliquerRemises() = 0;
    virtual void finaliser() = 0;
    
    // HOOKS avec comportement par défaut
    virtual bool avantTraitement() { return true; }  // Autorise par défaut
    virtual bool verifierStock() { return true; }    // OK par défaut
    virtual void apresTraitement() { }               // Rien par défaut
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Réutilisation** | Le code commun est dans la classe de base |
| **Contrôle** | La structure de l'algorithme est protégée |
| **Inversion de contrôle** | La classe de base appelle les sous-classes (Hollywood Principle) |
| **Extension ciblée** | Seules certaines parties sont modifiables |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Rigidité** | L'algorithme est fixe, difficile à changer |
| **Héritage** | Nécessite l'héritage (couplage fort) |
| **Complexité** | Beaucoup de méthodes à comprendre |
| **Debugging** | Le flux peut être difficile à suivre |

---

## 🎯 Cas d'Utilisation

1. **Frameworks** - Définir le flux d'exécution
2. **Parsers** - Lire/Traiter/Valider des données
3. **Tests** - Setup/Execute/Teardown
4. **Jeux** - Boucle de jeu standardisée
5. **Build systems** - Compile/Link/Package

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Stratégie](./Strategie.md) | Alternative: composition vs héritage |
| [Fabrique](../Creationnel/Fabrique.md) | Souvent utilisé avec Template Method |
| [Hook](./Observateur.md) | Les hooks sont similaires aux callbacks |

---

## 💡 Principe Hollywood

> **"Ne nous appelez pas, nous vous appellerons."**

La classe de base (Hollywood) appelle les méthodes des sous-classes (acteurs), pas l'inverse.

```
┌─────────────────────────────────────────────────────────────────┐
│                   INVERSION DE CONTRÔLE                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ClasseBase (Hollywood)          SousClasse (Acteur)           │
│         │                               │                       │
│         │   methodeTemplate()           │                       │
│         │──────────────────────────────►│                       │
│         │        etape1()               │                       │
│         │                               │                       │
│         │◄──────────────────────────────│                       │
│         │   appelle etapeAbstraite()    │                       │
│         │                               │                       │
│         │        implémente             │                       │
│         │──────────────────────────────►│                       │
│         │                               │                       │
│                                                                 │
│   Le framework (base) contrôle le flux, pas les sous-classes    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Stratégie](./Strategie.md)
