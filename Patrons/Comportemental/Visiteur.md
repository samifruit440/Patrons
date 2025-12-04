# 👤 Patron Visiteur (Visitor)

> **Patron Comportemental** - Permet de définir une nouvelle opération sans modifier les classes des éléments sur lesquels elle opère.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Représenter une **opération à effectuer** sur les éléments d'une structure d'objets. Le Visiteur permet de définir une **nouvelle opération** sans changer les classes des éléments.

---

## 🎯 Problème Résolu

- Comment ajouter des opérations à des classes sans les modifier?
- Comment éviter de "polluer" les classes avec des opérations non liées?
- Comment effectuer des opérations différentes selon le type concret?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON VISITEUR                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌────────────────────────┐     ┌────────────────────────┐     │
│   │     «interface»        │     │      «interface»       │     │
│   │       IElement         │     │       IVisiteur        │     │
│   ├────────────────────────┤     ├────────────────────────┤     │
│   │ +accepter(v: IVisiteur)│     │ +visiterA(e: ElementA) │     │
│   └───────────┬────────────┘     │ +visiterB(e: ElementB) │     │
│               │                  │ +visiterC(e: ElementC) │     │
│               │                  └───────────┬────────────┘     │
│    ┌──────────┼──────────┐                   │                  │
│    │          │          │        ┌──────────┴──────────┐       │
│    ▼          ▼          ▼        ▼                     ▼       │
│ ┌──────┐  ┌──────┐  ┌──────┐  ┌──────────┐      ┌──────────┐    │
│ │Elem A│  │Elem B│  │Elem C│  │Visiteur1 │      │Visiteur2 │    │
│ ├──────┤  ├──────┤  ├──────┤  ├──────────┤      ├──────────┤    │
│ │accept│  │accept│  │accept│  │visiterA()│      │visiterA()│    │
│ │(v){  │  │(v){  │  │(v){  │  │visiterB()│      │visiterB()│    │
│ │ v.   │  │ v.   │  │ v.   │  │visiterC()│      │visiterC()│    │
│ │visit │  │visit │  │visit │  └──────────┘      └──────────┘    │
│ │erA   │  │erB   │  │erC   │                                    │
│ │(this)│  │(this)│  │(this)│                                    │
│ │}     │  │}     │  │}     │                                    │
│ └──────┘  └──────┘  └──────┘                                    │
│                                                                 │
│   Double dispatch: l'élément appelle la bonne méthode           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Double Dispatch

```
┌─────────────────────────────────────────────────────────────────┐
│                    MÉCANISME DOUBLE DISPATCH                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client          Element           Visiteur                    │
│     │                │                  │                       │
│     │ accepter(v)    │                  │                       │
│     │───────────────►│                  │                       │
│     │                │                  │                       │
│     │                │  visiterX(this)  │                       │
│     │                │─────────────────►│                       │
│     │                │                  │                       │
│     │                │   accès aux      │                       │
│     │                │◄─ données de X ──│                       │
│     │                │                  │                       │
│                                                                 │
│   1er dispatch: accepter() → type de l'élément                  │
│   2e dispatch: visiterX() → type du visiteur                    │
│                                                                 │
│   Résultat: comportement dépend des DEUX types                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Visiteur

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <memory>
#include <sstream>
#include <iomanip>

// Forward declarations
class Cercle;
class Rectangle;
class Triangle;

// Interface Visiteur
class IVisiteurForme {
public:
    virtual void visiterCercle(Cercle* c) = 0;
    virtual void visiterRectangle(Rectangle* r) = 0;
    virtual void visiterTriangle(Triangle* t) = 0;
    virtual ~IVisiteurForme() = default;
};
```

### Interface Élément

```cpp
// Interface Élément
class IForme {
public:
    virtual void accepter(IVisiteurForme* visiteur) = 0;
    virtual std::string getNom() const = 0;
    virtual ~IForme() = default;
};
```

### Éléments Concrets

```cpp
// Élément Concret: Cercle
class Cercle : public IForme {
private:
    double rayon;
    
public:
    Cercle(double r) : rayon(r) {}
    
    double getRayon() const { return rayon; }
    
    void accepter(IVisiteurForme* visiteur) override {
        visiteur->visiterCercle(this);  // Double dispatch!
    }
    
    std::string getNom() const override { return "Cercle"; }
};

// Élément Concret: Rectangle
class Rectangle : public IForme {
private:
    double largeur, hauteur;
    
public:
    Rectangle(double l, double h) : largeur(l), hauteur(h) {}
    
    double getLargeur() const { return largeur; }
    double getHauteur() const { return hauteur; }
    
    void accepter(IVisiteurForme* visiteur) override {
        visiteur->visiterRectangle(this);
    }
    
    std::string getNom() const override { return "Rectangle"; }
};

// Élément Concret: Triangle
class Triangle : public IForme {
private:
    double base, hauteur;
    
public:
    Triangle(double b, double h) : base(b), hauteur(h) {}
    
    double getBase() const { return base; }
    double getHauteur() const { return hauteur; }
    
    void accepter(IVisiteurForme* visiteur) override {
        visiteur->visiterTriangle(this);
    }
    
    std::string getNom() const override { return "Triangle"; }
};
```

