# 🔌 Patron Adaptateur (Adapter)

> **Patron Structurel** - Convertit l'interface d'une classe en une autre interface attendue par les clients.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Convertir l'**interface d'une classe** en une autre interface que les clients attendent. L'Adaptateur permet à des classes incompatibles de collaborer.

---

## 🎯 Problème Résolu

- Comment utiliser une classe existante dont l'interface ne correspond pas?
- Comment intégrer du code tiers ou legacy?
- Comment faire collaborer des classes aux interfaces incompatibles?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON ADAPTATEUR                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ADAPTATEUR OBJET (composition)                                │
│                                                                 │
│   ┌──────────────┐      ┌──────────────────┐                    │
│   │    Client    │      │   «interface»    │                    │
│   └──────┬───────┘      │      Cible       │                    │
│          │ utilise      ├──────────────────┤                    │
│          └─────────────►│ +requete()       │                    │
│                         └────────┬─────────┘                    │
│                                  │                              │
│                         ┌────────▼─────────┐                    │
│                         │    Adaptateur    │                    │
│                         ├──────────────────┤                    │
│                         │ -adapte: Adapte  │◆───────┐          │
│                         ├──────────────────┤         │          │
│                         │ +requete() {     │         │          │
│                         │   adapte.        │         │          │
│                         │   requeteSpec()  │         │          │
│                         │ }                │         │          │
│                         └──────────────────┘         │          │
│                                                      │          │
│                                           ┌──────────▼────────┐ │
│                                           │      Adapté       │ │
│                                           ├───────────────────┤ │
│                                           │ +requeteSpec()    │ │
│                                           └───────────────────┘ │
│                                                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ADAPTATEUR CLASSE (héritage multiple)                         │
│                                                                 │
│   ┌──────────────────┐           ┌───────────────────┐          │
│   │      Cible       │           │      Adapté       │          │
│   ├──────────────────┤           ├───────────────────┤          │
│   │ +requete()       │           │ +requeteSpec()    │          │
│   └────────┬─────────┘           └─────────┬─────────┘          │
│            │                               │                    │
│            └───────────┬───────────────────┘                    │
│                        │ héritage privé                         │
│               ┌────────▼─────────┐                              │
│               │    Adaptateur    │                              │
│               ├──────────────────┤                              │
│               │ +requete() {     │                              │
│               │   requeteSpec()  │                              │
│               │ }                │                              │
│               └──────────────────┘                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔌 Analogie du Monde Réel

```
┌─────────────────────────────────────────────────────────────────┐
│                   ANALOGIE: PRISE ÉLECTRIQUE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Problème: Appareil français → Prise américaine                │
│                                                                 │
│   ┌─────────────┐    ┌─────────────────┐    ┌─────────────┐     │
│   │  Appareil   │    │   Adaptateur    │    │   Prise     │     │
│   │  français   │───►│   FR → US       │───►│ américaine  │     │
│   │  (2 broches)│    │                 │    │ (3 broches) │     │
│   └─────────────┘    └─────────────────┘    └─────────────┘     │
│                                                                 │
│   L'adaptateur convertit l'interface incompatible               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Attendue (Cible)

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <memory>
#include <cmath>

// Interface attendue par notre application
class IFormeGeometrique {
public:
    virtual void dessiner() const = 0;
    virtual double getAire() const = 0;
    virtual double getPerimetre() const = 0;
    virtual std::string getNom() const = 0;
    virtual ~IFormeGeometrique() = default;
};
```

### Classes Legacy (Adaptées)

