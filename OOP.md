# 🏗️ Concepts OOP en C++

> Guide complet des concepts orientés objet en C++ : héritage, composition, polymorphisme et bonnes pratiques

[⬅️ Retour à l'Index](./INDEX.md)

---

## 📋 Table des Matières

1. [Héritage Public](#-héritage-public)
2. [Héritage Privé](#-héritage-privé)
3. [Héritage Protégé](#-héritage-protégé)
4. [Composition vs Héritage](#-composition-vs-héritage)
5. [Méthodes Virtuelles](#-méthodes-virtuelles)
6. [Héritage Multiple](#-héritage-multiple)
7. [Bonnes Pratiques](#-bonnes-pratiques)

---

## 🔓 Héritage Public

L'héritage **public** est la forme la plus courante. La classe dérivée expose l'interface publique de la classe de base.

### Caractéristiques

- **Members publics** de la base → publics dans la dérivée ✅
- **Members protégés** de la base → protégés dans la dérivée ✅
- **Members privés** de la base → inaccessibles dans la dérivée ❌
- **Relation** : "est un(e)" (IS-A)

### Exemple

```cpp
#include <iostream>
using namespace std;

// Classe de base
class Animal {
public:
    void manger() {
        cout << "L'animal mange" << endl;
    }
    
protected:
    string nom;
    
private:
    int energie;
};

// Héritage PUBLIC
class Chien : public Animal {
public:
    void aboyer() {
        cout << nom << " aboie" << endl;  // ✅ nom est protected, accessible
        // cout << energie << endl;        // ❌ energie est privé, inaccessible
    }
};

int main() {
    Chien rex;
    rex.manger();   // ✅ Hérité et public
    rex.aboyer();   // ✅ Propre à Chien
    // rex.nom;     // ❌ Protected, inaccessible de l'extérieur
    
    return 0;
}
```

### Cas d'Usage

- **Interfaces et contrats** : La classe de base définit l'interface publique
- **Hiérarchies de types** : Animal → Chien, Chat, Oiseau
- **Polymorphisme** : Utilisation via pointeurs/références de la classe de base

---

## 🔒 Héritage Privé

L'héritage **privé** masque entièrement l'interface publique de la classe de base dans la dérivée.

### Caractéristiques

- **Tous les members** de la base → privés dans la dérivée
- **Relation** : "implémenté en termes de" (IMPLEMENTED-IN-TERMS-OF)
- **Usage** : Rare, surtout pour composition de traits privés

### Exemple

```cpp
#include <iostream>
using namespace std;

class Moteur {
public:
    void demarrer() {
        cout << "Moteur démarré" << endl;
    }
};

// Héritage PRIVÉ
class Voiture : private Moteur {
public:
    void conduire() {
        demarrer();  // ✅ Accessible internement
        cout << "Voiture en route" << endl;
    }
};

int main() {
    Voiture v;
    v.conduire();        // ✅
    // v.demarrer();     // ❌ Hérité mais privé, inaccessible!
    
    return 0;
}
```

### Cas d'Usage

- **Composition déguisée** : Quand on veut réutiliser l'implémentation
- **Traits privés** : Ajouter du comportement sans exposer l'interface
- ⚠️ **Rare en pratique** : Préférer la composition

---

## 🛡️ Héritage Protégé

L'héritage **protégé** expose l'interface publique comme protected pour les sous-classes.

### Caractéristiques

- **Members publics** de la base → protégés dans la dérivée
- **Members protégés** de la base → protégés dans la dérivée
- **Usage** : Très rare, cas intermédiaire peu utile

### Exemple

```cpp
class Parent {
public:
    void methodePublique() {}
};

// Héritage PROTÉGÉ
class Enfant : protected Parent {
public:
    void utiliser() {
        methodePublique();  // ✅ Accessible
    }
};

class PetitEnfant : public Enfant {
    // methodePublique() n'est pas accessible ici
    // (elle est protected dans Enfant)
};
```

---

## 🧩 Composition vs Héritage

**Composition** (HAS-A) et **Héritage** (IS-A) sont deux approches différentes.

### Comparaison

```cpp
// ❌ MAUVAIS: Héritage pour la composition
class Batterie {
public:
    void charger() {}
};

class Telephone : public Batterie {  // ❌ Téléphone n'EST PAS une batterie!
};

// ✅ BON: Composition
class Telephone {
private:
    Batterie batterie;  // Le téléphone HAS-A batterie
public:
    void charger() {
        batterie.charger();
    }
};
```

### Quand utiliser quoi?

| Situation | Héritage | Composition |
|-----------|----------|-------------|
| "EST UN" (animal → chien) | ✅ | ❌ |
| "A UN" (voiture → moteur) | ❌ | ✅ |
| Partager l'interface | ✅ | ❌ |
| Partager l'implémentation | ⚠️ | ✅ |
| Flexibilité à l'exécution | ❌ | ✅ |
| Liaisons statiques | ✅ | ⚠️ |

### Exemple Complet

```cpp
#include <iostream>
#include <memory>
using namespace std;

// Approche: Composition + Héritage
class Moteur {
public:
    virtual void demarrer() = 0;
    virtual ~Moteur() = default;
};

class MoteurEssence : public Moteur {
public:
    void demarrer() override {
        cout << "Moteur essence: vroooom!" << endl;
    }
};

class MoteurDiesel : public Moteur {
public:
    void demarrer() override {
        cout << "Moteur diesel: tatatata" << endl;
    }
};

// Voiture compose un moteur (HAS-A)
class Voiture {
private:
    unique_ptr<Moteur> moteur;  // Composition
    
public:
    Voiture(unique_ptr<Moteur> m) : moteur(move(m)) {}
    
    void conduire() {
        moteur->demarrer();
        cout << "En route!" << endl;
    }
};

int main() {
    // Flexible: changer le moteur à l'exécution
    Voiture v1(make_unique<MoteurEssence>());
    Voiture v2(make_unique<MoteurDiesel>());
    
    v1.conduire();  // Moteur essence
    v2.conduire();  // Moteur diesel
    
    return 0;
}
```

---

## 📌 Méthodes Virtuelles

Les **méthodes virtuelles** permettent le **polymorphisme dynamique** (late binding).

### Caractéristiques

- `virtual` dans la classe de base
- `override` (C++11) dans la classe dérivée
- Décision de quelle méthode appeler à l'exécution
- Nécessite des pointeurs/références

### Exemple

```cpp
#include <iostream>
using namespace std;

class Animal {
public:
    virtual void faire_bruit() {
        cout << "Son générique" << endl;
    }
    virtual ~Animal() = default;  // ✅ Destructeur virtuel!
};

class Chien : public Animal {
public:
    void faire_bruit() override {  // override = vérification du compilateur
        cout << "Woof!" << endl;
    }
};

class Chat : public Animal {
public:
    void faire_bruit() override {
        cout << "Meow!" << endl;
    }
};

int main() {
    // Polymorphisme !
    Animal* animaux[] = {
        new Chien(),
        new Chat(),
        new Animal()
    };
    
    for (Animal* a : animaux) {
        a->faire_bruit();  // Appel la bonne méthode!
    }
    
    // Cleanup
    for (Animal* a : animaux) {
        delete a;  // Appel le bon destructeur grâce à virtual
    }
    
    return 0;
}
```

### Output

```
Woof!
Meow!
Son générique
```

### Points Clés

- ✅ **Toujours** virtuel le destructeur si héritage public
- ✅ **override** pour vérification du compilateur
- ✅ Utiliser pointeurs/références, pas objets
- ❌ Ne pas virtualiser si pas nécessaire (overhead de performance)

---

## 🔗 Héritage Multiple

L'**héritage multiple** permet d'hériter de plusieurs classes (dangereux!).

### Le Problème Classique: Diamant

```cpp
    ┌─────────┐
    │  Être   │
    └────┬────┘
         │
    ┌────┴────┐
    │         │
 ┌──▼──┐   ┌──▼──┐
 │Homme│   │Femme│
 └──┬──┘   └──┬──┘
    │         │
    └────┬────┘
         │
      ┌──▼───┐
      │Parent│  ← Hérité de Être deux fois!
      └──────┘
```

### Problème

```cpp
class Etre {
public:
    void respirer() { }
};

class Homme : public Etre { };
class Femme : public Etre { };

class Parent : public Homme, public Femme { };

int main() {
    Parent p;
    p.respirer();  // ❌ ERREUR: Quelle respirer()? Homme ou Femme?
    return 0;
}
```

### Solution: Héritage Virtuel

```cpp
class Etre {
public:
    void respirer() { }
};

// Héritage virtuel
class Homme : virtual public Etre { };
class Femme : virtual public Etre { };

class Parent : public Homme, public Femme { };

int main() {
    Parent p;
    p.respirer();  // ✅ OK! Une seule copie de Etre
    return 0;
}
```

### Quand l'utiliser?

- ⚠️ **Très rare** en pratique
- **Mixins/Traits** : interfaces multiples
- **Composition préférable** : plus simple et clair

### Exemple: Interfaces Multiples

```cpp
#include <iostream>
using namespace std;

class Drawable {
public:
    virtual void draw() = 0;
    virtual ~Drawable() = default;
};

class Movable {
public:
    virtual void move() = 0;
    virtual ~Movable() = default;
};

// Un sprite: À LA FOIS dessinable ET mobile
class Sprite : public Drawable, public Movable {
public:
    void draw() override {
        cout << "Dessin du sprite" << endl;
    }
    
    void move() override {
        cout << "Déplacement du sprite" << endl;
    }
};

int main() {
    Sprite s;
    Drawable* d = &s;
    Movable* m = &s;
    
    d->draw();  // ✅
    m->move();  // ✅
    
    return 0;
}
```

---

## ✅ Bonnes Pratiques

### 1️⃣ Préférer Composition à Héritage

```cpp
// ❌ MAUVAIS
class Voiture : public Moteur, public Chassis, public Transmission { }

// ✅ BON
class Voiture {
    Moteur moteur;
    Chassis chassis;
    Transmission transmission;
};
```

### 2️⃣ Destructeur Virtuel en Cas d'Héritage Public

```cpp
// ❌ MAUVAIS
class Base {
public:
    ~Base() { }  // Non-virtuel!
};

// ✅ BON
class Base {
public:
    virtual ~Base() { }  // Virtuel!
};
```

### 3️⃣ Utiliser override et final

```cpp
// ❌ MAUVAIS
class Base {
public:
    virtual void method() { }
};

class Derived : public Base {
public:
    void method() { }  // Typo? C'est pas une redéfinition!
};

// ✅ BON
class Derived : public Base {
public:
    void method() override { }  // Erreur si Base n'a pas cette méthode
};

// Empêcher la redéfinition
class Final : public Base {
public:
    void method() override final { }  // Plus personne ne peut redéfinir
};
```

### 4️⃣ Interface vs Implémentation

```cpp
// ✅ Interface pure (abstract)
class Animal {
public:
    virtual void faire_bruit() = 0;  // Obligation pour les dérivées
    virtual ~Animal() = default;
};

// ✅ Implémentation de base
class Animal {
public:
    virtual void faire_bruit() {
        cout << "Son par défaut" << endl;
    }
    virtual ~Animal() = default;
};
```

### 5️⃣ Règle des 5 (ou 0)

```cpp
// ✅ Ou utiliser = default, ou tout implémenter correctement
class MyClass {
public:
    MyClass();                              // Constructeur
    MyClass(const MyClass&);                // Constructeur de copie
    MyClass& operator=(const MyClass&);     // Opérateur d'affectation
    ~MyClass();                             // Destructeur
    MyClass(MyClass&&) noexcept;            // Constructeur de déplacement
    MyClass& operator=(MyClass&&) noexcept; // Opérateur de déplacement
};
```

### 6️⃣ Éviter l'Héritage Multiple (sauf interfaces)

```cpp
// ❌ MAUVAIS: Implémentation multiple
class Worker : public Employee, public ContractorBase { }

// ✅ BON: Interfaces multiples
class Worker : public IEmployee, public IContractor { }

// ✅ BON: Composition pour l'implémentation
class Worker {
    Employee employee;
    ContractorLogic contractor;
};
```

### 7️⃣ Respecter le Contrat de Liskov

```cpp
// ❌ MAUVAIS: Viole LSP
class Rectangle {
public:
    virtual void setWidth(int w) { width = w; }
    virtual void setHeight(int h) { height = h; }
    virtual int area() { return width * height; }
};

class Square : public Rectangle {
public:
    void setWidth(int w) override {
        width = height = w;  // Carré! Pas ce qu'attendait Rectangle
    }
    void setHeight(int h) override {
        width = height = h;
    }
};

// ✅ BON: Hiérarchie correcte
class Shape {
public:
    virtual int area() = 0;
};

class Rectangle : public Shape { /* ... */ };
class Square : public Shape { /* ... */ };
```

---

## 📊 Résumé Comparatif

| Concept | Syntaxe | Usage | Recommandation |
|---------|---------|-------|---|
| **Héritage Public** | `class D : public B` | IS-A, polymorphisme | ✅ Courant |
| **Héritage Privé** | `class D : private B` | Composition impl. | ⚠️ Rare |
| **Héritage Protégé** | `class D : protected B` | Accès sous-classes | ❌ Très rare |
| **Composition** | `B member;` | HAS-A | ✅ Préférer |
| **Virtual** | `virtual void method()` | Polymorphisme dynamique | ✅ Courant |
| **Héritage Multiple** | `class D : public B1, B2` | Interfaces | ⚠️ Avec caution |
| **Héritage Virtuel** | `virtual public Base` | Résoudre diamant | ❌ Très rare |

---

## 🎯 Règle d'Or

> **Préférez la Composition à l'Héritage**

```cpp
// La plupart du temps:
// ✅ Composition + Interface
class Component {
    shared_ptr<Base> impl;  // Flexible, changeable
};

// Seulement si vraiment IS-A:
// ✅ Héritage Public
class Derived : public Base { };
```

---

[⬅️ Retour à l'Index](./INDEX.md) | [➡️ Patrons](./GRASP/README.md)
