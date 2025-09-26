# Déploiement Cloud Multi-Provider - MatchID

## Vue d'ensemble

MatchID supporte le déploiement sur multiple providers cloud avec une approche unifiée utilisant Docker, Kubernetes et des outils d'automatisation. Le système est conçu pour être portable entre Scaleway, AWS et OpenStack/OVH avec des configurations spécifiques à chaque environnement.

## Providers Cloud Supportés

### 1. Scaleway (SCW) - Provider Principal
**Région** : fr-par (Paris)
**Services utilisés** : Instances, Kubernetes Kapsule, Object Storage S3
**Avantages** : Souveraineté française, prix compétitifs, API moderne

### 2. Amazon Web Services (AWS)
**Région** : eu-west-1 (Irlande)
**Services utilisés** : EC2, EKS, S3
**Avantages** : Écosystème mature, services avancés

### 3. OpenStack/OVH
**Région** : GRA7 (Gravelines)
**Services utilisés** : Compute, Swift Storage
**Avantages** : Souveraineté européenne, OpenStack standard

## Architecture Cloud

### Déploiement Standard
```
┌─────────────────────────────────────────────────────────────┐
│                    Load Balancer / Ingress                 │
└─────────────────────┬───────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────┐
│                 Kubernetes Cluster                         │
│  ┌─────────────┐   │   ┌─────────────┐   ┌─────────────┐   │
│  │   Frontend  │   │   │   Backend   │   │  Services   │   │
│  │   (Nginx)   │   │   │(Python/Node)│   │(ES/PG/Redis)│   │
│  └─────────────┘   │   └─────────────┘   └─────────────┘   │
└─────────────────────┼───────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────┐
│                 Persistent Storage                          │
│  ┌─────────────┐   │   ┌─────────────┐   ┌─────────────┐   │
│  │ Object Store│   │   │  Block Store│   │   Backups   │   │
│  │   (S3/Swift)│   │   │    (SSD)    │   │  (Archives) │   │
│  └─────────────┘   │   └─────────────┘   └─────────────┘   │
└─────────────────────┴───────────────────────────────────────┘
```

## Configuration par Provider

### Scaleway (SCW)

#### Prérequis
```bash
# Installation de l'outil CLI Scaleway
curl -s https://raw.githubusercontent.com/scaleway/scaleway-cli/master/scripts/get.sh | sh

# Configuration
scw init
```

#### Variables d'Environnement
```bash
# Configuration Scaleway
export SCW_REGION=fr-par
export SCW_ZONE=fr-par-1
export SCW_PROJECT_ID=your-project-id
export SCW_SECRET_TOKEN=your-secret-token
export SCW_ORGANIZATION_ID=your-org-id

# Instance
export SCW_FLAVOR=GP1-S              # 2 vCPU, 4GB RAM
export SCW_IMAGE_ID=ubuntu_focal     # Ubuntu 20.04
export SCW_VOLUME_SIZE=20GB          # Disque SSD

# Kubernetes
export SCW_KUBE_VERSION=1.27.2
export SCW_KUBE_NODES=3
export SCW_KUBE_NODE_TYPE=GP1-S

# Storage S3
export SCW_S3_ENDPOINT=s3.fr-par.scw.cloud
export SCW_S3_REGION=fr-par
```

#### Déploiement Kubernetes sur Scaleway
```yaml
# k8s/scaleway/cluster.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: scaleway-config
data:
  region: fr-par
  zone: fr-par-1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: matchid-frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: matchid-frontend
  template:
    metadata:
      labels:
        app: matchid-frontend
    spec:
      containers:
      - name: nginx
        image: matchid/dataprep-frontend:latest
        ports:
        - containerPort: 80
        env:
        - name: BACKEND_HOST
          value: "matchid-backend-service"
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: matchid-frontend-service
spec:
  selector:
    app: matchid-frontend
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

#### Commandes de Déploiement Scaleway
```bash
# Création du cluster Kubernetes
make scw-create-cluster

# Déploiement des services
make scw-deploy-k8s

# Configuration du stockage
make scw-setup-storage

# Monitoring
make scw-setup-monitoring
```

### Amazon Web Services (AWS)

#### Prérequis
```bash
# Installation AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configuration
aws configure
```

#### Variables d'Environnement
```bash
# Configuration AWS
export AWS_REGION=eu-west-1
export AWS_PROFILE=default
export AWS_ACCOUNT_ID=your-account-id

# EC2
3.medium    # 2 vCPU, 4GB RAM
export EC2_AMI_ID=ami-0c02fb55956c7d316  # Amazon Linux 2
export EC2_KEY_PAIR=matchid-keypair
export EC2_SECURITY_GROUP=matchid-sg

# EKS
export EKS_CLUSTER_NAME=matchid-cluster
export EKS_NODE_GROUP_NAME=matchid-nodes
export EKS_KUBERNETES_VERSION=1.27

# S3
export S3_BUCKET=matchid-storage-eu-west-1
export S3_REGION=eu-west-1
```

#### Déploiement EKS sur AWS
```yaml
# k8s/aws/cluster.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: matchid-cluster
  region: eu-west-1

