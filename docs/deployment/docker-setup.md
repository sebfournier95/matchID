# Configuration Docker - MatchID

## Vue d'ensemble

MatchID utilise une architecture Docker multi-étapes avec des configurations distinctes pour le développement et la production. Le système est composé de conteneurs Nginx et Node.js orchestrés via Docker Compose.

## Architecture des Conteneurs

### Structure Multi-étapes

#### 1. Image Frontend (Node.js)
**Dockerfile** : [`packages/dataprep-frontend/Dockerfile`](../../packages/dataprep-frontend/Dockerfile)

**Étapes de build** :
- **base** : Installation des dépendances Node.js
- **development** : Serveur de développement Vite
- **build** : Build de production

**Image de base** : `node:20-alpine`

#### 2. Image Nginx
**Dockerfile** : [`packages/dataprep-frontend/nginx/Dockerfile`](../../packages/dataprep-frontend/nginx/Dockerfile)

**Étapes de build** :
- **base** : Configuration Nginx de base
- **development** : Proxy pour le développement
- **production** : Serveur statique optimisé

**Image de base** : `nginx:1.27.5-alpine`

#### 3. Images de Services Personnalisées

**PostgreSQL avec cstore** :
- **Image personnalisée** : `matchid/postgres_cstore`
- **Base** : `postgres:13`
- **Extensions** : cstore_fdw pour le stockage colonnaire
- **Optimisations** : Configuration pour les gros volumes de données

**Elasticsearch avec phonetic** :
- **Image personnalisée** : `matchid/elasticsearch-phonetic:8.6.1`
- **Base** : `elasticsearch:8.6.1`
- **Plugins** : analysis-phonetic pour la recherche phonétique
- **Configuration** : Optimisée pour l'appariement de données

**Redis** :
- **Image** : `redis:alpine`
- **Usage** : Cache et files d'attente (BullMQ)

## Configurations d'Environnement

### Développement Local

**Fichier** : [`docker-compose-dev.yml`](../../docker-compose-dev.yml)

#### Services

##### nginx-development
```yaml
services:
  nginx-development:
    image: ${DOCKER_USERNAME}/${DC_PREFIX}-nginx-development:${APP_VERSION}
    container_name: ${DC_PREFIX}-nginx-development
    ports:
      - ${PORT}:80
      - "35729:35729"  # LiveReload
    depends_on:
      - frontend-development
```

**Fonctionnalités** :
- Proxy inverse vers le serveur de développement
- Support du hot-reload
- Rate limiting configurable
- Routage API vers le backend

##### frontend-development
```yaml
services:
  frontend-development:
    build:
      context: ${FRONTEND}
      target: development
    container_name: ${DC_PREFIX}-frontend-development
    environment:
      APP: ${APP_NAME}
    volumes:
      - ${FRONTEND}/src:/${APP_NAME}/src
      - ${FRONTEND}/package.json:/${APP_NAME}/package.json
```

**Fonctionnalités** :
- Serveur Vite en mode développement
- Hot Module Replacement (HMR)
- Volumes montés pour le développement en temps réel

### Production

**Fichier** : [`packages/dataprep-frontend/docker-compose.yml`](../../packages/dataprep-frontend/docker-compose.yml)

#### Services Complets
```yaml
services:
  # Frontend Nginx
  nginx:
    image: ${DOCKER_USERNAME}/${DC_PREFIX}-${APP}:${APP_VERSION}
    
  # PostgreSQL avec cstore
  postgres:
    image: matchid/postgres_cstore:13
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    
  # Elasticsearch avec phonetic
  elasticsearch:
    image: matchid/elasticsearch-phonetic:8.6.1
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    
  # Redis pour les files d'attente
  redis:
    image: redis:alpine
    command: redis-server --appendonly yes
```

#### Service nginx
```yaml
services:
  nginx:
    image: ${DOCKER_USERNAME}/${DC_PREFIX}-${APP}:${APP_VERSION}
    build:
      context: ${NGINX}
      target: production
    container_name: ${DC_PREFIX}-${APP}
    ports:
      - ${PORT}:80
```

**Fonctionnalités** :
- Serveur de fichiers statiques optimisé
- Compression gzip
- Cache headers optimisés
- Rate limiting de production

### Tests

**Fichier** : [`docker-compose-test.yml`](../../docker-compose-test.yml)