```cpp
// ═══════════════════════════════════════
// CLASSES LEGACY (code existant à adapter)
// ═══════════════════════════════════════

// Bibliothèque legacy avec une API différente
namespace LegacyGraphics {
    
    class LegacyRectangle {
    private:
        double x1, y1, x2, y2;  // Coins opposés
        
    public:
        LegacyRectangle(double x1, double y1, double x2, double y2)
            : x1(x1), y1(y1), x2(x2), y2(y2) {}
        
        void oldDraw() const {
            std::cout << "LegacyRect: drawing from (" << x1 << "," << y1 
                      << ") to (" << x2 << "," << y2 << ")" << std::endl;
        }
        
        double calculateArea() const {
            return std::abs((x2 - x1) * (y2 - y1));
        }
        
        double getWidth() const { return std::abs(x2 - x1); }
        double getHeight() const { return std::abs(y2 - y1); }
    };
    
    class LegacyCircle {
    private:
        double centerX, centerY, radius;
        
    public:
        LegacyCircle(double cx, double cy, double r)
            : centerX(cx), centerY(cy), radius(r) {}
        
        void render() const {
            std::cout << "LegacyCircle: rendering at (" << centerX << "," 
                      << centerY << ") with radius " << radius << std::endl;
        }
        
        double computeArea() const {
            return 3.14159 * radius * radius;
        }
        
        double computeCircumference() const {
            return 2 * 3.14159 * radius;
        }
        
        double getRadius() const { return radius; }
    };
}

// API tierce avec encore une autre interface
namespace ThirdPartyShapes {
    
    struct Triangle {
        double sideA, sideB, sideC;
        
        Triangle(double a, double b, double c) : sideA(a), sideB(b), sideC(c) {}
        
        void display() const {
            std::cout << "Triangle: sides = " << sideA << ", " 
                      << sideB << ", " << sideC << std::endl;
        }
        
        double areaByHeron() const {
            double s = (sideA + sideB + sideC) / 2;
            return std::sqrt(s * (s - sideA) * (s - sideB) * (s - sideC));
        }
    };
}
```

### Adaptateurs

```cpp
// ═══════════════════════════════════════
// ADAPTATEURS
// ═══════════════════════════════════════

// Adaptateur pour LegacyRectangle
class RectangleAdapter : public IFormeGeometrique {
private:
    std::unique_ptr<LegacyGraphics::LegacyRectangle> rectangle;
    
public:
    RectangleAdapter(double x1, double y1, double x2, double y2)
        : rectangle(std::make_unique<LegacyGraphics::LegacyRectangle>(x1, y1, x2, y2)) {}
    
    void dessiner() const override {
        std::cout << "🔷 ";
        rectangle->oldDraw();  // Délègue à l'ancienne méthode
    }
    
    double getAire() const override {
        return rectangle->calculateArea();
    }
    
    double getPerimetre() const override {
        return 2 * (rectangle->getWidth() + rectangle->getHeight());
    }
    
    std::string getNom() const override {
        return "Rectangle (adapté)";
    }
};

// Adaptateur pour LegacyCircle
class CercleAdapter : public IFormeGeometrique {
private:
    std::unique_ptr<LegacyGraphics::LegacyCircle> cercle;
    
public:
    CercleAdapter(double cx, double cy, double r)
        : cercle(std::make_unique<LegacyGraphics::LegacyCircle>(cx, cy, r)) {}
    
    void dessiner() const override {
        std::cout << "⭕ ";
        cercle->render();
    }
    
    double getAire() const override {
        return cercle->computeArea();
    }
    
    double getPerimetre() const override {
        return cercle->computeCircumference();
    }
    
    std::string getNom() const override {
        return "Cercle (adapté)";
    }
};

// Adaptateur pour Triangle tiers
class TriangleAdapter : public IFormeGeometrique {
private:
    std::unique_ptr<ThirdPartyShapes::Triangle> triangle;
    
public:
    TriangleAdapter(double a, double b, double c)
        : triangle(std::make_unique<ThirdPartyShapes::Triangle>(a, b, c)) {}
    
    void dessiner() const override {
        std::cout << "🔺 ";
        triangle->display();
    }
    
    double getAire() const override {
        return triangle->areaByHeron();
    }
    
    double getPerimetre() const override {
        return triangle->sideA + triangle->sideB + triangle->sideC;
    }
    
    std::string getNom() const override {
        return "Triangle (adapté)";
    }
};
```

### Utilisation

