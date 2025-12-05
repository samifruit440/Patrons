# 🎭 Patrons Comportementaux

> Guide rapide : **quel problème → quel patron ?**

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📖 Introduction

Les **patrons comportementaux** définissent la manière dont les objets **communiquent** et **répartissent les responsabilités** entre eux. Ils permettent de gérer des algorithmes, des flux de contrôle et des interactions complexes de façon modulaire et extensible.

---

## 🗺️ Guide « Problème → Patron »

| Problème typique | Patron conseillé | Pourquoi |
|------------------|------------------|----------|
| Besoin de découpler l'émetteur d'une requête de son récepteur (annuler/refaire, file de commandes, macro) | [**Commande**](Commande.md) | Encapsule une requête en objet ; permet undo/redo, file d'attente, journalisation. |
| Un objet change de comportement selon son état interne (machine à états) | [**État**](Etat.md) | Délègue le comportement à des objets-états ; évite les gros `switch/case`. |
| Parcourir une collection sans exposer sa structure interne | [**Itérateur**](Iterateur.md) | Fournit une interface uniforme (`next()`, `hasNext()`) pour traverser n'importe quelle collection. |
| Trop de dépendances croisées entre objets (dialogue, formulaire, chat) | [**Médiateur**](Mediateur.md) | Centralise la communication ; les objets ne connaissent que le médiateur. |
| Notifier plusieurs objets quand un autre change (événements, MVC) | [**Observateur**](Observateur.md) | Définit une relation 1-N : le sujet notifie automatiquement les observateurs. |
| Définir le squelette d'un algorithme, mais laisser certaines étapes aux sous-classes | [**Patron Méthode**](PatronMethode.md) | La superclasse définit le flux global ; les sous-classes redéfinissent les étapes variables. |
| Choisir dynamiquement un algorithme parmi plusieurs (tri, validation, tarification) | [**Stratégie**](Strategie.md) | Encapsule chaque algorithme dans une classe ; le client choisit à l'exécution. |
| Ajouter des opérations à une structure d'objets sans modifier leurs classes | [**Visiteur**](Visiteur.md) | Sépare l'algorithme de la structure ; double dispatch pour traiter chaque type. |

---

## 🔎 Arbre de décision rapide

```
Besoin de...
│
├─► Encapsuler une action (undo, queue, log) ────────────► Commande
│
├─► Gérer des états/transitions ─────────────────────────► État
│
├─► Parcourir une collection ────────────────────────────► Itérateur
│
├─► Réduire le couplage entre N objets ──────────────────► Médiateur
│
├─► Notifier des changements (publish/subscribe) ────────► Observateur
│
├─► Fixer un algorithme mais varier certaines étapes ────► Patron Méthode
│
├─► Changer d'algorithme à l'exécution ──────────────────► Stratégie
│
└─► Ajouter des opérations sans toucher aux classes ─────► Visiteur
```

---

## 📚 Fiches détaillées

| Patron | Fiche |
|--------|-------|
| Commande | [Commande.md](Commande.md) |
| État | [Etat.md](Etat.md) |
| Itérateur | [Iterateur.md](Iterateur.md) |
| Médiateur | [Mediateur.md](Mediateur.md) |
| Observateur | [Observateur.md](Observateur.md) |
| Patron Méthode | [PatronMethode.md](PatronMethode.md) |
| Stratégie | [Strategie.md](Strategie.md) |
| Visiteur | [Visiteur.md](Visiteur.md) |

---

## 💡 Conseils

1. **Commande vs Stratégie** : Commande encapsule *une requête* (avec paramètres, historique) ; Stratégie encapsule *un algorithme* interchangeable.
2. **État vs Stratégie** : État change *automatiquement* de comportement selon l'état interne ; Stratégie est choisie *explicitement* par le client.
3. **Observateur vs Médiateur** : Observateur est 1-N unidirectionnel ; Médiateur est N-N bidirectionnel centralisé.
4. **Patron Méthode vs Stratégie** : Patron Méthode utilise l'héritage (structure fixe, étapes variables) ; Stratégie utilise la composition (algorithme entier remplaçable).

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Patrons Créationnels](../Creationnel/README.md) | [➡️ Patrons Structurels](../Structurel/README.md)
