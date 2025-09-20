# Flux de Données et API - MatchID

## Vue d'ensemble des Flux

MatchID traite les données selon un pipeline structuré allant de l'ingestion à la validation, en passant par la transformation et l'indexation. Ce document détaille les flux de données et les API impliquées.

## Architecture des Flux de Données

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Data.gouv.fr  │───▶│  Ingestion &     │───▶│   Stockage      │
│   (Source)      │    │  Validation      │    │   Cloud         │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │◀───│  Transformation  │───▶│ Elasticsearch   │
│   (Validation)  │    │   & Indexation   │    │   (Index)       │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

## 1. Flux d'Ingestion des Données

### Source : Data.gouv.fr

**API Endpoint** : `https://www.data.gouv.fr/api/1/datasets`

#### Dataset Principal
- **ID** : `service-public-fr-annuaire-de-l-administration-base-de-donnees-locales`
- **Type** : Fichiers de décès de l'état civil français
- **Format** : CSV compressé (.gz)
- **Fréquence** : Mise à jour mensuelle

#### Processus d'Ingestion

1. **Récupération du Catalogue**
   ```bash
   GET https://www.data.gouv.fr/api/1/datasets/{dataset_id}/
   ```
   - Récupération des métadonnées
   - Liste des ressources disponibles
   - Validation des checksums SHA1

2. **Téléchargement des Fichiers**
   ```bash
   # Exemple de ressource
   {
     "url": "https://www.data.gouv.fr/fr/datasets/r/{resource_id}",
     "checksum": {"value": "sha1_hash"},
     "format": "CSV"
   }
   ```

3. **Validation et Traitement**
   - Vérification SHA1
   - Conversion d'encodage (UTF-8 ↔ Latin-1 selon l'année)
   - Compression gzip
   - Génération de nouveaux checksums

4. **Stockage Cloud**
   - Upload vers S3/Swift/rclone
   - Réplication multi-zone (optionnelle)
   - Archivage long terme

### Variables de Configuration

```bash
# API Data.gouv.fr
DATAGOUV_API=https://www.data.gouv.fr/api/1/datasets
DATAGOUV_DATASET=service-public-fr-annuaire-de-l-administration-base-de-donnees-locales

# Filtres de fichiers
FILES_PATTERN=.*                    # Pattern général
FILES_PATTERN_FORCE=deces.*         # Pattern forcé pour les fichiers de décès

# Stockage
STORAGE_BUCKET=matchid              # Bucket de destination
STORAGE_CHUNK_SIZE=50M              # Taille des chunks pour gros fichiers
```

## 2. Flux de Transformation

### Pipeline de Traitement

#### Étape 1 : Datasets (Sources de Données)
**API Endpoint** : `/matchID/api/v0/datasets/{dataset_id}`

**Fonctionnalités** :
- Configuration des connecteurs (upload, elasticsearch)
- Définition des schémas de données
- Paramètres de connexion aux sources

**Exemple de Configuration Dataset** :
```yaml
connector: elasticsearch
table: deces_index
host: ${ES_HOST}
port: ${ES_PORT}
index: ${ES_INDEX}
```

#### Étape 2 : Recipes (Transformations)
**API Endpoint** : `/matchID/api/v0/recipes/{recipe_id}`

**Fonctionnalités** :
- Définition des transformations de données
- Règles de nettoyage et normalisation
- Algorithmes d'appariement
- Configuration des seuils de décision

**Exemple de Configuration Recipe** :
```yaml
input: source_dataset
output: processed_dataset
steps:
  - normalize_names
  - fuzzy_matching
  - scoring
```

#### Étape 3 : Exécution et Test
**API Endpoints** :
- `PUT /matchID/api/v0/recipes/{recipe_id}/test` : Test d'exécution
- `POST /matchID/api/v0/datasets/{dataset_id}/` : Prévisualisation

**Réponse Type** :
```json
{
  "data": [...],           // Données transformées
  "log": "...",           // Logs d'exécution
  "status": "success"     // Statut de l'exécution
}
```

## 3. Flux d'Indexation

### Elasticsearch Integration

