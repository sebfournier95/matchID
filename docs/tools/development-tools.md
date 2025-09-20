# Outils de Développement - MatchID

## Vue d'ensemble

MatchID fournit un ensemble complet d'outils pour le développement, le déploiement et la gestion des données. Ces outils sont principalement orchestrés via un Makefile complexe qui automatise les tâches courantes.

## Makefile Principal

**Localisation** : [`packages/tools/Makefile`](../../packages/tools/Makefile)

Le Makefile principal contient plus de 1200 lignes de scripts d'automatisation couvrant :
- Installation et configuration système
- Gestion Docker
- Déploiement multi-cloud
- Gestion des données
- Tests et monitoring

### Commandes Principales

#### Configuration Système
```bash
# Installation des outils de base
make tools-install              # curl, jq, etc.
make docker-install            # Installation Docker + Docker Compose
make config-init               # Configuration initiale
make config-next               # Configuration avancée
make config                    # Configuration complète
```

#### Gestion Docker
```bash
# Build et gestion des images
make docker-build              # Build des images
make docker-tag                # Tag des images
make docker-push               # Push vers le registry
make docker-pull               # Pull des images
make docker-check              # Vérification des images

# Logs et monitoring
make docker-logs-to-API        # Envoi des logs vers l'API
```

#### Déploiement Cloud
```bash
# Gestion des instances
make cloud-instance-up         # Démarrage d'instance
make cloud-instance-down       # Arrêt d'instance
make remote-deploy             # Déploiement distant
make remote-config             # Configuration distante
make remote-clean              # Nettoyage distant

# Tests API distants
make remote-test-api           # Test API via HTTPS
make remote-test-api-in-vpc    # Test API en VPC
make local-test-api            # Test API local
```

## Outils de Stockage

### rclone - Synchronisation Multi-Cloud

**Configuration** : Variables `RCLONE_*`

#### Fonctionnalités
- Support S3, Swift, et autres providers
- Synchronisation bidirectionnelle
- Montage de stockage distant
- Chunking pour gros fichiers

#### Commandes
```bash
# Gestion du catalogue
make rclone-get-catalog        # Récupération du catalogue
make get-catalog               # Alias générique

# Synchronisation
make rclone-sync-pull          # Sync distant → local
make rclone-sync-push          # Sync local → distant
make storage-sync-pull         # Alias générique pull
make storage-sync-push         # Alias générique push

# Transferts individuels
make rclone-push               # Upload d'un fichier
make rclone-pull               # Download d'un fichier
make storage-push              # Alias générique push
make storage-pull              # Alias générique pull

# Montage
make rclone-mount              # Montage du stockage distant
make storage-mount             # Alias générique
```

### Swift - Stockage OpenStack

**Configuration** : Variables `OS_*`

#### Commandes
```bash
make swift-get-catalog         # Catalogue Swift
make swift-push                # Upload Swift
make swift-pull                # Download Swift
```

### AWS S3

**Configuration** : Variables `AWS_*`

#### Commandes
```bash
make aws-get-catalog           # Catalogue S3
make aws-push                  # Upload S3
make aws-pull                  # Download S3
```

## Outils de Déploiement Cloud

### Scaleway (SCW)

#### Gestion des Instances
```bash
# Cycle de vie des instances
make SCW-instance-order        # Commande d'instance
make SCW-instance-start        # Démarrage
make SCW-instance-wait-running # Attente du démarrage
make SCW-instance-get-host     # Récupération de l'IP
make SCW-instance-delete       # Suppression

# Réseau privé
make SCW-instance-attach-private-network  # Attachement VPC

# Images et snapshots
make SCW-instance-snapshot     # Création de snapshot
make SCW-instance-image        # Création d'image
make SCW-instance-snapshot-delete  # Suppression snapshot
```

#### Gestion Multi-Instances
```bash
# Instances taguées
make SCW-instance-get-tagged-ids      # IDs des instances
make SCW-instance-get-tagged-hosts    # IPs des instances
make SCW-instance-delete-invalid      # Suppression instances invalides
```

### AWS EC2

#### Gestion des Instances
```bash
# Cycle de vie
make EC2-instance-order        # Commande d'instance
make EC2-instance-wait-running # Attente du démarrage
make EC2-instance-get-host     # Récupération de l'IP
make EC2-instance-delete       # Suppression

# Clés SSH
make EC2-add-sshkey           # Ajout de clé SSH
```

### OpenStack

#### Gestion des Instances
```bash
# Cycle de vie
make OS-instance-order         # Commande d'instance
make OS-instance-wait-running  # Attente du démarrage
make OS-instance-get-host      # Récupération de l'IP
make OS-instance-delete        # Suppression

# Clés SSH
make OS-add-sshkey            # Ajout de clé SSH
```

## Outils de Données

### Data.gouv.fr Integration

#### Récupération de Données
```bash
# Catalogue et métadonnées
make datagouv-get-catalog      # Récupération du catalogue
make datagouv-get-files        # Téléchargement des fichiers

# Synchronisation vers le stockage
make datagouv-to-storage       # Sync vers stockage générique
make datagouv-to-rclone        # Sync vers rclone
make datagouv-to-aws           # Sync vers AWS S3
```

