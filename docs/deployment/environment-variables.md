# Variables d'Environnement - MatchID

## Vue d'ensemble

MatchID utilise un système complet de variables d'environnement pour configurer tous les aspects du déploiement, du développement local au déploiement cloud multi-provider. Cette documentation détaille toutes les variables disponibles et leur utilisation.

## Variables de Base

### Application
```bash
# Identification de l'application
APP_GROUP=matchID                    # Groupe d'applications
APP_GROUP_MAIL=matchid.project@gmail.com  # Email de contact
APP_GROUP_DOMAIN=matchid.io          # Domaine principal
APP=dataprep-frontend                # Nom de l'application courante
APP_PATH=/path/to/app               # Chemin de l'application
APP_VERSION=1.0.0                   # Version de l'application

# Configuration Docker
DOCKER_USERNAME=matchid             # Nom d'utilisateur Docker Hub
DC_PREFIX=matchid                   # Préfixe des conteneurs
DC_NETWORK=matchid                  # Réseau Docker
DC_BUILD_ARGS=--pull --no-cache     # Arguments de build Docker
```

### Réseau et Ports
```bash
# Ports principaux
PORT=8080                           # Port d'exposition principal
FRONTEND_DEV_PORT=8081              # Port du serveur de développement
BACKEND_PORT=5000                   # Port du backend
ES_PORT=9200                        # Port Elasticsearch

# Hôtes
FRONTEND_DEV_HOST=frontend-development  # Hôte du frontend de développement
BACKEND_HOST=backend                # Hôte du backend
ES_HOST=elasticsearch               # Hôte Elasticsearch
```

## Variables Frontend

### Configuration de Base
```bash
# Application frontend
APP_NAME=dataprep-frontend          # Nom de l'application frontend
FRONTEND_DEV_PORT=8081              # Port de développement Vite
HOST=0.0.0.0                        # Interface d'écoute

# Chemins API
API_PATH=/matchID/api/v0            # Chemin de base de l'API
BACKEND_PROXY_PATH=/matchID/api/v0  # Chemin du proxy backend
ES_PROXY_PATH=/matchID/api/v0/es    # Chemin du proxy Elasticsearch
```

### Configuration API et Intégrations
```bash
# Authentification
BACKEND_TOKEN_USER=dev-token        # Token utilisateur pour le backend
API_EMAIL=contact@matchid.io        # Email de contact API

# Limites et performance
API_MAX_BODY=100M                   # Taille maximale des requêtes
ES_MAX_RESULTS=1000                 # Nombre maximum de résultats ES
API_SEND_TIMEOUT=600                # Timeout d'envoi API
API_READ_TIMEOUT=3600               # Timeout de lecture API

# Seuils de validation
AB_THRESHOLD=0.8                    # Seuil de décision automatique
```

### Intégrations Externes
```bash
# Data.gouv.fr
DATAGOUV_PROXY_PATH=/matchID/api/v0/datagouv
DATAGOUV_CATALOG_URL=https://www.data.gouv.fr/api/1/datasets
DATAGOUV_RESOURCES_URL=https://www.data.gouv.fr/fr/datasets/r
DATAGOUV_RESOURCES_PROXY=/matchID/api/v0/datagouv/resources
DATAGOUV_RESOURCES_REWRITE_PATH=/datagouv/resources

# Monitoring et analytics
GOOGLE_ANALYTICS_ID=GA-XXXXXXXXX   # ID Google Analytics
GOOGLE_ADSENSE_ID=ca-pub-XXXXXXXXX # ID Google AdSense
MITM_URL=https://mitm.matchid.io    # URL du service MITM

# Thème
THEME_DNUM=true                     # Activation du thème DNUM
```

## Variables de Rate Limiting

### Limites par Type d'API
```bash
# API de recherche
API_SEARCH_LIMIT_RATE=10r/s         # Limite de taux de recherche
API_SEARCH_USER_BURST=20            # Burst par utilisateur
API_SEARCH_GLOBAL_LIMIT_RATE=100r/s # Limite globale de recherche
API_SEARCH_GLOBAL_BURST=200         # Burst global de recherche

# API de soumission en lot
API_BULK_SUBMIT_LIMIT_RATE=1r/s     # Limite de soumission en lot
API_BULK_SUBMIT_BURST=5             # Burst de soumission

# API diverses
API_MISC_LIMIT_RATE=5r/s            # Limite API diverses
API_MISC_USER_BURST=10              # Burst utilisateur diverses
API_MISC_GLOBAL_LIMIT_RATE=50r/s    # Limite globale diverses
API_MISC_GLOBAL_BURST=100           # Burst global diverses

# API d'agrégation
API_AGG_LIMIT_RATE=2r/s             # Limite d'agrégation
API_AGG_USER_BURST=5                # Burst utilisateur agrégation
API_AGG_GLOBAL_LIMIT_RATE=20r/s     # Limite globale agrégation
API_AGG_GLOBAL_BURST=50             # Burst global agrégation

# API de téléchargement
API_DOWNLOAD_LIMIT_RATE=1r/s        # Limite de téléchargement
API_USER_SCOPE=ip                   # Scope des limites utilisateur
```

