# Guide de Démarrage Rapide - MatchID

## Prérequis

### Logiciels Requis
- **Docker** >= 20.10
- **Docker Compose** >= 1.27
- **Git**
- **Make** (pour les scripts d'automatisation)

### Système d'Exploitation
- Linux (Ubuntu/Debian recommandé)
- macOS
- Windows avec WSL2

## Installation Rapide

### 1. Clonage du Projet

```bash
git clone https://github.com/matchid-project/matchID.git
cd matchID
```

### 2. Configuration de Base

Créez le fichier de configuration local :

```bash
# Copiez le fichier d'exemple
cp artifacts.example artifacts

# Éditez la configuration
nano artifacts
```

**Configuration minimale** :
```bash
# Configuration de base
export APP_GROUP=matchID
export APP=dataprep-frontend
export APP_VERSION=dev
export PORT=8080

# Docker
export DOCKER_USERNAME=matchid
export DC_PREFIX=matchid
export DC_NETWORK=matchid

# Frontend
export FRONTEND_DEV_PORT=8081
export FRONTEND_DEV_HOST=frontend-development

# Backend (à adapter selon votre configuration)
export BACKEND_HOST=backend
export BACKEND_PORT=5000
export BACKEND_PROXY_PATH=/matchID/api/v0

# Elasticsearch (optionnel pour le développement)
export ES_HOST=elasticsearch
export ES_PORT=9200
export ES_INDEX=matchid
```

### 3. Démarrage en Mode Développement

#### Option A : Démarrage Complet (Recommandé)

```bash
# Démarrage de tous les services
make docker-compose-dev-up

# Ou directement avec docker-compose
docker-compose -f docker-compose-dev.yml up -d
```

#### Option B : Démarrage Frontend Uniquement

```bash
# Démarrage du frontend seul
cd packages/dataprep-frontend
docker-compose -f docker-compose-dev.yml up frontend-development
```

### 4. Vérification de l'Installation

Ouvrez votre navigateur et accédez à :
- **Frontend** : http://localhost:8080/matchID/
- **API** : http://localhost:8080/matchID/api/v0/ (si backend configuré)

## Développement Local

### Structure des Répertoires

```
matchID/
├── packages/
│   ├── dataprep-frontend/     # Frontend Vue.js
│   ├── tools/                 # Outils et scripts
│   └── website/               # Site web de documentation
├── tools/                     # Outils de déploiement
├── docker-compose-dev.yml     # Configuration développement
├── docker-compose-test.yml    # Configuration tests
└── artifacts                  # Configuration locale
```

### Commandes de Développement

#### Frontend

```bash
cd packages/dataprep-frontend

# Installation des dépendances
npm install

# Démarrage en mode développement
npm run dev

# Build de production
npm run build

# Prévisualisation du build
npm run preview
```

#### Docker

```bash
# Build des images de développement
docker-compose -f docker-compose-dev.yml build

# Démarrage des services
docker-compose -f docker-compose-dev.yml up -d

# Arrêt des services
docker-compose -f docker-compose-dev.yml down

# Logs des services
docker-compose -f docker-compose-dev.yml logs -f

# Redémarrage d'un service
docker-compose -f docker-compose-dev.yml restart frontend-development
```

### Hot Reload et Développement

Le système est configuré pour le hot reload automatique :

1. **Frontend** : Les modifications dans `packages/dataprep-frontend/src/` sont automatiquement rechargées
2. **Configuration Nginx** : Les templates sont montés en volume pour modification à chaud
3. **LiveReload** : Port 35729 exposé pour le rechargement automatique du navigateur

### Variables d'Environnement de Développement

#### Frontend
```bash
# Dans le conteneur frontend-development
APP=dataprep-frontend
PORT=8081
HOST=0.0.0.0

# Configuration API
API_EMAIL=contact@matchid.io
API_MAX_BODY=100M
ES_PROXY_PATH=/matchID/api/v0/es
ES_MAX_RESULTS=1000
BACKEND_TOKEN_USER=dev-token
BACKEND_PROXY_PATH=/matchID/api/v0
```

#### Nginx
```bash
# Configuration du proxy
FRONTEND_DEV_HOST=frontend-development
FRONTEND_DEV_PORT=8081
BACKEND_HOST=backend
BACKEND_PORT=5000

# Rate limiting (développement)
API_USER_LIMIT_RATE=10r/s
API_USER_BURST=20
API_GLOBAL_LIMIT_RATE=100r/s
API_GLOBAL_BURST=200
```

## Tests

### Tests Unitaires et d'Intégration

```bash
# Démarrage de l'environnement de test
docker-compose -f docker-compose-test.yml up --build

# Tests Playwright
cd packages/dataprep-frontend
npm run test:e2e
```

### Tests Manuels

1. **Interface** : Vérifiez que l'interface se charge correctement
2. **Navigation** : Testez la navigation entre les différentes sections
3. **API** : Vérifiez les appels API dans les outils de développement du navigateur

## Dépannage

### Problèmes Courants

#### Port déjà utilisé
```bash
# Vérifier les ports utilisés
netstat -tulpn | grep :8080

# Changer le port dans artifacts
export PORT=8081
```

#### Problèmes de permissions Docker
```bash
# Ajouter l'utilisateur au groupe docker
sudo usermod -aG docker $USER

# Redémarrer la session
newgrp docker
```

#### Erreurs de build NPM
```bash
# Nettoyer le cache NPM
docker-compose -f docker-compose-dev.yml exec frontend-development npm cache clean --force

# Reconstruire l'image
docker-compose -f docker-compose-dev.yml build --no-cache frontend-development
```

#### Problèmes de proxy
```bash
# Configuration proxy dans artifacts
export http_proxy=http://proxy.company.com:8080
export https_proxy=http://proxy.company.com:8080
export no_proxy=localhost,127.0.0.1,*.local
```

### Logs et Debugging

```bash
# Logs détaillés
docker-compose -f docker-compose-dev.yml logs -f --tail=100

# Logs d'un service spécifique
docker-compose -f docker-compose-dev.yml logs -f frontend-development

# Accès au conteneur pour debugging
docker-compose -f docker-compose-dev.yml exec frontend-development sh
```

### Réinitialisation Complète

```bash
# Arrêt et suppression des conteneurs
docker-compose -f docker-compose-dev.yml down -v

# Suppression des images
docker rmi $(docker images "matchid/*" -q)

# Nettoyage Docker
docker system prune -f

# Redémarrage complet
docker-compose -f docker-compose-dev.yml up --build -d
```

## Workflow de Développement

### 1. Développement de Fonctionnalités

```bash
# Créer une branche
git checkout -b feature/nouvelle-fonctionnalite

# Développer avec hot reload
# Les modifications sont automatiquement rechargées

# Tester localement
npm run test

# Commit et push
git add .
git commit -m "feat: nouvelle fonctionnalité"
git push origin feature/nouvelle-fonctionnalite
```

### 2. Tests et Validation

```bash
# Tests automatisés
docker-compose -f docker-compose-test.yml up

# Tests manuels
# Vérifier l'interface sur http://localhost:8080/matchID/

# Build de production pour validation
npm run build
```

### 3. Intégration

```bash
# Merge de la branche
git checkout main
git merge feature/nouvelle-fonctionnalite

# Tag de version (optionnel)
git tag v1.0.1
git push origin v1.0.1
```

## Ressources Utiles

### Documentation
- [Architecture Frontend](../frontend/architecture.md)
- [Configuration Docker](../deployment/docker-setup.md)
- [Variables d'Environnement](../deployment/environment-variables.md)

### Outils de Développement
- **Vue DevTools** : Extension navigateur pour Vue.js
- **Vite DevTools** : Outils de développement Vite
- **Docker Desktop** : Interface graphique pour Docker

### Liens Externes
- [Vue.js Documentation](https://vuejs.org/)
- [Vite Documentation](https://vitejs.dev/)
- [Docker Documentation](https://docs.docker.com/)

---

*Ce guide vous permet de démarrer rapidement le développement sur MatchID. Pour des configurations avancées, consultez la documentation détaillée.*