nodeGroups:
  - name: matchid-nodes
    instanceType: t3.medium
    desiredCapacity: 3
    minSize: 1
    maxSize: 5
    volumeSize: 20
    ssh:
      allow: true
      publicKeyName: matchid-keypair

addons:
  - name: vpc-cni
  - name: coredns
  - name: kube-proxy
  - name: aws-ebs-csi-driver
```

#### Commandes de Déploiement AWS
```bash
# Création du cluster EKS
make aws-create-cluster

# Déploiement des services
make aws-deploy-k8s

# Configuration du stockage S3
make aws-setup-storage

# Monitoring CloudWatch
make aws-setup-monitoring
```

### OpenStack/OVH

#### Prérequis
```bash
# Installation OpenStack CLI
pip install python-openstackclient
pip install python-swiftclient

# Configuration
source openrc.sh
```

#### Variables d'Environnement
```bash
# Configuration OpenStack
export OS_AUTH_URL=https://auth.cloud.ovh.net/v3/
export OS_IDENTITY_API_VERSION=3
export OS_REGION_NAME=GRA7
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default

# Credentials
export OS_PROJECT_ID=your-project-id
export OS_PROJECT_NAME=your-project-name
export OS_USERNAME=your-username
export OS_PASSWORD=your-password

# Instance
export OS_FLAVOR_NAME=s1-4        # 1 vCPU, 4GB RAM
export OS_IMAGE_NAME=Ubuntu-20.04
export OS_KEYPAIR_NAME=matchid-keypair
export OS_NETWORK_NAME=Ext-Net

# Swift Storage
export OS_CONTAINER_NAME=matchid-storage
```

#### Déploiement sur OpenStack
```bash
# Création d'instances
openstack server create \
  --flavor $OS_FLAVOR_NAME \
  --image $OS_IMAGE_NAME \
  --key-name $OS_KEYPAIR_NAME \
  --network $OS_NETWORK_NAME \
  matchid-node-1

# Configuration Docker Swarm
make os-setup-swarm

# Déploiement des services
make os-deploy-stack
```

## Configurations Kubernetes

### Manifestes Communs

#### Namespace
```yaml
# k8s/common/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: matchid
  labels:
    name: matchid
```

#### ConfigMap
```yaml
# k8s/common/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: matchid-config
  namespace: matchid
data:
  ES_VERSION: "8.6.1"
  POSTGRES_VERSION: "13"
  REDIS_VERSION: "alpine"
  APP_VERSION: "latest"
  DOCKER_USERNAME: "matchid"
```

#### Secrets
```yaml
# k8s/common/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: matchid-secrets
  namespace: matchid
type: Opaque
data:
  postgres-password: <base64-encoded-password>
  elasticsearch-password: <base64-encoded-password>
  backend-token-secret: <base64-encoded-token>
```

### Services de Données

#### Elasticsearch 8.6.1
```yaml
# k8s/common/elasticsearch.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: matchid
spec:
  serviceName: elasticsearch
  replicas: 1
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: matchid/elasticsearch-phonetic:8.6.1
        ports:
        - containerPort: 9200
        - containerPort: 9300
        env:
        - name: discovery.type
          value: single-node
        - name: ES_JAVA_OPTS
          value: "-Xms512m -Xmx512m"
        volumeMounts:
        - name: es-data
          mountPath: /usr/share/elasticsearch/data
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
  volumeClaimTemplates:
  - metadata:
      name: es-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-service
  namespace: matchid
spec:
  selector:
    app: elasticsearch
  ports:
  - port: 9200
    targetPort: 9200
```

#### PostgreSQL avec cstore
```yaml
# k8s/common/postgresql.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql
  namespace: matchid
spec:
  serviceName: postgresql
  replicas: 1
  selector:
    matchLabels:
      app: postgresql
  template:
    metadata:
      labels:
        app: postgresql
    spec:
      containers:
      - name: postgresql
        image: matchid/postgres_cstore:13
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_DB
          value: matchid
        - name: POSTGRES_USER
          value: matchid
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: matchid-secrets
              key: postgres-password
        volumeMounts:
        - name: pg-data
          mountPath: /var/lib/postgresql/data
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
  volumeClaimTemplates:
  - metadata:
      name: pg-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: postgresql-service
  namespace: matchid
spec:
  selector:
    app: postgresql
  ports:
  - port: 5432
    targetPort: 5432
```

#### Redis
```yaml
# k8s/common/redis.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: matchid
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:alpine
        ports:
        - containerPort: 6379
        command: ["redis-server", "--appendonly", "yes"]
        volumeMounts:
        - name: redis-data
          mountPath: /data
        resources:
          requests:
            memory: "256Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "200m"
      volumes:
      - name: redis-data
        persistentVolumeClaim:
          claimName: redis-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: matchid
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Service
metadata:
  name: redis-service
  namespace: matchid
spec:
  selector:
    app: redis
  ports:
  - port: 6379
    targetPort: 6379