#### Traitement des Fichiers
- Validation SHA1 automatique
- Conversion d'encodage (UTF-8 ↔ Latin-1)
- Compression gzip
- Génération de checksums

### Génération de Données de Test
```bash
make generate-test-file        # Génération de fichier de test
# Crée un fichier de 256MB avec des données aléatoires
```

## Outils de Monitoring

### New Relic Integration
```bash
make remote-install-monitor    # Installation New Relic + Fluent Bit
```

**Fonctionnalités** :
- Installation automatique de l'agent New Relic
- Configuration Fluent Bit pour les logs
- Support proxy pour les environnements d'entreprise
- Monitoring système et application

### Fluent Bit Configuration

**Fichiers** :
- [`packages/tools/fluent-bit/fluent-bit.conf`](../../packages/tools/fluent-bit/fluent-bit.conf)
- [`packages/tools/fluent-bit/plugins.conf`](../../packages/tools/fluent-bit/plugins.conf)

**Fonctionnalités** :
- Collecte de logs Docker
- Envoi vers New Relic
- Support S3 pour archivage
- Configuration proxy

## Outils de Load Balancing

### Nginx Configuration
```bash
# Configuration du load balancer
make nginx-conf-create         # Création de la configuration
make nginx-conf-backup         # Sauvegarde de la configuration
make nginx-conf-apply          # Application de la configuration
```

**Fonctionnalités** :
- Configuration automatique des upstreams
- Support multi-instances
- Backup automatique des configurations
- Rechargement sans interruption

## Outils de Test

### Tests de Performance
```bash
# Tests Artillery
make test-api-generic          # Tests de performance génériques
```

**Configuration** :
- Scénarios configurables
- Rapports détaillés
- Support multi-environnements

### Tests API
```bash
# Tests locaux et distants
make local-test-api            # Test API local
make remote-test-api           # Test API distant (HTTPS)
make remote-test-api-in-vpc    # Test API en VPC
```

**Fonctionnalités** :
- Tests de santé automatiques
- Validation JSON des réponses
- Support des données de test personnalisées

## Outils de Notification

### Slack Integration
```bash
make slack-notification        # Envoi de notification Slack
```

**Variables** :
- `SLACK_WEBHOOK` : Webhook Slack
- `SLACK_TITLE` : Titre du message
- `SLACK_MSG` : Contenu du message

## Outils de Sécurité

### Gestion des Clés SSH
```bash
# Génération automatique de clés
# Les clés sont générées automatiquement si elles n'existent pas
```

**Fonctionnalités** :
- Génération RSA 4096 bits
- Support des passphrases
- Déploiement automatique sur les clouds

### Configuration Proxy
```bash
make config-proxy              # Configuration proxy système
make docker-config-proxy       # Configuration proxy Docker
make remote-config-proxy       # Configuration proxy distant
```

## Outils de CDN

### Cloudflare Integration
```bash
make cdn-cache-purge           # Purge du cache CDN
```

**Fonctionnalités** :
- Purge automatique du cache
- Support API Cloudflare
- Intégration dans les workflows de déploiement

## Scripts d'Automatisation

### Déploiement Automatisé
```bash
# Workflow complet
make remote-actions ACTIONS="docker-build docker-push"
```

**Fonctionnalités** :
- Exécution de commandes distantes
- Support des variables d'environnement
- Gestion des erreurs et rollback

### Configuration Automatique
```bash
# Installation complète sur instance distante
make remote-config             # Configuration de base
make remote-install-monitor    # Installation monitoring
make remote-install-root-aws-credentials  # Credentials AWS
```

## Utilitaires Système

### Détection du Système
```bash
make os-type                   # Détection du type d'OS (DEB/RPM)
make version                   # Affichage de la version
```

### Gestion des Versions
- Versioning automatique basé sur Git
- Support des tags et branches
- Génération de hash de version

## Configuration Multi-Environnement

### Support Multi-Cloud
Le système supporte nativement :
- **Scaleway** (par défaut)
- **AWS EC2** (Outscale)
- **OpenStack** (OVH)

### Variables d'Environnement
Chaque provider a ses propres variables :
- `artifacts.SCW` : Configuration Scaleway
- `artifacts.EC2.outscale` : Configuration AWS/Outscale
- `artifacts.OS.ovh` : Configuration OpenStack/OVH

## Bonnes Pratiques

### Utilisation des Outils
1. **Configuration** : Toujours commencer par `make config`
2. **Tests** : Utiliser `make local-test-api` avant le déploiement
3. **Monitoring** : Activer le monitoring avec `make remote-install-monitor`
4. **Sauvegarde** : Utiliser les snapshots avant les modifications importantes

### Sécurité
1. **Clés** : Ne jamais commiter les clés privées
2. **Tokens** : Utiliser des variables d'environnement pour les secrets
3. **Proxy** : Configurer les proxies pour les environnements d'entreprise

### Performance
1. **Cache** : Utiliser le cache Docker avec `DC_BUILD_ARGS`
2. **Parallélisme** : Les déploiements multi-instances sont parallélisés
3. **Monitoring** : Surveiller les performances avec New Relic

---

*Ces outils permettent une gestion complète du cycle de vie de MatchID, du développement à la production.*