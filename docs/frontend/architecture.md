# Architecture Frontend - MatchID

## Vue d'ensemble

MatchID utilise deux frontends distincts selon les besoins :
- **DataPrep Frontend** : Application Vue.js 3 avec Vite pour la préparation de données
- **Deces UI** : Application Svelte pour les services de décès et recherche publique

Cette architecture dual permet d'optimiser chaque interface pour son usage spécifique.

## Stack Technique

### DataPrep Frontend (Vue.js)

#### Framework Principal
- **Vue.js 3.2.25** : Framework JavaScript réactif
- **Vue Router 4.0.12** : Routage SPA
- **Vite 6.3.4** : Outil de build et serveur de développement

#### Bibliothèques UI et Composants
- **Bulma 0.6.2** : Framework CSS
- **Font Awesome 4.7.0** : Icônes
- **CodeMirror** : Éditeur de code YAML
- **Chart.js 2.9.4** : Graphiques et visualisations
- **D3.js 7.2.1** : Visualisations de données avancées

#### Utilitaires et Plugins
- **vue3-shortkey** : Raccourcis clavier
- **vue3-clipboard** : Gestion du presse-papiers
- **moment.js** : Manipulation des dates
- **crypto-js** : Fonctions cryptographiques
- **diff** : Comparaison de texte

### Deces UI (Svelte)

#### Framework Principal
- **Svelte** : Framework JavaScript compilé
- **Rollup** : Bundler et outil de build
- **Sirv** : Serveur de développement

#### Bibliothèques et Dépendances
- **@elastic/elasticsearch 8.18.2** : Client Elasticsearch moderne
- **bullmq 5.21.2** : Gestion des files d'attente Redis
- **ioredis 5.3.2** : Client Redis optimisé
- **helmet 7.0.0** : Sécurité HTTP
- **cors 2.8.5** : Gestion CORS
- **express-rate-limit 6.10.0** : Rate limiting
- **winston 3.10.0** : Logging avancé
- **joi 17.10.1** : Validation de schémas

## Architecture des Composants

### Structure des Routes

```javascript
/matchID/                           # Racine de l'application
├── login                          # Authentification
├── projects                       # Liste des projets (Home)
├── jobs                          # Gestion des tâches
├── projects/:project             # Vue projet
├── projects/:project/recipes/:recipe        # Éditeur de recettes
├── projects/:project/datasets/:dataset     # Éditeur de datasets
└── projects/:project/datasets/:dataset/validation  # Validation
```

### Composants Principaux

#### 1. Home.vue - Gestionnaire de Projets
**Localisation** : [`src/components/Home.vue`](../../packages/dataprep-frontend/src/components/Home.vue)

**Fonctionnalités** :
- Affichage de la liste des projets
- Création de nouveaux projets
- Navigation vers les projets existants

**Communication** :
- Écoute l'événement `reloadProjects` via le bus d'événements
- Émet `reloadNav` pour rafraîchir la navigation

#### 2. Project.vue - Vue Projet
**Localisation** : [`src/components/Project.vue`](../../packages/dataprep-frontend/src/components/Project.vue)

**Fonctionnalités** :
- Gestion des datasets (upload, elasticsearch)
- Gestion des recettes de transformation
- Import et création d'objets
- Suppression de projets

**Sections** :
- **Datasets** : Connecteurs upload et elasticsearch
- **Recipes** : Recettes de transformation de données

#### 3. Recipe.vue - Éditeur de Recettes
**Localisation** : [`src/components/Recipe.vue`](../../packages/dataprep-frontend/src/components/Recipe.vue)

**Fonctionnalités** :
- Éditeur YAML pour les recettes
- Prévisualisation des données transformées
- Visualisation des logs d'exécution
- Modes d'affichage (compact, étendu, plein écran)

**API Endpoints** :
- `GET /api/v0/recipes/{recipe}` : Métadonnées de la recette
- `GET /api/v0/recipes/{recipe}/yaml` : Code YAML
- `PUT /api/v0/recipes/{recipe}/test` : Test d'exécution
- `POST /api/v0/conf/{project}/{source}` : Sauvegarde

#### 4. Dataset.vue - Éditeur de Datasets
**Localisation** : [`src/components/Dataset.vue`](../../packages/dataprep-frontend/src/components/Dataset.vue)

**Fonctionnalités** :
- Configuration des sources de données
- Éditeur YAML pour les datasets
- Prévisualisation des données
- Validation de la configuration

**API Endpoints** :
- `GET /api/v0/datasets/{dataset}` : Métadonnées du dataset
- `GET /api/v0/datasets/{dataset}/yaml` : Configuration YAML
- `POST /api/v0/datasets/{dataset}/` : Prévisualisation des données

