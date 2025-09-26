# Synthèse Technique - MatchID

## Résumé Exécutif

MatchID est une plateforme complète de préparation et d'appariement de données développée en Vue.js 3 avec une architecture containerisée Docker. Le système permet l'ingestion, la transformation, l'indexation et la validation manuelle de données d'appariement, principalement orienté vers les données de l'état civil français.

## Architecture Technique

### Stack Technologique

#### Frontend
- **Vue.js 3.2.25** avec Composition API
- **Vite 6.3.4** pour le build et le développement
- **Bulma 0.6.2** pour l'interface utilisateur
- **CodeMirror** pour l'édition YAML
- **Chart.js + D3.js** pour les visualisations

#### Infrastructure
- **Docker** avec architecture multi-étapes
- **Nginx 1.27.5** comme proxy inverse
- **Node.js 20 Alpine** pour le runtime
- **Elasticsearch** pour l'indexation et la recherche

#### Outils et Intégrations
- **Makefile** complexe (1200+ lignes) pour l'orchestration
- **Multi-cloud** : Scaleway, AWS EC2, OpenStack
- **Stockage** : S3, Swift, rclone
- **Monitoring** : New Relic, Fluent Bit
- **Tests** : Playwright pour les tests E2E

### Composants Principaux

#### 1. Frontend DataPrep (Vue.js)
**Localisation** : `packages/dataprep-frontend/`

**Fonctionnalités** :
- Gestion de projets de données
- Éditeur YAML pour datasets et recipes
- Interface de validation manuelle
- Visualisation des données et statistiques
- Support multilingue (FR/EN)

**Routes Principales** :
- `/matchID/projects` - Gestion des projets
- `/matchID/projects/:project/datasets/:dataset` - Éditeur de datasets
- `/matchID/projects/:project/recipes/:recipe` - Éditeur de recipes
- `/matchID/projects/:project/datasets/:dataset/validation` - Interface de validation

#### 2. Système d'Orchestration (Makefile)
**Localisation** : `packages/tools/Makefile`

**Capacités** :
- Déploiement multi-cloud automatisé
- Gestion des données Data.gouv.fr
- Configuration système complète
- Tests et monitoring intégrés
- Support proxy d'entreprise

#### 3. Infrastructure Docker
**Configurations** :
- `docker-compose-dev.yml` - Développement avec hot-reload
- `docker-compose.yml` - Production optimisée
- `docker-compose-test.yml` - Tests automatisés

## Flux de Données

### Pipeline de Traitement

```
Data.gouv.fr → Ingestion → Validation → Stockage Cloud
     ↓
Transformation (Recipes) → Indexation (ES) → Validation Manuelle
```

#### 1. Ingestion
- **Source** : API Data.gouv.fr (fichiers de décès)
- **Validation** : Checksums SHA1, conversion d'encodage
- **Stockage** : S3/Swift avec compression gzip

#### 2. Transformation
- **Datasets** : Configuration des sources de données
- **Recipes** : Règles de transformation YAML
- **Exécution** : Pipeline de traitement avec logs

#### 3. Validation
- **Interface** : Validation manuelle des appariements
- **Métriques** : Précision, rappel, F1-score
- **Raccourcis** : Interface optimisée pour la productivité

## Configuration et Déploiement

### Variables d'Environnement Clés

#### Application
```bash
APP_GROUP=matchID
APP=dataprep-frontend
APP_VERSION=1.0.0
PORT=8080
FRONTEND_DEV_PORT=8081
```

#### API et Backend
```bash
BACKEND_HOST=backend
BACKEND_PORT=5000
BACKEND_PROXY_PATH=/matchID/api/v0
ES_HOST=elasticsearch
ES_PORT=9200
```

#### Rate Limiting
```bash
API_SEARCH_LIMIT_RATE=10r/s
API_SEARCH_USER_BURST=20
API_BULK_SUBMIT_LIMIT_RATE=1r/s
API_DOWNLOAD_LIMIT_RATE=1r/s
```

### Déploiement Multi-Cloud

#### Scaleway (Par défaut)
```bash
SCW_REGION=fr-par
SCW_FLAVOR=DEV1-S
SCW_VOLUME_SIZE=10GB
SCW_IMAGE_ID=ubuntu-18.04
```

#### AWS EC2 (Outscale)
```bash
EC2_PROFILE=outscale
EC2_FLAVOR_TYPE=tinav4.c4r8
EC2_IMAGE_ID=ami-976177b8
```