```

## Ingress et Load Balancing

### Nginx Ingress Controller
```yaml
# k8s/common/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: matchid-ingress
  namespace: matchid
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
  - hosts:
    - matchid.io
    - api.matchid.io
    secretName: matchid-tls
  rules:
  - host: matchid.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: matchid-frontend-service
            port:
              number: 80
  - host: api.matchid.io
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: matchid-backend-service
            port:
              number: 5000
```

## Monitoring et Observabilité

### Prometheus et Grafana
```yaml
# k8s/monitoring/prometheus.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: matchid
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
    - job_name: 'elasticsearch'
      static_configs:
      - targets: ['elasticsearch-service:9200']
    - job_name: 'postgresql'
      static_configs:
      - targets: ['postgresql-service:5432']
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: matchid
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:latest
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus
        - name: data
          mountPath: /prometheus
      volumes:
      - name: config
        configMap:
          name: prometheus-config
      - name: data
        persistentVolumeClaim:
          claimName: prometheus-pvc
```

## Automatisation du Déploiement

### Makefile Targets
```makefile
# Déploiement multi-cloud
.PHONY: deploy-cloud
deploy-cloud:
	@echo "Déploiement sur $(CLOUD_PROVIDER)"
	$(MAKE) $(CLOUD_PROVIDER)-deploy

# Scaleway
.PHONY: scw-deploy
scw-deploy:
	@echo "Déploiement Scaleway"
	scw k8s cluster create name=$(SCW_CLUSTER_NAME) version=$(SCW_KUBE_VERSION)
	kubectl apply -f k8s/common/
	kubectl apply -f k8s/scaleway/

# AWS
.PHONY: aws-deploy
aws-deploy:
	@echo "Déploiement AWS"
	eksctl create cluster -f k8s/aws/cluster.yaml
	kubectl apply -f k8s/common/
	kubectl apply -f k8s/aws/

# OpenStack
.PHONY: os-deploy
os-deploy:
	@echo "Déploiement OpenStack"
	$(MAKE) os-create-instances
	$(MAKE) os-setup-k8s
	kubectl apply -f k8s/common/
	kubectl apply -f k8s/openstack/
```

### Scripts de Déploiement
```bash
#!/bin/bash
# scripts/deploy-cloud.sh

set -e

CLOUD_PROVIDER=${1:-scw}
ENVIRONMENT=${2:-production}

echo "Déploiement MatchID sur $CLOUD_PROVIDER ($ENVIRONMENT)"

# Chargement de la configuration
source artifacts
source config/$CLOUD_PROVIDER.env

# Validation des prérequis
./scripts/check-prerequisites.sh $CLOUD_PROVIDER

# Déploiement de l'infrastructure
case $CLOUD_PROVIDER in
  scw)
    ./scripts/deploy-scaleway.sh $ENVIRONMENT
    ;;
  aws)
    ./scripts/deploy-aws.sh $ENVIRONMENT
    ;;
  os)
    ./scripts/deploy-openstack.sh $ENVIRONMENT
    ;;
  *)
    echo "Provider non supporté: $CLOUD_PROVIDER"
    exit 1
    ;;
esac

# Vérification du déploiement
./scripts/verify-deployment.sh

echo "Déploiement terminé avec succès!"
```

## Sécurité Cloud

### Network Policies
```yaml
# k8s/security/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: matchid-network-policy
  namespace: matchid
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: matchid
    ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 5000
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
  - to:
    - namespaceSelector:
        matchLabels:
          name: matchid
```

## Backup et Disaster Recovery

### Backup Strategy
```bash
#!/bin/bash
# scripts/backup.sh

# Backup PostgreSQL
kubectl exec -n matchid postgresql-0 -- pg_dump -U matchid matchid > backup/postgres-$(date +%Y%m%d).sql

# Backup Elasticsearch
kubectl exec -n matchid elasticsearch-0 -- curl -X POST "localhost:9200/_snapshot/backup/$(date +%Y%m%d)"

# Upload vers le stockage cloud
case $CLOUD_PROVIDER in
  scw)
    rclone copy backup/ scw:matchid-backups/
    ;;
  aws)
    aws s3 sync backup/ s3://matchid-backups/
    ;;
  os)
    swift upload matchid-backups backup/
    ;;
esac
```

### Disaster Recovery
```bash
#!/bin/bash
# scripts/restore.sh

BACKUP_DATE=${1:-$(date +%Y%m%d)}

echo "Restauration depuis la sauvegarde du $BACKUP_DATE"

# Restauration PostgreSQL
kubectl exec -n matchid postgresql-0 -- psql -U matchid -d matchid < backup/postgres-$BACKUP_DATE.sql

# Restauration Elasticsearch
kubectl exec -n matchid elasticsearch-0 -- curl -X POST "localhost:9200/_snapshot/backup/$BACKUP_DATE/_restore"

echo "Restauration terminée"
```

---

*Ce guide de déploiement cloud permet une mise en production robuste et scalable de MatchID sur multiple providers avec des outils d'automatisation et de monitoring avancés.*
export EC2_INSTANCE_TYPE=t