#### 5. Validation/Controller.vue - Interface de Validation
**Localisation** : [`src/components/Validation/Controller.vue`](../../packages/dataprep-frontend/src/components/Validation/Controller.vue)

**Fonctionnalités** :
- Validation manuelle des appariements
- Statistiques de performance
- Interface de décision (positif/négatif/indécis)
- Raccourcis clavier pour la productivité

### Composants Utilitaires

#### Éditeurs
- **YamlEditor.vue** : Éditeur CodeMirror pour YAML
- **Shortcuts.vue** : Aide contextuelle pour les raccourcis

#### Visualisation
- **DataViewer.vue** : Tableau de données avec filtrage
- **LogsViewer.vue** : Affichage des logs d'exécution
- **Graph.vue** : Visualisations graphiques

#### Interface
- **Navigation.vue** : Barre de navigation principale
- **Message.vue** : Notifications système
- **ProgressBar.vue** : Indicateurs de progression

#### Gestion d'Objets
- **Object/New.vue** : Création d'objets (projets, datasets, recettes)
- **Object/Import.vue** : Import de datasets
- **Object/Delete.vue** : Suppression d'objets

## Configuration et Internationalisation

### Configuration API
**Fichier** : [`src/assets/json/backend.json`](../../packages/dataprep-frontend/src/assets/json/backend.json)
```json
{
    "api": {
        "url": "/matchID/api/v0/"
    }
}
```

### Localisation
**Fichier** : [`src/assets/json/lang.json`](../../packages/dataprep-frontend/src/assets/json/lang.json)

**Langues supportées** :
- Français (par défaut)
- Anglais

**Sections traduites** :
- Interface générale
- Navigation
- Validation
- Éditeurs
- Gestion d'objets
- Authentification

## Communication et État

### Bus d'Événements
**Fichier** : [`src/eventBus.js`](../../packages/dataprep-frontend/src/eventBus.js)

**Événements principaux** :
- `reloadNav` : Rafraîchissement de la navigation
- `reloadProjects` : Mise à jour de la liste des projets
- `reloadDatasets` : Mise à jour des datasets
- `reloadRecipes` : Mise à jour des recettes
- `message` : Notifications système
- `deleteObject` : Suppression d'objets
- `langChange` : Changement de langue

### Gestion d'État Global
L'application utilise un mixin global pour partager :
- `apiUrl` : URL de base de l'API
- `localization` : Données de localisation
- `lang` : Langue courante

## Fonctionnalités Avancées

### Raccourcis Clavier
- **Ctrl+S** : Sauvegarde dans les éditeurs
- **Navigation** : Flèches pour la validation
- **Décisions** : Touches numériques pour les choix

### Modes d'Affichage
Les éditeurs supportent trois modes :
1. **Compact** (mode 0) : Éditeur réduit
2. **Étendu** (mode 1) : Éditeur agrandi
3. **Plein écran** (mode 2) : Éditeur en plein écran

### Validation en Temps Réel
- Validation YAML en temps réel
- Prévisualisation des données
- Feedback visuel des erreurs

## Intégrations Backend

### DataPrep Frontend - Endpoints API
- `/api/v0/projects` : Gestion des projets
- `/api/v0/datasets` : Gestion des datasets
- `/api/v0/recipes` : Gestion des recettes
- `/api/v0/conf` : Configuration et sauvegarde
- `/api/v0/jobs` : Gestion des tâches

### Deces UI - Endpoints API
- `/api/v1/search` : Recherche de personnes décédées
- `/api/v1/validation` : Validation d'appariements
- `/api/v1/stats` : Statistiques et métriques
- `/api/v1/bulk` : Traitement en lot

### Services Backend Intégrés
- **dataprep-backend** (Python/Flask) : Préparation de données
- **deces-backend** (Node.js/TypeScript) : Services de décès
- **Elasticsearch 8.6.1** : Moteur de recherche
- **PostgreSQL 13 + cstore** : Stockage relationnel et analytique
- **Redis alpine** : Cache et files d'attente BullMQ

### Authentification et Sécurité
- Support de l'authentification par token JWT
- Variables BACKEND_TOKEN_* pour la sécurité
- Rate limiting configuré par endpoint
- Intégration avec les providers OAuth (GitHub, Facebook, Twitter)

### Gestion des Erreurs
- Notifications système pour les erreurs
- Retry automatique pour les requêtes échouées
- Feedback visuel des états de chargement
- Logging centralisé avec Winston

---

*Cette architecture modulaire permet une maintenance aisée et une extensibilité pour de nouvelles fonctionnalités.*