# 🏭 Patrons Créationnels

> Guide rapide : **quel problème → quel patron ?**

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📖 Introduction

Les **patrons créationnels** gèrent la **création d'objets** de façon à découpler le code client des classes concrètes qu'il instancie. Ils permettent de rendre le système indépendant de la manière dont les objets sont créés, composés et représentés.

---

## 🗺️ Guide « Problème → Patron »

| Problème typique | Patron conseillé | Pourquoi |
|------------------|------------------|----------|
| Créer un objet sans spécifier sa classe exacte (produit variable) | [**Fabrique (Factory Method)**](Fabrique.md) | Délègue l'instanciation aux sous-classes via une méthode virtuelle. |
| Créer des familles d'objets liés sans dépendre des classes concrètes (thèmes UI, plateformes) | [**Fabrique Abstraite**](FabriqueAbstraite.md) | Fournit une interface pour créer une famille de produits cohérents. |
| S'assurer qu'une classe n'a qu'une seule instance (config, cache, logger) | [**Singleton**](Singleton.md) | Contrôle l'instanciation et fournit un point d'accès global unique. |

---

## 🔎 Arbre de décision rapide

```
Besoin de...
│
├─► Instancier UN type variable selon contexte ──────────► Fabrique (Factory Method)
│
├─► Instancier une FAMILLE de types cohérents ───────────► Fabrique Abstraite
│
└─► Garantir UNE SEULE instance globale ─────────────────► Singleton
```

---

## ⚖️ Comparaison rapide

| Critère | Fabrique | Fabrique Abstraite | Singleton |
|---------|----------|--------------------|-----------|
| **Portée** | Un produit | Famille de produits | Instance unique |
| **Mécanisme** | Méthode virtuelle | Interface/groupe de fabriques | Constructeur privé + accès static |
| **Extensibilité** | Ajouter sous-classes | Ajouter nouvelle fabrique concrète | Limitée (une instance) |
| **Cas d'usage** | Création conditionnelle | Thèmes, plateformes, bases de données | Configuration, logging, cache |

---

## 📚 Fiches détaillées

| Patron | Fiche |
|--------|-------|
| Fabrique (Factory Method) | [Fabrique.md](Fabrique.md) |
| Fabrique Abstraite | [FabriqueAbstraite.md](FabriqueAbstraite.md) |
| Singleton | [Singleton.md](Singleton.md) |

---

## 💡 Conseils

1. **Fabrique vs Fabrique Abstraite** : Factory Method crée *un* produit ; Abstract Factory crée *plusieurs* produits liés.
2. **Singleton avec précaution** : pratique mais peut créer un couplage global ; privilégier l'injection de dépendances quand c'est possible.
3. **Combiner les patrons** : Abstract Factory utilise souvent des Factory Methods en interne.

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Patrons Comportementaux](../Comportemental/README.md) | [➡️ Patrons Structurels](../Structurel/README.md)