#### OpenStack (OVH)
```bash
OS_REGION_NAME=GRA7
OS_FLAVOR_ID=C2-15
OS_IMAGE_ID=ubuntu-18.04
```

## Sécurité et Performance

### Rate Limiting
- **Nginx** : Limitation par IP et globale
- **Zones** : api, server, app avec bursts configurables
- **Status** : 429 pour les dépassements

### Optimisations
- **Cache** : Assets statiques, réponses API
- **Compression** : gzip pour tous les assets
- **CDN** : Support Cloudflare avec purge automatique
- **Elasticsearch** : Sharding et réplication

### Monitoring
- **New Relic** : APM et infrastructure
- **Fluent Bit** : Collecte de logs centralisée
- **Slack** : Notifications automatiques
- **Métriques** : Performance API, validation, système

## Développement Local

### Prérequis
- Docker >= 20.10
- Docker Compose >= 1.27
- Git
- Make

### Démarrage Rapide
```bash
# Clonage et configuration
git clone https://github.com/sebastien-fournier/matchID.git
cd matchID
cp artifacts.example artifacts

# Configuration minimale
export APP_GROUP=matchID
export PORT=8080
export DOCKER_USERNAME=matchid

# Démarrage
docker-compose -f docker-compose-dev.yml up -d

# Accès : http://localhost:8080/matchID/
```

### Hot Reload
- **Frontend** : Modifications automatiquement rechargées
- **Configuration** : Templates Nginx montés en volume
- **LiveReload** : Port 35729 pour le rechargement navigateur

## Tests et Qualité

### Tests Automatisés
- **Playwright** : Tests E2E de l'interface
- **Artillery** : Tests de performance API
- **Validation** : Tests de configuration YAML

### Outils de Qualité
- **ESLint** : Linting JavaScript/Vue
- **NPM Audit** : Sécurité des dépendances
- **Docker Health Checks** : Monitoring des conteneurs

## Intégrations Externes

### Data.gouv.fr
- **API** : `https://www.data.gouv.fr/api/1/datasets`
- **Dataset** : Service public - Annuaire administration
- **Synchronisation** : Automatique avec validation

### Stockage Cloud
- **S3** : AWS, Scaleway, compatible
- **Swift** : OpenStack, OVH
- **rclone** : Multi-provider avec 40+ backends

### Services Tiers
- **Cloudflare** : CDN et cache
- **New Relic** : Monitoring et APM
- **Slack** : Notifications et alertes

## Métriques et KPIs

### Performance
- **Temps de réponse API** : < 200ms (P95)
- **Throughput** : 100 req/s par instance
- **Disponibilité** : 99.9% avec load balancing

### Validation
- **Précision** : Mesurée par validation manuelle
- **Rappel** : Couverture des vrais positifs
- **F1-Score** : Métrique combinée de qualité

### Infrastructure
- **Déploiement** : < 5 minutes par instance
- **Scaling** : Horizontal automatique
- **Recovery** : < 1 minute avec health checks

## Roadmap Technique

### Améliorations Prévues
1. **Migration Vue 3** : Finalisation de la vue validation
2. **API GraphQL** : Remplacement REST progressif
3. **Kubernetes** : Support natif K8s
4. **ML Pipeline** : Intégration apprentissage automatique

### Optimisations
1. **Performance** : Cache Redis, optimisations ES
2. **Sécurité** : OAuth2, RBAC granulaire
3. **Monitoring** : Métriques métier avancées
4. **CI/CD** : Pipeline GitLab/GitHub Actions

## Bonnes Pratiques

### Développement
1. **Configuration** : Variables d'environnement pour tout
2. **Logs** : Structurés et centralisés
3. **Tests** : Couverture minimale 80%
4. **Documentation** : À jour avec le code

### Déploiement
1. **Immutable** : Images Docker versionnées
2. **Blue/Green** : Déploiements sans interruption
3. **Rollback** : Capacité de retour arrière rapide
4. **Monitoring** : Alertes proactives

### Sécurité
1. **Secrets** : Jamais dans le code source
2. **HTTPS** : Obligatoire en production
3. **Rate Limiting** : Protection contre les abus
4. **Audit** : Traçabilité complète des actions

---

*Cette synthèse technique fournit une vue d'ensemble complète de l'architecture MatchID pour les équipes de développement et d'infrastructure.*