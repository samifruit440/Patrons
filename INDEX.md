# 📚 Guide

> Documentation complète sur les principes SOLID, patrons GRASP et Design Patterns

---

## 🧭 Navigation Rapide

### 📐 Principes Fondamentaux
| Sujet | Description |
|-------|-------------|
| [🎯 Principes SOLID](./SOLID/README.md) | Les 5 principes de conception orientée objet |
| [🎓 Patrons GRASP](./GRASP/README.md) | General Responsibility Assignment Software Patterns |

### 🏗️ Patrons de Conception (Design Patterns)

#### Patrons Créationnels
| Patron | Description |
|--------|-------------|
| [🏭 Fabrique (Factory)](./Patrons/Creationnel/Fabrique.md) | Crée des objets sans spécifier leur classe concrète |
| [🏭 Fabrique Abstraite](./Patrons/Creationnel/FabriqueAbstraite.md) | Familles d'objets liés sans spécifier classes concrètes |
| [1️⃣ Singleton](./Patrons/Creationnel/Singleton.md) | Garantit une instance unique d'une classe |

#### Patrons Structurels
| Patron | Description |
|--------|-------------|
| [🎨 Décorateur](./Patrons/Structurel/Decorateur.md) | Ajoute dynamiquement des responsabilités |
| [🌳 Composite](./Patrons/Structurel/Composite.md) | Compose des objets en structures arborescentes |
| [🔌 Adaptateur](./Patrons/Structurel/Adaptateur.md) | Convertit l'interface d'une classe |
| [🏠 Façade](./Patrons/Structurel/Facade.md) | Interface simplifiée pour un sous-système |
| [🔗 Proxy](./Patrons/Structurel/Proxy.md) | Substitut contrôlant l'accès à un objet |

#### Patrons Comportementaux
| Patron | Description |
|--------|-------------|
| [👁️ Observateur](./Patrons/Comportemental/Observateur.md) | Notification de changement d'état |
| [📋 Patron de Méthode](./Patrons/Comportemental/PatronMethode.md) | Squelette d'algorithme avec étapes redéfinissables |
| [🔄 Itérateur](./Patrons/Comportemental/Iterateur.md) | Parcours séquentiel d'une collection |
| [⌨️ Commande](./Patrons/Comportemental/Commande.md) | Encapsule une requête comme objet |
| [🔀 État](./Patrons/Comportemental/Etat.md) | Modifie le comportement selon l'état |
| [👤 Visiteur](./Patrons/Comportemental/Visiteur.md) | Opérations sur éléments d'une structure |
| [🎛️ Médiateur](./Patrons/Comportemental/Mediateur.md) | Encapsule les interactions entre objets |
| [📊 Stratégie](./Patrons/Comportemental/Strategie.md) | Famille d'algorithmes interchangeables |

---

## 📊 Classification des Patrons GoF

```
┌─────────────────────────────────────────────────────────────────┐
│                    PATRONS DE CONCEPTION                        │
├─────────────────┬─────────────────┬─────────────────────────────┤
│   CRÉATIONNELS  │   STRUCTURELS   │       COMPORTEMENTAUX       │
├─────────────────┼─────────────────┼─────────────────────────────┤
│ • Fabrique      │ • Adaptateur    │ • Observateur               │
│ • Fabrique      │ • Composite     │ • Patron de Méthode         │
│   Abstraite     │ • Décorateur    │ • Itérateur                 │
│ • Singleton     │ • Façade        │ • Commande                  │
│                 │ • Proxy         │ • État                      │
│                 │                 │ • Visiteur                  │
│                 │                 │ • Médiateur                 │
│                 │                 │ • Stratégie                 │
└─────────────────┴─────────────────┴─────────────────────────────┘
```

---

## 🎯 Par où commencer?

1. **Débutant** → Commencez par [SOLID](./SOLID/README.md) et [GRASP](./GRASP/README.md)
2. **Intermédiaire** → Explorez les patrons [Stratégie](./Patrons/Comportemental/Strategie.md), [Observateur](./Patrons/Comportemental/Observateur.md) et [Fabrique](./Patrons/Creationnel/Fabrique.md)
3. **Avancé** → Maîtrisez [Visiteur](./Patrons/Comportemental/Visiteur.md), [Médiateur](./Patrons/Comportemental/Mediateur.md) et [Fabrique Abstraite](./Patrons/Creationnel/FabriqueAbstraite.md)

---

## 📖 Ressources

- **Livre de référence**: "Design Patterns: Elements of Reusable Object-Oriented Software" (Gang of Four)
- **Livre GRASP**: "Applying UML and Patterns" - Craig Larman
- **Polymtl**: LOG2400 Analyse et Conception de logiciels

---

*Dernière mise à jour: Décembre 2025*