```cpp
// Client qui travaille avec l'interface unifiée
class DessinateurFormes {
public:
    void dessinerEtCalculer(const std::vector<std::shared_ptr<IFormeGeometrique>>& formes) {
        std::cout << "\n=== Dessin et Calculs ===" << std::endl;
        
        double aireTotal = 0;
        double perimetreTotal = 0;
        
        for (const auto& forme : formes) {
            forme->dessiner();
            
            double aire = forme->getAire();
            double perimetre = forme->getPerimetre();
            
            std::cout << "   → Aire: " << aire 
                      << ", Périmètre: " << perimetre << std::endl;
            
            aireTotal += aire;
            perimetreTotal += perimetre;
        }
        
        std::cout << "\n📊 Totaux:" << std::endl;
        std::cout << "   Aire totale: " << aireTotal << std::endl;
        std::cout << "   Périmètre total: " << perimetreTotal << std::endl;
    }
};

int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON ADAPTATEUR - DEMO         ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Créer des formes via les adaptateurs
    std::vector<std::shared_ptr<IFormeGeometrique>> formes;
    
    // Rectangle legacy adapté
    formes.push_back(std::make_shared<RectangleAdapter>(0, 0, 10, 5));
    
    // Cercle legacy adapté
    formes.push_back(std::make_shared<CercleAdapter>(0, 0, 7));
    
    // Triangle tiers adapté
    formes.push_back(std::make_shared<TriangleAdapter>(3, 4, 5));
    
    // Deuxième rectangle
    formes.push_back(std::make_shared<RectangleAdapter>(0, 0, 3, 3));
    
    // Le client utilise l'interface unifiée
    DessinateurFormes dessinateur;
    dessinateur.dessinerEtCalculer(formes);
    
    // Liste des formes
    std::cout << "\n=== Formes disponibles ===" << std::endl;
    for (const auto& forme : formes) {
        std::cout << "• " << forme->getNom() << std::endl;
    }
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON ADAPTATEUR - DEMO         ║
╚═══════════════════════════════════════╝

=== Dessin et Calculs ===
🔷 LegacyRect: drawing from (0,0) to (10,5)
   → Aire: 50, Périmètre: 30
⭕ LegacyCircle: rendering at (0,0) with radius 7
   → Aire: 153.938, Périmètre: 43.9823
🔺 Triangle: sides = 3, 4, 5
   → Aire: 6, Périmètre: 12
🔷 LegacyRect: drawing from (0,0) to (3,3)
   → Aire: 9, Périmètre: 12

📊 Totaux:
   Aire totale: 218.938
   Périmètre total: 97.9823

=== Formes disponibles ===
• Rectangle (adapté)
• Cercle (adapté)
• Triangle (adapté)
• Rectangle (adapté)
```

---

## 🔧 Adaptateur Bidirectionnel

```cpp
// Adapter qui fonctionne dans les deux sens
class BidirectionalAdapter : public IFormeGeometrique, 
                             public LegacyGraphics::LegacyRectangle {
public:
    BidirectionalAdapter(double x1, double y1, double x2, double y2)
        : LegacyRectangle(x1, y1, x2, y2) {}
    
    // Interface moderne
    void dessiner() const override { oldDraw(); }
    double getAire() const override { return calculateArea(); }
    // ...
    
    // Peut aussi être utilisé comme LegacyRectangle
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Réutilisation** | Utiliser du code existant |
| **Séparation** | Client découplé des classes legacy |
| **Single Responsibility** | Conversion séparée de la logique |
| **Open/Closed** | Nouveaux adaptateurs sans modifier l'existant |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Complexité** | Couche supplémentaire de code |
| **Performance** | Indirection supplémentaire |
| **Trop d'adaptateurs** | Peut devenir difficile à gérer |

---

## 🎯 Cas d'Utilisation

1. **Intégration legacy** - Adapter du vieux code
2. **Bibliothèques tierces** - Wrapper pour API externe
3. **Tests** - Adapter des mocks
4. **Formats de données** - XML ↔ JSON

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Décorateur](./Decorateur.md) | Modifie comportement vs interface |
| [Proxy](./Proxy.md) | Même interface vs interface différente |
| [Façade](./Facade.md) | Simplifie une interface complexe |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Façade](./Facade.md)
