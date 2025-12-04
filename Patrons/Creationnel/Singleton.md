# 1️⃣ Patron Singleton

> **Patron Créationnel** - Garantit qu'une classe n'a qu'une seule instance et fournit un point d'accès global à cette instance.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Garantir qu'une classe n'a **qu'une seule instance** et fournir un **point d'accès global** à celle-ci.

---

## 🎯 Problème Résolu

- Comment s'assurer qu'une classe n'a qu'une seule instance?
- Comment fournir un point d'accès global sans variable globale?
- Comment contrôler l'accès à une ressource partagée?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON SINGLETON                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│                    ┌───────────────────────────┐                │
│                    │        Singleton          │                │
│                    ├───────────────────────────┤                │
│                    │ -instance: Singleton      │ ◄── statique   │
│                    │ -donnees                  │                │
│                    ├───────────────────────────┤                │
│                    │ -Singleton() { }          │ ◄── privé      │
│                    │ +getInstance(): Singleton │ ◄── statique   │
│                    │ +operation()              │                │
│                    └───────────────────────────┘                │
│                              ▲                                  │
│                              │                                  │
│                         retourne                                │
│                              │                                  │
│         ┌────────────────────┴────────────────────┐             │
│         │                                         │             │
│    ┌────▼────┐                              ┌─────▼────┐        │
│    │ Client1 │                              │ Client2  │        │
│    └─────────┘                              └──────────┘        │
│                                                                 │
│    Singleton::getInstance()                                     │
│    → Toujours la MÊME instance                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚠️ Différentes Implémentations

```
┌─────────────────────────────────────────────────────────────────┐
│              IMPLÉMENTATIONS DU SINGLETON                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. LAZY (création à la demande)                               │
│      ┌─────────────────────────────────┐                        │
│      │ getInstance() {                 │                        │
│      │   if (instance == null)         │ Créé au premier appel  │
│      │     instance = new Singleton(); │                        │
│      │   return instance;              │                        │
│      │ }                               │                        │
│      └─────────────────────────────────┘                        │
│      ⚠️ Problème: pas thread-safe                               │
│                                                                 │
│   2. EAGER (création au chargement)                             │
│      ┌─────────────────────────────────┐                        │
│      │ static Singleton instance = ... │ Créé au démarrage      │
│      │ getInstance() { return instance }│                       │
│      └─────────────────────────────────┘                        │
│      ✓ Thread-safe, mais créé même si non utilisé               │
│                                                                 │
│   3. THREAD-SAFE (avec synchronisation)                         │
│      ┌─────────────────────────────────┐                        │
│      │ getInstance() {                 │                        │
│      │   lock();                       │ Verrou de mutex        │
│      │   if (instance == null)         │                        │
│      │     instance = new Singleton(); │                        │
│      │   unlock();                     │                        │
│      │   return instance;              │                        │
│      │ }                               │                        │
│      └─────────────────────────────────┘                        │
│      ⚠️ Performance: verrou à chaque appel                      │
│                                                                 │
│   4. DOUBLE-CHECKED LOCKING                                     │
│      ┌─────────────────────────────────┐                        │
│      │ getInstance() {                 │                        │
│      │   if (instance == null) {       │ Premier check          │
│      │     lock();                     │                        │
│      │     if (instance == null)       │ Second check           │
│      │       instance = new...;        │                        │
│      │     unlock();                   │                        │
│      │   }                             │                        │
│      │   return instance;              │                        │
│      │ }                               │                        │
│      └─────────────────────────────────┘                        │
│      ✓ Thread-safe et performant                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Singleton Simple (Non thread-safe)

```cpp
#include <iostream>
#include <string>
#include <map>

// ⚠️ Version simple - NON thread-safe
class ConfigurationSimple {
private:
    static ConfigurationSimple* instance;
    std::map<std::string, std::string> settings;
    
    // Constructeur privé
    ConfigurationSimple() {
        std::cout << "🔧 Création du Singleton Configuration" << std::endl;
        // Charger la configuration par défaut
        settings["app.name"] = "MonApplication";
        settings["app.version"] = "1.0.0";
        settings["db.host"] = "localhost";
    }
    
public:
    // Supprimer copie et assignation
    ConfigurationSimple(const ConfigurationSimple&) = delete;
    ConfigurationSimple& operator=(const ConfigurationSimple&) = delete;
    
