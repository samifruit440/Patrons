# 👁️ Patron Observateur (Observer)

> **Patron Comportemental** - Définit une dépendance un-à-plusieurs pour notifier automatiquement les changements d'état.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Définir une relation de **dépendance un-à-plusieurs** entre objets, de façon que lorsqu'un objet change d'état, tous ses dépendants soient **notifiés** et mis à jour **automatiquement**.

---

## 🎯 Problème Résolu

- Comment notifier plusieurs objets d'un changement sans créer de couplage fort?
- Comment permettre un nombre variable d'observateurs?
- Comment éviter le polling constant pour vérifier les changements?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON OBSERVATEUR                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│    ┌──────────────────────┐       ┌──────────────────────┐      │
│    │      «interface»     │       │      «interface»     │      │
│    │        Sujet         │       │      Observateur     │      │
│    ├──────────────────────┤       ├──────────────────────┤      │
│    │ +attacher(obs)       │       │ +mettreAJour(sujet)  │      │
│    │ +detacher(obs)       │◆────►│                      │      │
│    │ +notifier()          │       └──────────┬───────────┘      │
│    └──────────┬───────────┘                  │                  │
│               │                              │                  │
│               │                              │                  │
│    ┌──────────▼───────────┐       ┌──────────▼───────────┐      │
│    │     SujetConcret     │       │  ObservateurConcret  │      │
│    ├──────────────────────┤       ├──────────────────────┤      │
│    │ -etat                │       │ -etatObservateur     │      │
│    │ -observateurs[]      │       │ -sujet               │      │
│    ├──────────────────────┤       ├──────────────────────┤      │
│    │ +getEtat()           │◄──────│ +mettreAJour()       │      │
│    │ +setEtat(e)          │       └──────────────────────┘      │
│    │ +notifier()          │                                     │
│    └──────────────────────┘                                     │
│                                                                 │
│    notifier() {                                                 │
│        for (obs : observateurs)                                 │
│            obs.mettreAJour(this)                                │
│    }                                                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Flux d'Exécution

```
┌─────────────────────────────────────────────────────────────────┐
│                    SÉQUENCE D'EXÉCUTION                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Client        Sujet           Obs1            Obs2            │
│     │             │               │               │             │
│     │ setEtat(x)  │               │               │             │
│     │────────────►│               │               │             │
│     │             │               │               │             │
│     │             │ notifier()    │               │             │
│     │             │──────────────►│               │             │
│     │             │  mettreAJour()│               │             │
│     │             │               │               │             │
│     │             │──────────────────────────────►│             │
│     │             │            mettreAJour()      │             │
│     │             │               │               │             │
│     │             │◄──────────────│               │             │
│     │             │    getEtat()  │               │             │
│     │             │◄─────────────────────────────►│             │
│     │             │                   getEtat()   │             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interfaces

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

// Forward declaration
class ISujet;

// Interface Observateur
class IObservateur {
public:
    virtual void mettreAJour(ISujet* sujet) = 0;
    virtual ~IObservateur() = default;
};

// Interface Sujet
class ISujet {
public:
    virtual void attacher(IObservateur* obs) = 0;
    virtual void detacher(IObservateur* obs) = 0;
    virtual void notifier() = 0;
    virtual ~ISujet() = default;
};
```

### Sujet Concret

```cpp
// Sujet Concret: Station Météo
class StationMeteo : public ISujet {
private:
    std::vector<IObservateur*> observateurs;
    float temperature;
    float humidite;
    float pression;

public:
    void attacher(IObservateur* obs) override {
        observateurs.push_back(obs);
        std::cout << "Observateur attaché. Total: " 
                  << observateurs.size() << std::endl;
    }
    
    void detacher(IObservateur* obs) override {
        observateurs.erase(
            std::remove(observateurs.begin(), observateurs.end(), obs),
            observateurs.end()
        );
        std::cout << "Observateur détaché. Total: " 
                  << observateurs.size() << std::endl;
    }
    
    void notifier() override {
        std::cout << "\n=== Notification des observateurs ===" << std::endl;
        for (IObservateur* obs : observateurs) {
            obs->mettreAJour(this);
        }
    }
    
    // Méthode qui déclenche la notification
    void setMesures(float temp, float hum, float pres) {
        std::cout << "\nNouvelles mesures reçues!" << std::endl;
        temperature = temp;
        humidite = hum;
        pression = pres;
        notifier();  // Notifie automatiquement
    }
    
    // Getters pour les observateurs
    float getTemperature() const { return temperature; }
    float getHumidite() const { return humidite; }
    float getPression() const { return pression; }
};
```

### Observateurs Concrets

