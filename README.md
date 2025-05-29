# 📘 Documentation du module goblin-repositor

## Aperçu

Le module `goblin-repositor` est un gestionnaire de dépôts Debian pour l'écosystème Xcraft. Il permet de créer, gérer et servir des dépôts Debian personnalisés, facilitant ainsi la distribution de paquets logiciels dans un environnement contrôlé. Le module fournit un serveur HTTP intégré pour rendre les dépôts accessibles via le réseau.

## Sommaire

- [Structure du module](#structure-du-module)
- [Fonctionnement global](#fonctionnement-global)
- [Exemples d'utilisation](#exemples-dutilisation)
- [Interactions avec d'autres modules](#interactions-avec-dautres-modules)
- [Configuration avancée](#configuration-avancée)
- [Détails des sources](#détails-des-sources)
- [Utilisation du dépôt sur un système Debian](#utilisation-du-dépôt-sur-un-système-debian)
- [Fonctionnement interne](#fonctionnement-interne)

## Structure du module

- **Service principal** : Un goblin singleton qui gère les opérations sur les dépôts
- **Serveur HTTP** : Un serveur Express qui expose les dépôts Debian via HTTP
- **Utilitaires GPG** : Fonctionnalités pour générer et gérer les clés de signature des dépôts

## Fonctionnement global

Le module `goblin-repositor` fonctionne en trois étapes principales :

1. **Bootstrap** : Installation des dépendances nécessaires (gnupg, reprepro)
2. **Initialisation** : Création d'un nouveau dépôt Debian avec génération de clés GPG
3. **Publication** : Ajout de paquets Debian (.deb) au dépôt

Une fois configuré, le module peut démarrer un serveur HTTP qui expose les dépôts, permettant aux clients d'accéder aux paquets via un gestionnaire de paquets standard comme `apt`.

## Exemples d'utilisation

### Initialiser un nouveau dépôt Debian

```javascript
const repositor = this.quest.getAPI('repositor');
// Initialiser un dépôt pour une distribution spécifique
await repositor.initialize({distribution: 'myDistribution'});
```

### Publier un paquet dans le dépôt

```javascript
const repositor = this.quest.getAPI('repositor');
// Publier un fichier .deb spécifique
await repositor.publishDeb({
  distribution: 'myDistribution',
  packageDeb: '/path/to/package.deb',
});

// Publier tous les paquets d'une distribution Xcraft
await repositor.publishPackage({
  distribution: 'myDistribution',
  packageDistrib: 'sourceDistribution',
});
```

### Supprimer un paquet du dépôt

```javascript
const repositor = this.quest.getAPI('repositor');
// Supprimer un paquet par son nom
await repositor.removePackage({
  distribution: 'myDistribution',
  packageName: 'my-package',
});
```

## Interactions avec d'autres modules

- **[xcraft-core-etc]** : Pour charger la configuration du module
- **[xcraft-core-goblin]** : Fournit l'infrastructure d'acteurs
- **[xcraft-contrib-pacman]** : Pour accéder aux racines des paquets
- **[xcraft-core-env]** : Pour gérer les variables d'environnement
- **[xcraft-core-platform]** : Pour obtenir des informations sur l'architecture
- **[xcraft-core-fs]** : Pour les opérations sur le système de fichiers

## Configuration avancée

Le module peut être configuré via le fichier `config.js` :

| Option          | Description                                    | Type    | Valeur par défaut |
| --------------- | ---------------------------------------------- | ------- | ----------------- |
| `http.enabled`  | Activer le serveur HTTP pour les dépôts Debian | Boolean | `true`            |
| `http.port`     | Port du serveur HTTP pour les dépôts           | Number  | `23432`           |
| `http.hostname` | Nom d'hôte HTTP pour les dépôts                | String  | `0.0.0.0`         |
| `osRelease`     | Version de l'OS comme stretch, buster, etc.    | String  | ``                |

### Variables d'environnement

| Variable    | Description                       | Exemple             | Valeur par défaut |
| ----------- | --------------------------------- | ------------------- | ----------------- |
| `GNUPGHOME` | Répertoire contenant les clés GPG | `/home/user/.gnupg` | -                 |

## Détails des sources

### `repositor.js`

Fichier principal qui expose les commandes Xcraft du module.

### `lib/service.js`

Ce fichier définit le service Goblin principal qui gère les opérations sur les dépôts Debian. Il expose plusieurs quêtes pour initialiser, publier et gérer les paquets.

#### Méthodes publiques

- **`bootstrap()`** - Installe les dépendances nécessaires (gnupg, reprepro) pour le fonctionnement du module.

- **`initialize({distribution, $suite})`** - Initialise un nouveau dépôt Debian pour la distribution spécifiée. Génère une clé GPG pour signer les paquets et configure le dépôt.

  - `distribution` : Nom de la distribution à créer
  - `$suite` : Suite Debian (par défaut: 'stable')

- **`publishDeb({distribution, packageDeb})`** - Publie un fichier .deb spécifique dans le dépôt.

  - `distribution` : Nom de la distribution cible
  - `packageDeb` : Chemin vers le fichier .deb à publier

- **`publishPackage({distribution, packageDistrib})`** - Publie tous les paquets .deb d'une distribution Xcraft dans le dépôt spécifié.

  - `distribution` : Nom de la distribution cible
  - `packageDistrib` : Nom de la distribution source contenant les paquets

- **`removePackage({distribution, packageName})`** - Supprime un paquet du dépôt par son nom.

  - `distribution` : Nom de la distribution cible
  - `packageName` : Nom du paquet à supprimer

- **`_generateKey({distribution, repoDir})`** - Méthode interne pour générer une clé GPG pour la signature des paquets.

### `lib/debHttp.js`

Ce fichier définit la classe `DebHttp` qui implémente un serveur HTTP pour exposer les dépôts Debian. Il utilise Express pour servir les fichiers statiques des dépôts et Chokidar pour surveiller les changements dans les répertoires des dépôts.

#### Méthodes principales

- **`constructor(port, hostname)`** - Initialise le serveur HTTP avec le port et le nom d'hôte spécifiés.
- **`serve()`** - Démarre le serveur HTTP.
- **`dispose()`** - Arrête proprement le serveur HTTP et le watcher de fichiers.

### `lib/index.js`

Ce fichier exporte les fonctionnalités principales du module, notamment l'accès au serveur HTTP et la fonction de nettoyage.

#### Méthodes principales

- **`debHttp()`** - Crée et retourne une instance du serveur HTTP si activé dans la configuration.
- **`dispose()`** - Nettoie les ressources du module, notamment en arrêtant le serveur HTTP.

## Utilisation du dépôt sur un système Debian

Une fois le dépôt créé et les paquets publiés, vous pouvez l'utiliser sur un système Debian en ajoutant une entrée dans un fichier source.list :

```
deb [signed-by=/xcraft/var/prodroot.<distribution>/linux-amd64/var/deb/public.gpg.key] file:/xcraft/var/prodroot.<distribution>/linux-amd64/var/deb/ <distribution> non-free
```

Pour un dépôt servi via HTTP, utilisez :

```
deb [signed-by=/path/to/public.gpg.key] http://<hostname>:<port>/<distribution>/ <codename> non-free
```

## Fonctionnement interne

### Gestion des clés GPG

Le module génère automatiquement une paire de clés GPG pour chaque dépôt créé. La clé publique est exportée dans le répertoire du dépôt pour permettre aux clients de vérifier les signatures des paquets.

### Structure des répertoires

Les dépôts sont organisés selon la structure standard de reprepro :

```
/xcraft/var/prodroot.<distribution>/linux-<arch>/var/deb/
├── conf/
│   └── distributions
├── db/
├── dists/
│   └── <codename>/
├── pool/
│   └── non-free/
└── public.gpg.key
```

### Surveillance des dépôts

Le serveur HTTP surveille les répertoires des dépôts et met à jour automatiquement les routes lorsque de nouveaux dépôts sont créés. Il utilise Chokidar pour détecter les changements dans les répertoires et ajoute dynamiquement de nouvelles routes Express pour servir les nouveaux dépôts.

_Cette documentation a été mise à jour automatiquement._

[xcraft-core-etc]: https://github.com/Xcraft-Inc/xcraft-core-etc
[xcraft-core-goblin]: https://github.com/Xcraft-Inc/xcraft-core-goblin
[xcraft-contrib-pacman]: https://github.com/Xcraft-Inc/xcraft-contrib-pacman
[xcraft-core-env]: https://github.com/Xcraft-Inc/xcraft-core-env
[xcraft-core-platform]: https://github.com/Xcraft-Inc/xcraft-core-platform
[xcraft-core-fs]: https://github.com/Xcraft-Inc/xcraft-core-fs