    static ConfigurationSimple* getInstance() {
        if (instance == nullptr) {
            instance = new ConfigurationSimple();
        }
        return instance;
    }
    
    std::string get(const std::string& key) const {
        auto it = settings.find(key);
        return (it != settings.end()) ? it->second : "";
    }
    
    void set(const std::string& key, const std::string& value) {
        settings[key] = value;
    }
};

// Initialisation du membre statique
ConfigurationSimple* ConfigurationSimple::instance = nullptr;
```

### Singleton Thread-Safe (C++11 Meyer's Singleton)

```cpp
#include <iostream>
#include <string>
#include <map>
#include <mutex>

// ✅ Version thread-safe avec C++11 (Meyer's Singleton)
class Logger {
private:
    std::map<std::string, int> compteurs;
    std::mutex mutex;
    
    // Constructeur privé
    Logger() {
        std::cout << "📝 Création du Singleton Logger" << std::endl;
    }
    
public:
    // Supprimer copie et assignation
    Logger(const Logger&) = delete;
    Logger& operator=(const Logger&) = delete;
    
    // Meyer's Singleton - thread-safe en C++11
    static Logger& getInstance() {
        static Logger instance;  // Initialisé une seule fois
        return instance;
    }
    
    void log(const std::string& niveau, const std::string& message) {
        std::lock_guard<std::mutex> lock(mutex);
        compteurs[niveau]++;
        std::cout << "[" << niveau << "] " << message << std::endl;
    }
    
    void info(const std::string& msg)    { log("INFO", msg); }
    void warning(const std::string& msg) { log("WARNING", msg); }
    void error(const std::string& msg)   { log("ERROR", msg); }
    
    void afficherStatistiques() {
        std::lock_guard<std::mutex> lock(mutex);
        std::cout << "\n📊 Statistiques du Logger:" << std::endl;
        for (const auto& pair : compteurs) {
            std::cout << "   " << pair.first << ": " << pair.second << std::endl;
        }
    }
};
```

### Singleton avec Double-Checked Locking

```cpp
#include <mutex>
#include <memory>
#include <atomic>

// Version avec double-checked locking (pré-C++11 style)
class DatabaseConnection {
private:
    static std::atomic<DatabaseConnection*> instance;
    static std::mutex mutex;
    
    std::string connectionString;
    bool connected = false;
    
    DatabaseConnection(const std::string& connStr) 
        : connectionString(connStr) {
        std::cout << "🔌 Création connexion DB: " << connStr << std::endl;
    }
    
public:
    DatabaseConnection(const DatabaseConnection&) = delete;
    DatabaseConnection& operator=(const DatabaseConnection&) = delete;
    
    static DatabaseConnection* getInstance(const std::string& connStr = "default") {
        DatabaseConnection* tmp = instance.load(std::memory_order_acquire);
        
        if (tmp == nullptr) {
            std::lock_guard<std::mutex> lock(mutex);
            tmp = instance.load(std::memory_order_relaxed);
            
            if (tmp == nullptr) {
                tmp = new DatabaseConnection(connStr);
                instance.store(tmp, std::memory_order_release);
            }
        }
        return tmp;
    }
    
    void connect() {
        if (!connected) {
            std::cout << "   → Connexion établie" << std::endl;
            connected = true;
        }
    }
    
    void query(const std::string& sql) {
        if (connected) {
            std::cout << "   → Exécution: " << sql << std::endl;
        }
    }
    
    void disconnect() {
        if (connected) {
            std::cout << "   → Déconnexion" << std::endl;
            connected = false;
        }
    }
    
    static void resetInstance() {
        std::lock_guard<std::mutex> lock(mutex);
        delete instance.load();
        instance.store(nullptr);
    }
};

