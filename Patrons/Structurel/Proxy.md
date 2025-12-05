# 🔗 Patron Proxy

> **Patron Structurel** - Fournit un substitut ou un placeholder pour un autre objet afin de contrôler l'accès à celui-ci.

[⬅️ Retour à l'Index](../../INDEX.md)

---

## 📋 Intention

Fournir un **substitut** ou un **placeholder** pour un autre objet afin de **contrôler l'accès** à celui-ci.

---

## 🎯 Problème Résolu

- Comment contrôler l'accès à un objet coûteux?
- Comment ajouter des fonctionnalités sans modifier l'objet?
- Comment différer la création d'un objet lourd?

---

## 📊 Diagramme UML

```
┌─────────────────────────────────────────────────────────────────┐
│                      PATRON PROXY                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌───────────┐        ┌──────────────────┐                     │
│   │  Client   │        │   «interface»    │                     │
│   └─────┬─────┘        │      Sujet       │                     │
│         │   Utilise    ├──────────────────┤                     │
│         └─────────────►│ +requete()       │                     │
│                        └────────┬─────────┘                     │
│                                 │                               │
│                    ┌────────────┴────────────┐                  │
│                    │                         │                  │
│           ┌────────▼─────────┐      ┌────────▼─────────┐        │
│           │      Proxy       │      │   SujetReel      │        │
│           ├──────────────────┤      ├──────────────────┤        │
│           │ -sujetReel       │─────►│ +requete()       │        │
│           ├──────────────────┤      │   // opération   │        │
│           │ +requete() {     │      │   // coûteuse    │        │
│           │   // avant       │      └──────────────────┘        │
│           │   sujetReel.     │                                  │
│           │     requete()    │                                  │
│           │   // après       │                                  │
│           │ }                │                                  │
│           └──────────────────┘                                  │
│                                                                 │
│   Le Proxy a la MÊME interface que le Sujet Réel                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔐 Types de Proxy

```
┌─────────────────────────────────────────────────────────────────┐
│                      TYPES DE PROXY                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   1. PROXY VIRTUEL (Lazy Loading)                               │
│      ┌──────────────────────────────────────────┐               │
│      │ Diffère la création de l'objet coûteux   │               │
│      │ jusqu'à ce qu'il soit vraiment nécessaire│               │
│      └──────────────────────────────────────────┘               │
│      Ex: Charger une image seulement à l'affichage              │
│                                                                 │
│   2. PROXY DE PROTECTION (Access Control)                       │
│      ┌─────────────────────────────────────────┐                │
│      │ Contrôle l'accès selon les permissions  │                │
│      └─────────────────────────────────────────┘                │
│      Ex: Vérifier les droits avant d'exécuter                   │
│                                                                 │
│   3. PROXY DISTANT (Remote Proxy)                               │
│      ┌─────────────────────────────────────────┐                │
│      │ Représente un objet dans un autre       │                │
│      │ espace d'adressage (réseau)             │                │
│      └─────────────────────────────────────────┘                │
│      Ex: RPC, Web Services                                      │
│                                                                 │
│   4. PROXY DE CACHE (Caching Proxy)                             │
│      ┌─────────────────────────────────────────┐                │
│      │ Met en cache les résultats pour éviter  │                │
│      │ des appels coûteux répétés              │                │
│      └─────────────────────────────────────────┘                │
│      Ex: Cache de requêtes BD                                   │
│                                                                 │
│   5. PROXY DE LOG (Logging Proxy)                               │
│      ┌─────────────────────────────────────────┐                │
│      │ Journalise les appels à l'objet réel    │                │
│      └─────────────────────────────────────────┘                │
│      Ex: Audit, debugging                                       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 💻 Exemple de Code Complet

### Interface Commune

```cpp
#include <iostream>
#include <string>
#include <memory>
#include <map>
#include <chrono>
#include <thread>

// Interface commune
class IImage {
public:
    virtual void afficher() = 0;
    virtual int getLargeur() const = 0;
    virtual int getHauteur() const = 0;
    virtual std::string getNom() const = 0;
    virtual ~IImage() = default;
};
```

### Sujet Réel (Image Lourde)

```cpp
// Sujet Réel: Image haute résolution (coûteuse à charger)
class ImageHauteResolution : public IImage {
private:
    std::string nomFichier;
    std::string donnees;  // Simule les données de l'image
    int largeur = 0;
    int hauteur = 0;
    
    void chargerDepuisDisque() {
        std::cout << "   ⏳ Chargement de " << nomFichier << " depuis le disque..." << std::endl;
        
        // Simule un chargement lent
        std::this_thread::sleep_for(std::chrono::milliseconds(500));
        
        // Simule les données
        largeur = 1920;
        hauteur = 1080;
        donnees = "<<données de l'image " + nomFichier + ">>";
        
        std::cout << "   ✓ Image chargée! (" << largeur << "x" << hauteur << ")" << std::endl;
    }
    
public:
    ImageHauteResolution(const std::string& fichier) : nomFichier(fichier) {
        chargerDepuisDisque();  // Chargement immédiat
    }
    
    void afficher() override {
        std::cout << "   🖼️  Affichage de " << nomFichier 
                  << " [" << largeur << "x" << hauteur << "]" << std::endl;
    }
    
    int getLargeur() const override { return largeur; }
    int getHauteur() const override { return hauteur; }
    std::string getNom() const override { return nomFichier; }
};
```

### Proxy Virtuel (Lazy Loading)

```cpp
// ═══════════════════════════════════════
// PROXY VIRTUEL (Lazy Loading)
// ═══════════════════════════════════════

class ProxyImageVirtuelle : public IImage {
private:
    std::string nomFichier;
    mutable std::unique_ptr<ImageHauteResolution> imageReelle;
    
    // Charge l'image seulement si nécessaire
    void chargerSiNecessaire() const {
        if (!imageReelle) {
            std::cout << "   📦 Proxy: Création de l'image réelle..." << std::endl;
            imageReelle = std::make_unique<ImageHauteResolution>(nomFichier);
        }
    }
    
public:
    ProxyImageVirtuelle(const std::string& fichier) : nomFichier(fichier) {
        std::cout << "   📋 Proxy créé pour: " << fichier << " (pas encore chargé)" << std::endl;
    }
    
    void afficher() override {
        chargerSiNecessaire();
        imageReelle->afficher();
    }
    
    int getLargeur() const override {
        chargerSiNecessaire();
        return imageReelle->getLargeur();
    }
    
    int getHauteur() const override {
        chargerSiNecessaire();
        return imageReelle->getHauteur();
    }
    
    std::string getNom() const override {
        return nomFichier;  // Pas besoin de charger pour le nom
    }
};
```

### Proxy de Protection

```cpp
// ═══════════════════════════════════════
// PROXY DE PROTECTION (Access Control)
// ═══════════════════════════════════════

enum class NiveauAcces { INVITE, UTILISATEUR, ADMIN };

class ProxyImageProtegee : public IImage {
private:
    std::unique_ptr<IImage> image;
    NiveauAcces niveauRequis;
    NiveauAcces niveauUtilisateur;
    
public:
    ProxyImageProtegee(std::unique_ptr<IImage> img, 
                       NiveauAcces requis, 
                       NiveauAcces utilisateur)
        : image(std::move(img)), niveauRequis(requis), niveauUtilisateur(utilisateur) {}
    
    void afficher() override {
        if (niveauUtilisateur >= niveauRequis) {
            std::cout << "   🔓 Accès autorisé" << std::endl;
            image->afficher();
        } else {
            std::cout << "   🔒 Accès refusé! Niveau insuffisant." << std::endl;
        }
    }
    
    int getLargeur() const override {
        if (niveauUtilisateur >= niveauRequis) {
            return image->getLargeur();
        }
        return 0;
    }
    
    int getHauteur() const override {
        if (niveauUtilisateur >= niveauRequis) {
            return image->getHauteur();
        }
        return 0;
    }
    
    std::string getNom() const override {
        return image->getNom();
    }
};
```

### Proxy de Cache

```cpp
// ═══════════════════════════════════════
// PROXY DE CACHE
// ═══════════════════════════════════════

class ProxyImageCache : public IImage {
private:
    static std::map<std::string, std::shared_ptr<ImageHauteResolution>> cache;
    std::string nomFichier;
    std::shared_ptr<ImageHauteResolution> image;
    
public:
    ProxyImageCache(const std::string& fichier) : nomFichier(fichier) {
        auto it = cache.find(fichier);
        if (it != cache.end()) {
            std::cout << "   💾 Cache HIT pour: " << fichier << std::endl;
            image = it->second;
        } else {
            std::cout << "   💾 Cache MISS pour: " << fichier << std::endl;
            image = std::make_shared<ImageHauteResolution>(fichier);
            cache[fichier] = image;
        }
    }
    
    void afficher() override { image->afficher(); }
    int getLargeur() const override { return image->getLargeur(); }
    int getHauteur() const override { return image->getHauteur(); }
    std::string getNom() const override { return image->getNom(); }
    
    static void viderCache() {
        std::cout << "   🗑️  Cache vidé" << std::endl;
        cache.clear();
    }
};

std::map<std::string, std::shared_ptr<ImageHauteResolution>> ProxyImageCache::cache;
```

### Proxy de Logging

```cpp
// ═══════════════════════════════════════
// PROXY DE LOGGING
// ═══════════════════════════════════════

class ProxyImageLog : public IImage {
private:
    std::unique_ptr<IImage> image;
    
    void log(const std::string& operation) const {
        auto now = std::chrono::system_clock::now();
        std::cout << "   📝 [LOG] " << operation << " sur " << image->getNom() << std::endl;
    }
    
public:
    ProxyImageLog(std::unique_ptr<IImage> img) : image(std::move(img)) {}
    
    void afficher() override {
        log("afficher()");
        image->afficher();
    }
    
    int getLargeur() const override {
        log("getLargeur()");
        return image->getLargeur();
    }
    
    int getHauteur() const override {
        log("getHauteur()");
        return image->getHauteur();
    }
    
    std::string getNom() const override {
        return image->getNom();
    }
};
```

### Utilisation

```cpp
int main() {
    std::cout << "╔═══════════════════════════════════════╗" << std::endl;
    std::cout << "║      PATRON PROXY - DEMO              ║" << std::endl;
    std::cout << "╚═══════════════════════════════════════╝" << std::endl;
    
    // ═══ PROXY VIRTUEL ═══
    std::cout << "\n=== Proxy Virtuel (Lazy Loading) ===" << std::endl;
    
    std::cout << "\nCréation des proxys (pas de chargement):" << std::endl;
    ProxyImageVirtuelle img1("photo_vacances.jpg");
    ProxyImageVirtuelle img2("photo_famille.jpg");
    ProxyImageVirtuelle img3("photo_travail.jpg");
    
    std::cout << "\nAccès au nom (pas de chargement):" << std::endl;
    std::cout << "   Nom: " << img1.getNom() << std::endl;
    
    std::cout << "\nAffichage (déclenche le chargement):" << std::endl;
    img1.afficher();
    
    std::cout << "\nDeuxième affichage (déjà chargé):" << std::endl;
    img1.afficher();
    
    // ═══ PROXY DE PROTECTION ═══
    std::cout << "\n=== Proxy de Protection ===" << std::endl;
    
    auto imageConfidentielle = std::make_unique<ProxyImageVirtuelle>("document_secret.jpg");
    
    std::cout << "\nAccès en tant qu'INVITE:" << std::endl;
    ProxyImageProtegee proxyInvite(
        std::make_unique<ProxyImageVirtuelle>("document_secret.jpg"),
        NiveauAcces::ADMIN,
        NiveauAcces::INVITE
    );
    proxyInvite.afficher();
    
    std::cout << "\nAccès en tant qu'ADMIN:" << std::endl;
    ProxyImageProtegee proxyAdmin(
        std::make_unique<ProxyImageVirtuelle>("document_secret.jpg"),
        NiveauAcces::ADMIN,
        NiveauAcces::ADMIN
    );
    proxyAdmin.afficher();
    
    // ═══ PROXY DE CACHE ═══
    std::cout << "\n=== Proxy de Cache ===" << std::endl;
    
    std::cout << "\nPremier accès (cache miss):" << std::endl;
    ProxyImageCache cache1("logo.png");
    
    std::cout << "\nDeuxième accès même fichier (cache hit):" << std::endl;
    ProxyImageCache cache2("logo.png");
    
    std::cout << "\nNouveau fichier (cache miss):" << std::endl;
    ProxyImageCache cache3("banner.png");
    
    return 0;
}
```

### Sortie

```
╔═══════════════════════════════════════╗
║      PATRON PROXY - DEMO              ║
╚═══════════════════════════════════════╝

=== Proxy Virtuel (Lazy Loading) ===

Création des proxys (pas de chargement):
   📋 Proxy créé pour: photo_vacances.jpg (pas encore chargé)
   📋 Proxy créé pour: photo_famille.jpg (pas encore chargé)
   📋 Proxy créé pour: photo_travail.jpg (pas encore chargé)

Accès au nom (pas de chargement):
   Nom: photo_vacances.jpg

Affichage (déclenche le chargement):
   📦 Proxy: Création de l'image réelle...
   ⏳ Chargement de photo_vacances.jpg depuis le disque...
   ✓ Image chargée! (1920x1080)
   🖼️  Affichage de photo_vacances.jpg [1920x1080]

Deuxième affichage (déjà chargé):
   🖼️  Affichage de photo_vacances.jpg [1920x1080]

=== Proxy de Protection ===

Accès en tant qu'INVITE:
   🔒 Accès refusé! Niveau insuffisant.

Accès en tant qu'ADMIN:
   🔓 Accès autorisé
   📦 Proxy: Création de l'image réelle...
   ...

=== Proxy de Cache ===

Premier accès (cache miss):
   💾 Cache MISS pour: logo.png
   ⏳ Chargement de logo.png depuis le disque...
   ✓ Image chargée! (1920x1080)

Deuxième accès même fichier (cache hit):
   💾 Cache HIT pour: logo.png

Nouveau fichier (cache miss):
   💾 Cache MISS pour: banner.png
   ...
```

---

## ✅ Avantages

| Avantage | Description |
|----------|-------------|
| **Contrôle** | Intercepte les accès à l'objet réel |
| **Lazy Loading** | Diffère la création coûteuse |
| **Transparent** | Même interface que l'objet réel |
| **Sécurité** | Peut ajouter contrôle d'accès |

---

## ❌ Inconvénients

| Inconvénient | Description |
|--------------|-------------|
| **Complexité** | Couche supplémentaire |
| **Performance** | Indirection à chaque appel |
| **Synchronisation** | Complexe en multi-thread |

---

## 🎯 Cas d'Utilisation

1. **Images** - Lazy loading de gros fichiers
2. **Base de données** - Connexion à la demande
3. **Services web** - Cache de requêtes
4. **Sécurité** - Contrôle d'accès
5. **Logging** - Audit des accès

---

## 🔗 Patrons Connexes

| Patron | Relation |
|--------|----------|
| [Décorateur](./Decorateur.md) | Ajoute fonctionnalité vs contrôle accès |
| [Adaptateur](./Adaptateur.md) | Change interface vs même interface |
| [Façade](./Facade.md) | Simplifie vs contrôle |

---

[⬅️ Retour à l'Index](../../INDEX.md) | [➡️ État](../Comportemental/Etat.md)