#### Service ui-test
```yaml
services:
  ui-test:
    image: mcr.microsoft.com/playwright:v${PLAYWRIGHT_VERSION}
    container_name: ${DC_PREFIX}
    working_dir: /ui-test
    volumes:
      - ${FRONTEND}/ui-test/:/ui-test
    command: >
      bash -c "mkdir -p /ui-test/node_modules && 
               chown -R pwuser:pwuser /ui-test/node_modules && 
               yarn install && 
               su pwuser -c 'yarn test'"
```

## Configuration Nginx

### Développement

**Template** : [`packages/dataprep-frontend/nginx/default-dev.template`](../../packages/dataprep-frontend/nginx/default-dev.template)

#### Upstreams
```nginx
upstream frontend-dev {
  server <FRONTEND_DEV_HOST>:<FRONTEND_DEV_PORT>;
}

upstream backend {
  server <BACKEND_HOST>:<BACKEND_PORT>;
}
```

#### Routage
- **/** : Redirection vers `/<APP_NAME>/`
- **`<API_PATH>/*`** : Proxy vers le backend avec rate limiting
- **`/<APP_NAME>/*`** : Proxy vers le serveur de développement Vite

#### Rate Limiting
```nginx
limit_req zone=api burst=<API_USER_BURST>;
limit_req zone=server burst=<API_GLOBAL_BURST>;
limit_req_status 429;
```

### Production

**Template** : [`packages/dataprep-frontend/nginx/default-run.template`](../../packages/dataprep-frontend/nginx/default-run.template)

#### Optimisations
- Compression gzip activée
- Cache headers pour les assets statiques
- Sécurité headers
- Rate limiting optimisé pour la production

## Nouveaux Fichiers de Configuration

### Fichier .env
**Localisation** : [`.env`](../../.env)
**Usage** : Variables d'environnement pour Docker Compose
```bash
# Versions des services
ES_VERSION=8.6.1
POSTGRES_VERSION=13
REDIS_VERSION=alpine

# Configuration réseau
DC_NETWORK=matchid
DC_PREFIX=matchid
PORT=8080
```

### Fichier artifacts
**Localisation** : [`artifacts`](../../artifacts)
**Usage** : Configuration d'environnement et déploiement
- Variables de build et runtime
- Configuration cloud multi-provider
- Tokens et secrets

### Répertoire config/
**Localisation** : [`config/`](../../config/)
**Usage** : Fichiers de configuration des services
- Configuration Elasticsearch
- Configuration PostgreSQL
- Configuration Nginx
- Templates de déploiement

## Variables d'Environnement

### Variables de Build

#### Frontend
```bash
# Proxy et registries
http_proxy=${http_proxy}
https_proxy=${https_proxy}
no_proxy=${no_proxy}
npm_registry=${NPM_REGISTRY}
sass_registry=${SASS_REGISTRY}

# Configuration NPM
NPM_FIX=${NPM_FIX}                    # Correction automatique des vulnérabilités
NPM_LATEST=${NPM_LATEST}              # Utilisation de la dernière version NPM
NPM_VERBOSE=${NPM_VERBOSE}            # Mode verbose
NPM_AUDIT_IGNORE=${NPM_AUDIT_IGNORE}  # Ignorer l'audit de sécurité

# Application
app_path=/${APP_NAME}
app_name=${APP_NAME}
app_ver=${APP_VERSION}
```

#### Nginx
```bash
# Application
APP_NAME=${APP_NAME}
API_PATH=${API_PATH}

# Backend
BACKEND_HOST=${BACKEND_HOST}
BACKEND_PORT=${BACKEND_PORT}

# Frontend (développement)
FRONTEND_DEV_HOST=${FRONTEND_DEV_HOST}
FRONTEND_DEV_PORT=${FRONTEND_DEV_PORT}
```

### Variables de Runtime

#### Rate Limiting
```bash
# Limites par utilisateur
API_USER_LIMIT_RATE=${API_USER_LIMIT_RATE}
API_USER_BURST=${API_USER_BURST}
API_USER_SCOPE=${API_USER_SCOPE}

# Limites globales
API_GLOBAL_LIMIT_RATE=${API_GLOBAL_LIMIT_RATE}
API_GLOBAL_BURST=${API_GLOBAL_BURST}

# Téléchargements
API_DOWNLOAD_LIMIT_RATE=${API_DOWNLOAD_LIMIT_RATE}
```

#### Réseau
```bash
# Ports
PORT=${PORT}                          # Port d'exposition principal
FRONTEND_DEV_PORT=${FRONTEND_DEV_PORT} # Port du serveur de développement

# Réseau Docker
DC_NETWORK=${DC_NETWORK}              # Réseau Docker externe

# Services
ES_VERSION=${ES_VERSION}              # Version Elasticsearch (8.6.1)
POSTGRES_VERSION=${POSTGRES_VERSION}  # Version PostgreSQL (13)
REDIS_VERSION=${REDIS_VERSION}        # Version Redis (alpine)
```

## Processus de Build

### Développement

1. **Build de l'image Nginx de développement**
   ```bash
   docker-compose -f docker-compose-dev.yml build nginx-development
   ```

2. **Build de l'image Frontend de développement**
   ```bash
   docker-compose -f docker-compose-dev.yml build frontend-development
   ```

3. **Démarrage des services**
   ```bash
   docker-compose -f docker-compose-dev.yml up -d
   ```

### Production

1. **Build de l'application**
   ```bash
   # Dans le conteneur de build
   npm run build
   ```

2. **Création de l'archive de distribution**
   ```bash
   tar -czf ${app_pkg}-${app_ver}-dist.tar.gz dist/
   ```

3. **Build de l'image Nginx de production**
   ```bash
   docker-compose build nginx
   ```

### Tests

1. **Préparation de l'environnement de test**
   ```bash
   docker-compose -f docker-compose-test.yml build
   ```

2. **Exécution des tests**
   ```bash
   docker-compose -f docker-compose-test.yml run ui-test
   ```

## Volumes et Persistance

### Développement
```yaml
volumes:
  # Code source (hot-reload)
  - ${FRONTEND}/src:/${APP_NAME}/src
  - ${FRONTEND}/package.json:/${APP_NAME}/package.json
  - ${FRONTEND}/vite.config.js:/${APP_NAME}/vite.config.js
  
  # Configuration Nginx
  - ${NGINX}/default-dev.template:/etc/nginx/conf.d/default.template
  - ${NGINX}/nginx-dev.template:/etc/nginx/nginx.template
```

### Production
```yaml
volumes:
  # Logs Nginx
  - /var/log/nginx
```

### Tests
```yaml
volumes:
  # Tests et node_modules
  - ${FRONTEND}/ui-test/:/ui-test
  - ui-test-node-modules:/ui-test/node_modules
```

## Sécurité

### Rate Limiting
- Limitation par zone (api, server, app)
- Burst configurable par type d'endpoint
- Status 429 pour les dépassements

### Headers de Sécurité
- CORS configuré pour les API
- Headers de forwarding pour les proxies
- Support HTTPS avec headers appropriés

### Isolation
- Conteneurs isolés par réseau Docker
- Utilisateurs non-root dans les conteneurs
- Volumes read-only quand possible

## Monitoring et Logs

### Logs Nginx
- Logs d'accès et d'erreur
- Format personnalisable
- Rotation automatique

### Logs Application
- Logs Node.js via stdout/stderr
- Intégration avec les drivers de logging Docker

### Health Checks
- Endpoints de santé configurables
- Monitoring des upstreams
- Alertes sur les échecs

## Diagramme des Interactions entre Services

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client Web    │────│  Nginx Proxy    │────│  Frontend App   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  Backend APIs   │
                       │ (dataprep/deces)│
                       └─────────────────┘
                                │
                    ┌───────────┼───────────┐
                    ▼           ▼           ▼
            ┌─────────────┐ ┌─────────┐ ┌─────────┐
            │Elasticsearch│ │PostgreSQL│ │  Redis  │
            │   8.6.1     │ │+ cstore │ │ alpine  │
            │ + phonetic  │ │   v13   │ │(BullMQ) │
            └─────────────┘ └─────────┘ └─────────┘
```

### Flux de Données
1. **Client** → Nginx (port 8080)
2. **Nginx** → Frontend (Svelte/Vue.js)
3. **Frontend** → Backend APIs (Python/Node.js)
4. **Backend** → Services de données :
   - **Elasticsearch** : Recherche et indexation
   - **PostgreSQL** : Stockage relationnel avec cstore
   - **Redis** : Cache et files d'attente BullMQ

### Ports et Réseaux
- **8080** : Exposition publique (Nginx)
- **8081** : Frontend développement (Vite)
- **5000** : Backend dataprep (Python/Flask)
- **3000** : Backend deces (Node.js/Express)
- **9200** : Elasticsearch
- **5432** : PostgreSQL
- **6379** : Redis

---

*Cette configuration Docker permet un déploiement flexible et scalable avec une séparation claire entre les environnements et une architecture de services complète.*