// Initialisation
std::atomic<DatabaseConnection*> DatabaseConnection::instance{nullptr};
std::mutex DatabaseConnection::mutex;
```

### Utilisation

```cpp
int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON SINGLETON - DEMO          ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // Test Logger (Meyer's Singleton)
    std::cout << "\n=== Test Logger ===" << std::endl;
    
    Logger::getInstance().info("Application démarrée");
    Logger::getInstance().info("Chargement des modules");
    Logger::getInstance().warning("Configuration par défaut utilisée");
    Logger::getInstance().error("Fichier non trouvé");
    Logger::getInstance().info("Module chargé");
    
    // Même instance
    Logger& log1 = Logger::getInstance();
    Logger& log2 = Logger::getInstance();
    
    std::cout << "\nMême instance? " 
              << (&log1 == &log2 ? "OUI ✓" : "NON ✗") << std::endl;
    
    Logger::getInstance().afficherStatistiques();
    
    // Test DatabaseConnection
    std::cout << "\n=== Test Database ===" << std::endl;
    
    auto* db1 = DatabaseConnection::getInstance("mysql://localhost:3306/app");
    db1->connect();
    db1->query("SELECT * FROM users");
    
    // Deuxième appel - même instance
    auto* db2 = DatabaseConnection::getInstance("autre_connexion");  // Ignoré!
    db2->query("INSERT INTO logs VALUES(...)");
    
    std::cout << "\nMême instance? " 
              << (db1 == db2 ? "OUI ✓" : "NON ✗") << std::endl;
    
    db1->disconnect();
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON SINGLETON - DEMO          ║
╚═══════════════════════════════════════╝

=== Test Logger ===
📝 Création du Singleton Logger
[INFO] Application démarrée
[INFO] Chargement des modules
[WARNING] Configuration par défaut utilisée
[ERROR] Fichier non trouvé
[INFO] Module chargé

Même instance? OUI ✓

📊 Statistiques du Logger:
   ERROR: 1
   INFO: 3
   WARNING: 1

=== Test Database ===
🔌 Création connexion DB: mysql://localhost:3306/app
   → Connexion établie
   → Exécution: SELECT * FROM users
   → Exécution: INSERT INTO logs VALUES(...)

Même instance? OUI ✓
   → Déconnexion
```

---

## 🧪 Singleton et Tests

```cpp
// Problème: Singleton difficile à tester

// Solution 1: Interface pour injection
class ILogger {
public:
    virtual void log(const std::string& msg) = 0;
    virtual ~ILogger() = default;
};

class RealLogger : public ILogger {
public:
    void log(const std::string& msg) override { /* ... */ }
};

class MockLogger : public ILogger {
public:
    std::vector<std::string> messages;
    void log(const std::string& msg) override {
        messages.push_back(msg);
    }
};

// Solution 2: Méthode de reset pour les tests
class TestableLogger {
    static TestableLogger* instance;
public:
    static TestableLogger& getInstance();
    
    // Pour les tests seulement!
    static void resetForTesting() {
        delete instance;
        instance = nullptr;
    }
};
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Instance unique** | Garantie d'une seule instance |
| **Accès global** | Point d'accès connu |
| **Lazy loading** | Création à la demande |
| **Contrôle** | Gestion centralisée d'une ressource |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **État global** | Difficile à tester |
| **Couplage** | Code dépend du Singleton |
| **Thread-safety** | Complexe à implémenter correctement |
| **Lifetime** | Destruction pas toujours claire |

---

## ⚠️ Anti-Pattern?

Le Singleton est souvent considéré comme un **anti-pattern** car:

1. **État global caché** - Dépendances non explicites
2. **Tests difficiles** - Impossible d'isoler
3. **Violation SRP** - Gère sa création ET sa logique
4. **Couplage fort** - Code directement lié au Singleton

### Alternatives

```cpp
// ❌ Singleton direct
void maFonction() {
    Logger::getInstance().log("...");  // Couplage fort
}

// ✅ Injection de dépendance
void maFonction(ILogger& logger) {
    logger.log("...");  // Découplé, testable
}
```

---

## 🎯 Cas d'Utilisation Légitimes

1. **Logging** - Un seul point de journalisation
2. **Configuration** - Paramètres globaux
3. **Cache** - Cache partagé
4. **Pool de connexions** - Ressource limitée
5. **Registry** - Registre d'objets

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Fabrique Abstraite](./FabriqueAbstraite.md) | Souvent un singleton |
| [État](../Comportemental/Etat.md) | Objets état peuvent être singletons |
| [Façade](../Structurel/Facade.md) | Peut être un singleton |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ Adaptateur](../Structurel/Adaptateur.md)