## Variables de Développement

### Configuration NPM
```bash
# Registries
NPM_REGISTRY=https://registry.npmjs.org/  # Registry NPM
SASS_REGISTRY=https://github.com/sass/node-sass/releases/download  # Registry Sass

# Options de build
NPM_FIX=true                        # Correction automatique des vulnérabilités
NPM_LATEST=true                     # Utilisation de la dernière version NPM
NPM_GIT=true                        # Support des dépendances Git
NPM_VERBOSE=true                    # Mode verbose
NPM_AUDIT_IGNORE=false              # Ignorer l'audit de sécurité
NPM_AUDIT_LEVEL=moderate            # Niveau d'audit de sécurité

# Proxy (si nécessaire)
http_proxy=http://proxy.company.com:8080
https_proxy=http://proxy.company.com:8080
no_proxy=localhost,127.0.0.1,*.local
```

### Configuration Système
```bash
# Miroirs et sources
MIRROR_DEBIAN=http://deb.debian.org/debian  # Miroir Debian
CURL_OPTS=--retry 5 --retry-delay 2 -A "GitHub Actions"  # Options cURL

# SSH et sécurité
SSH_TIMEOUT=90                      # Timeout SSH
SSH_PASSPHRASE=""                   # Passphrase SSH (vide par défaut)
```

## Variables Cloud - Scaleway (SCW)

### Configuration de Base
```bash
# Région et endpoints
SCW_REGION=fr-par                   # Région Scaleway
SCW_ZONE=fr-par-1                   # Zone Scaleway
SCW_ENDPOINT=s3.fr-par.scw.cloud    # Endpoint S3
SCW_API=https://api.scaleway.com/instance/v1/zones/fr-par-1
SCW_IPAM_API=https://api.scaleway.com/ipam/v1alpha1/regions/fr-par

# Authentification (à configurer)
SCW_ORGANIZATION_ID=your-org-id     # ID d'organisation
SCW_PROJECT_ID=your-project-id      # ID de projet
SCW_SECRET_TOKEN=your-secret-token  # Token secret
```

### Configuration d'Instance
```bash
# Type et taille d'instance
SCW_FLAVOR=DEV1-S                   # Type d'instance
SCW_VOLUME_SIZE=10000000000         # Taille du volume (10GB)
SCW_VOLUME_TYPE=l_ssd               # Type de volume

# Réseau
SCW_IP=public_ip.address            # Type d'IP (public/private)
SCW_DOMAIN=priv.cloud.scaleway.com  # Domaine privé
SCW_PRIVATE_NETWORK_ID=your-vpc-id  # ID du réseau privé (optionnel)

# SSH
SCW_SSHUSER=ubuntu                  # Utilisateur SSH
CLOUD_SSHOPTS=-J bastion           # Options SSH (bastion si nécessaire)
```

### Images et Kubernetes
```bash
# Images système
SCW_IMAGE_BASE_ID=bfcb8579-a98f-464c-a958-af80eeef020b  # Image de base Ubuntu
SCW_IMAGE_ID=bfcb8579-a98f-464c-a958-af80eeef020b       # Image courante
SCW_IMAGE_TOOLS_ID=bfcb8579-a98f-464c-a958-af80eeef020b # Image outils

# Kubernetes (optionnel)
SCW_KUBE_API=https://api.scaleway.com/k8s/v1/regions/fr-par/clusters
SCW_KUBE_NODES=1                    # Nombre de nœuds
SCW_KUBE_VERSION=1.27.2             # Version Kubernetes
```

## Variables Cloud - AWS EC2

### Configuration Outscale
```bash
# Profil et endpoint
EC2_PROFILE=outscale                # Profil AWS CLI
EC2_ENDPOINT_OPTION=--endpoint https://fcu.eu-west-2.outscale.com

# Instance
EC2_IMAGE_ID=ami-976177b8           # AMI Ubuntu 18.04
EC2_FLAVOR_TYPE=tinav4.c4r8         # Type d'instance
EC2_IP=PublicIpAddress              # Type d'IP (Public/Private)
EC2_SSHUSER=outscale                # Utilisateur SSH
```

## Variables Cloud - OpenStack