### Visiteurs Concrets

```cpp
// Visiteur 1: Calcul d'aire
class VisiteurAire : public IVisiteurForme {
private:
    double aireTotal = 0;
    
public:
    void visiterCercle(Cercle* c) override {
        double aire = 3.14159 * c->getRayon() * c->getRayon();
        std::cout << "   📐 Aire cercle (r=" << c->getRayon() 
                  << "): " << std::fixed << std::setprecision(2) 
                  << aire << std::endl;
        aireTotal += aire;
    }
    
    void visiterRectangle(Rectangle* r) override {
        double aire = r->getLargeur() * r->getHauteur();
        std::cout << "   📐 Aire rectangle (" << r->getLargeur() 
                  << "x" << r->getHauteur() << "): " << aire << std::endl;
        aireTotal += aire;
    }
    
    void visiterTriangle(Triangle* t) override {
        double aire = (t->getBase() * t->getHauteur()) / 2.0;
        std::cout << "   📐 Aire triangle (b=" << t->getBase() 
                  << ", h=" << t->getHauteur() << "): " << aire << std::endl;
        aireTotal += aire;
    }
    
    double getAireTotal() const { return aireTotal; }
    void reset() { aireTotal = 0; }
};

// Visiteur 2: Calcul de périmètre
class VisiteurPerimetre : public IVisiteurForme {
private:
    double perimetreTotal = 0;
    
public:
    void visiterCercle(Cercle* c) override {
        double perimetre = 2 * 3.14159 * c->getRayon();
        std::cout << "   📏 Périmètre cercle: " << std::fixed 
                  << std::setprecision(2) << perimetre << std::endl;
        perimetreTotal += perimetre;
    }
    
    void visiterRectangle(Rectangle* r) override {
        double perimetre = 2 * (r->getLargeur() + r->getHauteur());
        std::cout << "   📏 Périmètre rectangle: " << perimetre << std::endl;
        perimetreTotal += perimetre;
    }
    
    void visiterTriangle(Triangle* t) override {
        // Simplifié: triangle rectangle
        double hypotenuse = std::sqrt(t->getBase() * t->getBase() + 
                                       t->getHauteur() * t->getHauteur());
        double perimetre = t->getBase() + t->getHauteur() + hypotenuse;
        std::cout << "   📏 Périmètre triangle: " << std::fixed 
                  << std::setprecision(2) << perimetre << std::endl;
        perimetreTotal += perimetre;
    }
    
    double getPerimetreTotal() const { return perimetreTotal; }
};

// Visiteur 3: Export XML
class VisiteurExportXML : public IVisiteurForme {
private:
    std::stringstream xml;
    
public:
    VisiteurExportXML() {
        xml << "<?xml version=\"1.0\"?>\n<formes>\n";
    }
    
    void visiterCercle(Cercle* c) override {
        xml << "  <cercle rayon=\"" << c->getRayon() << "\"/>\n";
    }
    
    void visiterRectangle(Rectangle* r) override {
        xml << "  <rectangle largeur=\"" << r->getLargeur() 
            << "\" hauteur=\"" << r->getHauteur() << "\"/>\n";
    }
    
    void visiterTriangle(Triangle* t) override {
        xml << "  <triangle base=\"" << t->getBase() 
            << "\" hauteur=\"" << t->getHauteur() << "\"/>\n";
    }
    
    std::string getXML() {
        return xml.str() + "</formes>";
    }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔════════════════════════════════════════╗" << std::endl;
    std::cout << "║       PATRON VISITEUR - DEMO           ║" << std::endl;
    std::cout << "╚════════════════════════════════════════╝" << std::endl;
    
    // Créer les formes
    std::vector<std::shared_ptr<IForme>> formes;
    formes.push_back(std::make_shared<Cercle>(5.0));
    formes.push_back(std::make_shared<Rectangle>(4.0, 6.0));
    formes.push_back(std::make_shared<Triangle>(3.0, 4.0));
    formes.push_back(std::make_shared<Cercle>(2.5));
    
    std::cout << "\nFormes créées: " << formes.size() << std::endl;
    
    // Visiteur 1: Calculer les aires
    std::cout << "\n=== Calcul des Aires ===" << std::endl;
    VisiteurAire visiteurAire;
    for (auto& forme : formes) {
        forme->accepter(&visiteurAire);
    }
    std::cout << "   ═══════════════════════" << std::endl;
    std::cout << "   📊 AIRE TOTALE: " << visiteurAire.getAireTotal() << std::endl;
    
    // Visiteur 2: Calculer les périmètres
    std::cout << "\n=== Calcul des Périmètres ===" << std::endl;
    VisiteurPerimetre visiteurPerimetre;
    for (auto& forme : formes) {
        forme->accepter(&visiteurPerimetre);
    }
    std::cout << "   ═══════════════════════" << std::endl;
    std::cout << "   📊 PÉRIMÈTRE TOTAL: " 
              << visiteurPerimetre.getPerimetreTotal() << std::endl;
    
    // Visiteur 3: Export XML
    std::cout << "\n=== Export XML ===" << std::endl;
    VisiteurExportXML visiteurXML;
    for (auto& forme : formes) {
        forme->accepter(&visiteurXML);
    }
    std::cout << visiteurXML.getXML() << std::endl;
    
    return 0;
}
```