```cpp
// Observateur 1: Affichage Actuel
class AffichageActuel : public IObservateur {
private:
    float temperature;
    float humidite;
    
public:
    void mettreAJour(ISujet* sujet) override {
        StationMeteo* station = dynamic_cast<StationMeteo*>(sujet);
        if (station) {
            temperature = station->getTemperature();
            humidite = station->getHumidite();
            afficher();
        }
    }
    
    void afficher() {
        std::cout << "[Affichage Actuel] Température: " << temperature 
                  << "°C, Humidité: " << humidite << "%" << std::endl;
    }
};

// Observateur 2: Affichage Statistiques
class AffichageStatistiques : public IObservateur {
private:
    float tempMin = 999;
    float tempMax = -999;
    float tempTotal = 0;
    int nbLectures = 0;
    
public:
    void mettreAJour(ISujet* sujet) override {
        StationMeteo* station = dynamic_cast<StationMeteo*>(sujet);
        if (station) {
            float temp = station->getTemperature();
            tempTotal += temp;
            nbLectures++;
            
            if (temp < tempMin) tempMin = temp;
            if (temp > tempMax) tempMax = temp;
            
            afficher();
        }
    }
    
    void afficher() {
        std::cout << "[Statistiques] Moy: " << (tempTotal / nbLectures)
                  << "°C, Min: " << tempMin 
                  << "°C, Max: " << tempMax << "°C" << std::endl;
    }
};

// Observateur 3: Alerte Température
class AlerteTemperature : public IObservateur {
private:
    float seuilAlerte;
    
public:
    AlerteTemperature(float seuil) : seuilAlerte(seuil) {}
    
    void mettreAJour(ISujet* sujet) override {
        StationMeteo* station = dynamic_cast<StationMeteo*>(sujet);
        if (station) {
            float temp = station->getTemperature();
            if (temp > seuilAlerte) {
                std::cout << "[⚠️ ALERTE] Température élevée: " << temp 
                          << "°C (seuil: " << seuilAlerte << "°C)" << std::endl;
            }
        }
    }
};
```

### Utilisation

```cpp
int main() {
    // Créer le sujet
    StationMeteo* station = new StationMeteo();
    
    // Créer les observateurs
    AffichageActuel* affichage = new AffichageActuel();
    AffichageStatistiques* stats = new AffichageStatistiques();
    AlerteTemperature* alerte = new AlerteTemperature(30.0f);
    
    // Attacher les observateurs
    station->attacher(affichage);
    station->attacher(stats);
    station->attacher(alerte);
    
    // Simuler des changements de mesures
    station->setMesures(25.0f, 65.0f, 1013.0f);
    station->setMesures(28.0f, 70.0f, 1015.0f);
    station->setMesures(32.0f, 80.0f, 1010.0f);  // Déclenche l'alerte!
    
    // Détacher un observateur
    std::cout << "\n--- Détachement de l'alerte ---" << std::endl;
    station->detacher(alerte);
    
    station->setMesures(35.0f, 85.0f, 1008.0f);  // Plus d'alerte
    
    // Nettoyage
    delete alerte;
    delete stats;
    delete affichage;
    delete station;
    
    return 0;
}
```

### Sortie

```
Observateur attaché. Total: 1
Observateur attaché. Total: 2
Observateur attaché. Total: 3

Nouvelles mesures reçues!

=== Notification des observateurs ===
[Affichage Actuel] Température: 25°C, Humidité: 65%
[Statistiques] Moy: 25°C, Min: 25°C, Max: 25°C

Nouvelles mesures reçues!

=== Notification des observateurs ===
[Affichage Actuel] Température: 28°C, Humidité: 70%
[Statistiques] Moy: 26.5°C, Min: 25°C, Max: 28°C

Nouvelles mesures reçues!

=== Notification des observateurs ===
[Affichage Actuel] Température: 32°C, Humidité: 80%
[Statistiques] Moy: 28.33°C, Min: 25°C, Max: 32°C
[⚠️ ALERTE] Température élevée: 32°C (seuil: 30°C)

--- Détachement de l'alerte ---
Observateur détaché. Total: 2

Nouvelles mesures reçues!

=== Notification des observateurs ===
[Affichage Actuel] Température: 35°C, Humidité: 85%
[Statistiques] Moy: 30°C, Min: 25°C, Max: 35°C
```

---

## 🔄 Variantes du Patron

### Push vs Pull

```cpp
// PUSH: Le sujet envoie les données
class IObservateurPush {
public:
    virtual void mettreAJour(float temp, float hum) = 0;  // Données poussées
};

// PULL: L'observateur récupère les données
class IObservateurPull {
public:
    virtual void mettreAJour(ISujet* sujet) = 0;  // L'observateur "tire" ce qu'il veut
};
```

### Avec événements typés

```cpp
enum class TypeEvenement { TEMPERATURE, HUMIDITE, PRESSION };

class IObservateur {
public:
    virtual void mettreAJour(TypeEvenement type, float valeur) = 0;
};

class StationMeteo {
public:
    void setTemperature(float temp) {
        temperature = temp;
        notifier(TypeEvenement::TEMPERATURE, temp);
    }
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Couplage faible** | Sujet et observateurs sont indépendants |
| **Dynamique** | Ajout/retrait d'observateurs à l'exécution |
| **Broadcast** | Un événement peut notifier plusieurs objets |
| **Open/Closed** | Nouveaux observateurs sans modifier le sujet |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Ordre imprévisible** | L'ordre de notification n'est pas garanti |
| **Fuites mémoire** | Oubli de détacher = observateurs orphelins |
| **Mises à jour en cascade** | Un observateur modifie un autre sujet |
| **Performance** | Beaucoup d'observateurs = lent |

---

## 🎯 Cas d'Utilisation

1. **Interface graphique** - MVC, liaison de données
2. **Systèmes d'événements** - Listeners, callbacks
3. **Messaging** - Publish/Subscribe
4. **Notifications** - Alertes, mises à jour
5. **Logging** - Multiple destinations

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Médiateur](./Mediateur.md) | Centralise les communications |
| [Singleton](../Creationnel/Singleton.md) | Le sujet peut être unique |
| [Commande](./Commande.md) | Encapsule les notifications |

---

## 💡 Bonnes Pratiques

1. **Gérer les détachements** - Éviter les fuites mémoire
2. **Éviter les boucles** - Observer qui notifie son propre sujet
3. **Thread-safety** - Synchroniser dans les environnements multi-threads
4. **Weak references** - Utiliser des références faibles si possible

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Décorateur](../Structurel/Decorateur.md)
