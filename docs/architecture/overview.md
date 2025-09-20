# Architecture Globale - MatchID

## Vue d'ensemble du Système

MatchID est une plateforme de préparation et d'appariement de données composée de plusieurs services interconnectés. Le projet utilise une architecture basée sur des conteneurs Docker avec une séparation claire entre le frontend, les services backend, et les outils d'infrastructure.

## Composants Principaux

### 1. Frontend - DataPrep Frontend
- **Technologie** : Vue.js 3 avec Vite
- **Port** : 8081 (développement)
- **Localisation** : [`packages/dataprep-frontend/`](../../packages/dataprep-frontend/)
- **Description** : Interface utilisateur pour la préparation et la validation des données

#### Caractéristiques techniques :
- Framework : Vue.js 3.2.25
- Build tool : Vite 6.3.4
- UI Framework : Bulma 0.6.2
- Graphiques : Chart.js 2.9.4, D3.js 7.2.1
- Éditeur de code : CodeMirror (codemirror-editor-vue3)

### 2. Services d'Infrastructure

#### Nginx (Proxy Reverse)
- **Rôle** : Proxy inverse et serveur web
- **Configuration** : [`packages/dataprep-frontend/nginx/`](../../packages/dataprep-frontend/nginx/)
- **Ports** : 
  - Production : Variable `${PORT}`
  - Développement : `${PORT}` + 35729 (LiveReload)

#### Elasticsearch
- **Rôle** : Moteur de recherche et d'indexation
- **Configuration** : Via variables d'environnement
- **Accès** : `${ES_HOST}:${ES_PORT}/${ES_INDEX}`

### 3. Outils et Utilitaires

#### Tools Package
- **Localisation** : [`packages/tools/`](../../packages/tools/) et [`tools/`](../../tools/)
- **Rôle** : Scripts d'automatisation, déploiement cloud, et gestion des données
- **Technologies** : 
  - Makefile complexe pour l'orchestration
  - Support multi-cloud (SCW, AWS, OpenStack)
  - Intégration avec des services de stockage (S3, Swift, rclone)

## Architecture de Déploiement

### Environnements

#### Développement Local
- **Configuration** : [`docker-compose-dev.yml`](../../docker-compose-dev.yml)
- **Services** :
  - `nginx-development` : Proxy avec hot-reload
  - `frontend-development` : Serveur de développement Vite

#### Tests
- **Configuration** : [`docker-compose-test.yml`](../../docker-compose-test.yml)
- **Services** :
  - `ui-test` : Tests automatisés avec Playwright

### Variables d'Environnement Clés

#### Frontend
```bash
APP=${APP}                              # Nom de l'application
PORT=${FRONTEND_DEV_PORT}              # Port du serveur de développement
HOST=0.0.0.0                          # Interface d'écoute
```

#### Backend et API
```bash
BACKEND_PROXY_PATH=${BACKEND_PROXY_PATH}    # Chemin du proxy backend
BACKEND_HOST=${BACKEND_HOST}                # Hôte du backend
BACKEND_PORT=${BACKEND_PORT}                # Port du backend
BACKEND_TOKEN_USER=${BACKEND_TOKEN_USER}    # Token d'authentification
```

#### Elasticsearch
```bash
ES_PROXY_PATH=${ES_PROXY_PATH}         # Chemin du proxy Elasticsearch
ES_HOST=${ES_HOST}                     # Hôte Elasticsearch
ES_PORT=${ES_PORT}                     # Port Elasticsearch
ES_INDEX=${ES_INDEX}                   # Index Elasticsearch
ES_MAX_RESULTS=${ES_MAX_RESULTS}       # Limite de résultats
```

#### Intégrations Externes
```bash
DATAGOUV_PROXY_PATH=${DATAGOUV_PROXY_PATH}           # Proxy Data.gouv.fr
DATAGOUV_CATALOG_URL=${DATAGOUV_CATALOG_URL}         # URL du catalogue
DATAGOUV_RESOURCES_URL=${DATAGOUV_RESOURCES_URL}     # URL des ressources
```

## Flux de Données

### 1. Ingestion de Données
- Source principale : Data.gouv.fr (fichiers de décès)
- Traitement : Téléchargement, validation SHA1, conversion d'encodage
- Stockage : Compression gzip, stockage cloud (S3/Swift)

### 2. Traitement et Indexation
- Pipeline de transformation des données
- Indexation dans Elasticsearch
- Génération de statistiques et métriques

### 3. Interface Utilisateur
- Visualisation des données via le frontend Vue.js
- Outils de validation et d'appariement
- Export et téléchargement des résultats

## Sécurité et Authentification

### Limitation de Taux (Rate Limiting)
Configuration Nginx avec différents seuils :
- `API_SEARCH_LIMIT_RATE` : Recherches
- `API_BULK_SUBMIT_LIMIT_RATE` : Soumissions en lot
- `API_MISC_LIMIT_RATE` : API diverses
- `API_AGG_LIMIT_RATE` : Agrégations
- `API_DOWNLOAD_LIMIT_RATE` : Téléchargements

### Authentification
- Token utilisateur : `BACKEND_TOKEN_USER`
- Scope API : `API_USER_SCOPE`

## Monitoring et Observabilité

### Logs
- Fluent Bit pour la collecte de logs
- Intégration New Relic (optionnelle)
- Logs Docker centralisés

### Métriques
- Monitoring des performances API
- Statistiques d'utilisation
- Alertes Slack configurables

## Déploiement Cloud

### Providers Supportés
1. **Scaleway (SCW)** - Configuration par défaut
2. **AWS EC2** - Support complet
3. **OpenStack** - Support OVH et autres

### Fonctionnalités
- Provisioning automatique d'instances
- Configuration réseau (VPC, réseaux privés)
- Snapshots et images personnalisées
- Load balancing avec Nginx
- Déploiement multi-instances

## Intégrations Externes

### Data.gouv.fr
- API : `https://www.data.gouv.fr/api/1/datasets`
- Dataset principal : Service public - Annuaire de l'administration
- Synchronisation automatique des fichiers

### Services de Stockage
- **AWS S3** : Stockage principal
- **Swift** : Alternative OpenStack
- **rclone** : Outil de synchronisation multi-cloud

### CDN et Cache
- Cloudflare (optionnel)
- Purge automatique du cache

---

*Cette architecture permet une scalabilité horizontale et une haute disponibilité grâce à la containerisation et au support multi-cloud.*