**Configuration** :
```bash
ES_HOST=elasticsearch               # Hôte Elasticsearch
ES_PORT=9200                       # Port Elasticsearch
ES_INDEX=matchid                   # Index principal
ES_MAX_RESULTS=1000                # Limite de résultats
ES_PROXY_PATH=/matchID/api/v0/es   # Chemin du proxy
```

#### Processus d'Indexation

1. **Préparation des Données**
   - Application des recipes de transformation
   - Normalisation des champs
   - Génération des scores d'appariement

2. **Indexation Bulk**
   - Insertion par lots pour optimiser les performances
   - Gestion des erreurs et retry automatique
   - Mise à jour incrémentale

3. **Configuration d'Index**
   ```json
   {
     "mappings": {
       "properties": {
         "nom": {"type": "text", "analyzer": "french"},
         "prenom": {"type": "text", "analyzer": "french"},
         "date_naissance": {"type": "date"},
         "score": {"type": "float"}
       }
     }
   }
   ```

## 4. Flux de Validation

### Interface de Validation

**Route** : `/matchID/projects/{project}/datasets/{dataset}/validation`

#### Processus de Validation Manuelle

1. **Récupération des Paires**
   - Sélection aléatoire ou ciblée
   - Filtrage par seuil de score
   - Exclusion des paires déjà validées

2. **Présentation à l'Utilisateur**
   - Affichage côte à côte des enregistrements
   - Mise en évidence des différences
   - Calcul des scores de similarité

3. **Collecte des Décisions**
   - Positif (match confirmé)
   - Négatif (pas de match)
   - Indécis (nécessite une révision)

4. **Stockage des Annotations**
   - Sauvegarde des décisions utilisateur
   - Horodatage et traçabilité
   - Calcul des métriques de performance

### API de Validation

**Endpoints Principaux** :
```bash
# Récupération des paires à valider
GET /matchID/api/v0/validation/{dataset}/pairs

# Soumission d'une décision
POST /matchID/api/v0/validation/{dataset}/decision
{
  "pair_id": "12345",
  "decision": "positive",
  "confidence": 0.9,
  "user_id": "validator_1"
}

# Statistiques de validation
GET /matchID/api/v0/validation/{dataset}/stats
```

## 5. API Frontend

### Configuration API

**Fichier** : [`packages/dataprep-frontend/src/assets/json/backend.json`](../../packages/dataprep-frontend/src/assets/json/backend.json)
```json
{
    "api": {
        "url": "/matchID/api/v0/"
    }
}
```

### Endpoints Frontend

#### Gestion des Projets
```bash
GET /matchID/api/v0/projects              # Liste des projets
POST /matchID/api/v0/projects             # Création de projet
DELETE /matchID/api/v0/projects/{id}      # Suppression de projet
```

#### Gestion des Datasets
```bash
GET /matchID/api/v0/datasets              # Liste des datasets
GET /matchID/api/v0/datasets/{id}         # Détails d'un dataset
GET /matchID/api/v0/datasets/{id}/yaml    # Configuration YAML
POST /matchID/api/v0/datasets/{id}/       # Prévisualisation des données
```

#### Gestion des Recipes
```bash
GET /matchID/api/v0/recipes               # Liste des recipes
GET /matchID/api/v0/recipes/{id}          # Détails d'une recipe
GET /matchID/api/v0/recipes/{id}/yaml     # Configuration YAML
PUT /matchID/api/v0/recipes/{id}/test     # Test d'exécution
```

#### Configuration et Sauvegarde
```bash
POST /matchID/api/v0/conf/{project}/{source}  # Sauvegarde de configuration
{
  "yaml": "configuration_yaml_content"
}
```

#### Gestion des Jobs
```bash
GET /matchID/api/v0/jobs                  # Liste des tâches
GET /matchID/api/v0/jobs/{id}             # Détails d'une tâche
POST /matchID/api/v0/jobs                 # Création de tâche
DELETE /matchID/api/v0/jobs/{id}          # Annulation de tâche
```

## 6. Rate Limiting et Sécurité

### Configuration Nginx

**Template** : [`packages/dataprep-frontend/nginx/default-dev.template`](../../packages/dataprep-frontend/nginx/default-dev.template)