### Sortie

```
╔════════════════════════════════════════╗
║       PATRON VISITEUR - DEMO           ║
╚════════════════════════════════════════╝

Formes créées: 4

=== Calcul des Aires ===
   📐 Aire cercle (r=5): 78.54
   📐 Aire rectangle (4x6): 24.00
   📐 Aire triangle (b=3, h=4): 6.00
   📐 Aire cercle (r=2.5): 19.63
   ═══════════════════════
   📊 AIRE TOTALE: 128.17

=== Calcul des Périmètres ===
   📏 Périmètre cercle: 31.42
   📏 Périmètre rectangle: 20.00
   📏 Périmètre triangle: 12.00
   📏 Périmètre cercle: 15.71
   ═══════════════════════
   📊 PÉRIMÈTRE TOTAL: 79.13

=== Export XML ===
<?xml version="1.0"?>
<formes>
  <cercle rayon="5"/>
  <rectangle largeur="4" hauteur="6"/>
  <triangle base="3" hauteur="4"/>
  <cercle rayon="2.5"/>
</formes>
```

---

## 🆚 Avec vs Sans Visiteur

```
┌─────────────────────────────────────────────────────────────────┐
│                      COMPARAISON                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ❌ SANS VISITEUR (opérations dans les classes):               │
│   ┌─────────────────────────────────────┐                       │
│   │ class Cercle {                      │                       │
│   │   double aire() { ... }             │                       │
│   │   double perimetre() { ... }        │                       │
│   │   string toXML() { ... }            │ ← Pollue la classe    │
│   │   string toJSON() { ... }           │   avec des opérations │
│   │   void dessiner() { ... }           │   non essentielles    │
│   │   // Nouvelle op? Modifier classe!  │                       │
│   │ }                                   │                       │
│   └─────────────────────────────────────┘                       │
│                                                                 │
│   ✅ AVEC VISITEUR:                                             │
│   ┌─────────────────────────────────────┐                       │
│   │ class Cercle {                      │                       │
│   │   void accepter(IVisiteur* v) {     │ ← Une seule méthode   │
│   │     v->visiterCercle(this);         │                       │
│   │   }                                 │                       │
│   │ }                                   │                       │
│   │                                     │                       │
│   │ // Nouvelle opération = nouveau visiteur                    │
│   │ class VisiteurJSON : public IVisiteur { ... }               │
│   └─────────────────────────────────────┘                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🌳 Visiteur avec Composite

```cpp
// Le Visiteur s'intègre bien avec le Composite
class GroupeFormes : public IForme {
private:
    std::vector<std::shared_ptr<IForme>> enfants;
    
public:
    void ajouter(std::shared_ptr<IForme> f) { 
        enfants.push_back(f); 
    }
    
    void accepter(IVisiteurForme* visiteur) override {
        // Propager le visiteur à tous les enfants
        for (auto& enfant : enfants) {
            enfant->accepter(visiteur);
        }
    }
    
    std::string getNom() const override { return "Groupe"; }
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Séparation** | Opérations séparées des classes |
| **Extensibilité** | Nouvelle opération = nouveau visiteur |
| **Accumulation** | Peut accumuler des résultats |
| **Double dispatch** | Comportement selon deux types |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Nouvel élément** | Ajouter un élément = modifier tous les visiteurs |
| **Encapsulation** | Le visiteur accède aux détails internes |
| **Complexité** | Structure peut sembler complexe |

---

## 🎯 Cas d'Utilisation

1. **Compilateurs** - AST traversal (analyse, optimisation, génération)
2. **Graphiques** - Calculs sur formes (aire, périmètre, rendu)
3. **Export** - Sérialisation en différents formats (XML, JSON, HTML)
4. **Rapports** - Génération de statistiques sur structures
5. **Validation** - Vérification de règles métier

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Composite](../Structurel/Composite.md) | Visiteur parcourt souvent un composite |
| [Itérateur](./Iterateur.md) | Alternative pour parcourir |
| [Interpréteur](./Interpreteur.md) | Utilise souvent le visiteur |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Médiateur](./Mediateur.md)
