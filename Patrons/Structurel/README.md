# 🧱 Patrons Structurels

> Guide rapide : **quel problème → quel patron ?**

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📖 Introduction

Les **patrons structurels** s'intéressent à la **composition des classes et objets** pour former des structures plus grandes tout en gardant de la flexibilité et de l'efficacité. Ils aident à créer des interfaces claires et à réutiliser du code existant.

---

## 🗺️ Guide « Problème → Patron »

| Problème typique | Patron conseillé | Pourquoi |
|------------------|------------------|----------|
| Intégrer une classe existante dont l'interface ne correspond pas à celle attendue | [**Adaptateur**](Adaptateur.md) | Convertit l'interface d'une classe en une autre interface attendue par le client. |
| Représenter une hiérarchie partie-tout (arbres, menus, systèmes de fichiers) | [**Composite**](Composite.md) | Traite uniformément les objets individuels et les compositions d'objets. |
| Ajouter dynamiquement des responsabilités à un objet sans modifier sa classe | [**Décorateur**](Decorateur.md) | Enveloppe un objet pour lui ajouter des comportements ; empilable. |
| Simplifier l'accès à un sous-système complexe (bibliothèque, API) | [**Façade**](Facade.md) | Fournit une interface unifiée et simplifiée pour un ensemble de classes. |
| Contrôler l'accès à un objet (lazy loading, contrôle d'accès, cache, logging) | [**Proxy**](Proxy.md) | Fournit un substitut qui contrôle l'accès à l'objet réel. |

---

## 🔎 Arbre de décision rapide

```
Besoin de...
│
├─► Faire collaborer deux interfaces incompatibles ─────────► Adaptateur
│
├─► Traiter des objets simples et composés de la même façon ► Composite
│
├─► Ajouter des comportements à un objet à l'exécution ─────► Décorateur
│
├─► Simplifier une API complexe ────────────────────────────► Façade
│
└─► Contrôler/différer l'accès à un objet ──────────────────► Proxy
```

---

## ⚖️ Comparaison rapide

| Critère | Adaptateur | Composite | Décorateur | Façade | Proxy |
|---------|------------|-----------|------------|--------|-------|
| **But** | Compatibilité d'interface | Structure arborescente | Ajout de comportement | Simplification | Contrôle d'accès |
| **Mécanisme** | Wrapper convertisseur | Récursivité composant/composite | Wrapper empilable | Interface unique | Substitut transparent |
| **Modifie l'interface ?** | Oui (traduit) | Non (unifie) | Non (étend) | Oui (simplifie) | Non (même interface) |

---

## 📚 Fiches détaillées

| Patron | Fiche |
|--------|-------|
| Adaptateur | [Adaptateur.md](Adaptateur.md) |
| Composite | [Composite.md](Composite.md) |
| Décorateur | [Decorateur.md](Decorateur.md) |
| Façade | [Facade.md](Facade.md) |
| Proxy | [Proxy.md](Proxy.md) |

---

## 💡 Conseils

1. **Adaptateur vs Façade** : Adaptateur rend deux interfaces compatibles ; Façade simplifie une interface (pas de traduction de types).
2. **Décorateur vs Proxy** : Décorateur ajoute des fonctionnalités ; Proxy contrôle l'accès (lazy load, sécurité, cache).
3. **Composite** : idéal pour les structures récursives (menus, arborescences de fichiers, composants graphiques).
4. **Empiler les Décorateurs** : on peut en chaîner plusieurs (`new BufferedStream(new GzipStream(file))`).

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Patrons Comportementaux](../Comportemental/README.md) | [➡️ Patrons Créationnels](../Creationnel/README.md)