### Configuration OVH
```bash
# Authentification OpenStack
OS_AUTH_URL=https://auth.cloud.ovh.net/v3/
OS_IDENTITY_API_VERSION=3
OS_REGION_NAME=GRA7
OS_USER_DOMAIN_NAME=Default
OS_PROJECT_DOMAIN_NAME=Default

# Credentials (à configurer)
OS_TENANT_ID=your-tenant-id         # ID du tenant
OS_TENANT_NAME=your-tenant-name     # Nom du tenant
OS_USERNAME=your-username           # Nom d'utilisateur
OS_PASSWORD=your-password           # Mot de passe

# Instance
OS_FLAVOR_ID=dcde8fb2-9fcc-4da5-bbb3-5c181e68dfe7  # C2-15 (4vCPU 15GB)
OS_IMAGE_ID=c7cd265e-87ae-4f2e-b95c-2f5571d302cf   # Ubuntu 18.04
OS_SSHUSER=ubuntu                   # Utilisateur SSH

# Swift Storage
OS_SWIFT_URL=https://storage.gra.cloud.ovh.net/v1/
OS_SWIFT_ID=AUTH_your-swift-id      # ID Swift
```

## Variables de Stockage

### Configuration S3/rclone
```bash
# Stockage principal
STORAGE_CLI=rclone                  # CLI de stockage (rclone/swift/aws)
STORAGE_BUCKET=matchid              # Nom du bucket
STORAGE_CHUNK_SIZE=50M              # Taille des chunks
STORAGE_ACCESS_KEY=your-access-key  # Clé d'accès
STORAGE_SECRET_KEY=your-secret-key  # Clé secrète

# Configuration rclone pour Scaleway
RCLONE_PROVIDER=s3
RCLONE_CONFIG_S3_TYPE=s3
RCLONE_CONFIG_S3_ENV_AUTH=false
RCLONE_CONFIG_S3_ENDPOINT=s3.fr-par.scw.cloud
RCLONE_CONFIG_S3_REGION=fr-par
RCLONE_CONFIG_S3_ACL=public-read
RCLONE_CONFIG_S3_FORCE_PATH_STYLE=false
```

## Variables de Monitoring

### New Relic
```bash
# Configuration New Relic
NEW_RELIC_API_KEY=your-api-key      # Clé API New Relic
NEW_RELIC_ACCOUNT_ID=your-account-id # ID de compte
NEW_RELIC_INGEST_KEY=your-ingest-key # Clé d'ingestion
NEW_RELIC_REGION=EU                 # Région (EU/US)

# Monitoring
MONITOR_BUCKET=matchid-logs         # Bucket de logs
MONITOR_DIR=/var/log/matchid        # Répertoire de logs
```

### CDN et Cache
```bash
# Cloudflare
CDN_ZONE_ID=your-zone-id            # ID de zone Cloudflare
CDN_TOKEN=your-cloudflare-token     # Token Cloudflare

# Nginx
NGINX_HOST=nginx.matchid.io         # Hôte Nginx
NGINX_USER=nginx                    # Utilisateur Nginx
```

## Variables de Notification

### Slack
```bash
# Notifications Slack
SLACK_WEBHOOK=your-webhook-path     # Chemin du webhook Slack
SLACK_TITLE="MatchID Notification"  # Titre par défaut
SLACK_MSG="Message"                 # Message par défaut
```

## Variables de Test

### Performance et Tests
```bash
# Tests de performance
PERF_SCENARIO=scenario.yml          # Scénario de test
PERF_TEST_ENV=development           # Environnement de test
PLAYWRIGHT_VERSION=1.40.0          # Version Playwright

# Tests API
API_TEST_PATH=health                # Chemin de test API
API_TEST_DATA='{"test": true}'      # Données de test
API_TEST_JSON_PATH=status           # Chemin JSON de validation
TEST_HOST=localhost                 # Hôte de test
```

## Variables Data.gouv.fr

### Configuration Dataset
```bash
# Dataset principal
DATAGOUV_API=https://www.data.gouv.fr/api/1/datasets
DATAGOUV_DATASET=service-public-fr-annuaire-de-l-administration-base-de-donnees-locales

# Filtres de fichiers
FILES_PATTERN=.*                    # Pattern de fichiers à traiter
FILES_PATTERN_FORCE=deces.*         # Pattern de fichiers forcés
```

## Exemples de Configuration

### Développement Local Minimal
```bash
# Configuration minimale pour le développement
export APP_GROUP=matchID
export APP=dataprep-frontend
export APP_VERSION=dev
export PORT=8080
export FRONTEND_DEV_PORT=8081
export DOCKER_USERNAME=matchid
export DC_PREFIX=matchid
export DC_NETWORK=matchid
```

### Production Scaleway
```bash
# Configuration production Scaleway
export CLOUD=SCW
export SCW_REGION=fr-par
export SCW_PROJECT_ID=your-project-id
export SCW_SECRET_TOKEN=your-token
export SCW_FLAVOR=GP1-S
export STORAGE_ACCESS_KEY=your-s3-key
export STORAGE_SECRET_KEY=your-s3-secret
```

### Tests Automatisés
```bash
# Configuration pour les tests
export TEST_HOST=localhost
export PORT=8080
export PLAYWRIGHT_VERSION=1.40.0
export API_TEST_PATH=health
```

---

*Cette configuration complète permet de déployer MatchID dans différents environnements avec une flexibilité maximale.*