#### Zones de Rate Limiting
```nginx
# Configuration dans nginx.conf
limit_req_zone $binary_remote_addr zone=api:10m rate=${API_SEARCH_LIMIT_RATE};
limit_req_zone $server_name zone=server:10m rate=${API_SEARCH_GLOBAL_LIMIT_RATE};
```

#### Application des Limites
```nginx
location ~ "^/matchID/api/v0/.*" {
    limit_req zone=api burst=${API_USER_BURST};
    limit_req zone=server burst=${API_GLOBAL_BURST};
    limit_req_status 429;
    proxy_pass http://backend;
}
```

### Variables de Rate Limiting

```bash
# Recherche
API_SEARCH_LIMIT_RATE=10r/s         # 10 requêtes/seconde par IP
API_SEARCH_USER_BURST=20            # Burst de 20 requêtes
API_SEARCH_GLOBAL_LIMIT_RATE=100r/s # 100 requêtes/seconde global
API_SEARCH_GLOBAL_BURST=200         # Burst global de 200

# Soumission en lot
API_BULK_SUBMIT_LIMIT_RATE=1r/s     # 1 requête/seconde
API_BULK_SUBMIT_BURST=5             # Burst de 5

# Téléchargements
API_DOWNLOAD_LIMIT_RATE=1r/s        # 1 téléchargement/seconde
```

## 7. Monitoring des Flux

### Métriques Collectées

#### Performance API
- Temps de réponse par endpoint
- Taux d'erreur (4xx, 5xx)
- Throughput (requêtes/seconde)
- Taille des réponses

#### Traitement des Données
- Nombre d'enregistrements traités
- Temps de traitement par batch
- Taux d'erreur de transformation
- Utilisation des ressources

#### Validation
- Nombre de paires validées
- Temps moyen de validation
- Distribution des décisions
- Métriques de qualité (précision, rappel)

### Logs Structurés

**Configuration Fluent Bit** : [`packages/tools/fluent-bit/fluent-bit.conf`](../../packages/tools/fluent-bit/fluent-bit.conf)

```ini
[INPUT]
    Name tail
    Path /var/log/nginx/access.log
    Tag nginx.access

[FILTER]
    Name parser
    Match nginx.access
    Key_Name log
    Parser nginx_access

[OUTPUT]
    Name newrelic
    Match *
    licenseKey ${NEW_RELIC_INGEST_KEY}
```

## 8. Gestion des Erreurs

### Stratégies de Retry

#### Ingestion de Données
- Retry automatique avec backoff exponentiel
- Validation des checksums après chaque tentative
- Fallback vers sources alternatives

#### API Calls
- Circuit breaker pour éviter la surcharge
- Timeout configurables par type d'opération
- Mise en cache des réponses fréquentes

#### Traitement Batch
- Reprise sur erreur au dernier point de contrôle
- Isolation des erreurs par batch
- Notification automatique des échecs critiques

### Codes d'Erreur API

```json
{
  "400": "Bad Request - Paramètres invalides",
  "401": "Unauthorized - Token manquant ou invalide",
  "403": "Forbidden - Permissions insuffisantes",
  "404": "Not Found - Ressource introuvable",
  "429": "Too Many Requests - Rate limit dépassé",
  "500": "Internal Server Error - Erreur serveur",
  "503": "Service Unavailable - Service temporairement indisponible"
}
```

## 9. Optimisations Performance

### Cache Strategy

#### Frontend
- Cache navigateur pour les assets statiques
- Cache API pour les données peu changeantes
- Invalidation automatique sur mise à jour

#### Backend
- Cache Redis pour les requêtes fréquentes
- Cache Elasticsearch pour les agrégations
- Cache de session pour les utilisateurs connectés

### Optimisations Base de Données

#### Elasticsearch
- Sharding adapté au volume de données
- Réplication pour la haute disponibilité
- Index templates pour la cohérence

#### Requêtes
- Pagination systématique
- Filtres précoces pour réduire le dataset
- Agrégations optimisées

---

*Cette architecture de flux de données assure une traçabilité complète et une performance optimale pour le traitement des données d'appariement.*