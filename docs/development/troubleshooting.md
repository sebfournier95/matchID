# Guide de Dépannage - MatchID

## Problèmes Courants et Solutions

### 1. Problèmes de Démarrage

#### Port déjà utilisé
**Symptôme** : Erreur `bind: address already in use`

**Solution** :
```bash
# Vérifier les ports utilisés
netstat -tulpn | grep :8080
lsof -i :8080

# Changer le port dans la configuration
export PORT=8081
# Ou modifier le fichier artifacts
echo "export PORT=8081" >> artifacts

# Redémarrer les services
docker-compose -f docker-compose-dev.yml down
docker-compose -f docker-compose-dev.yml up -d
```

#### Conteneurs qui ne démarrent pas
**Symptôme** : Conteneurs en état `Exited` ou `Restarting`

**Diagnostic** :
```bash
# Vérifier l'état des conteneurs
docker-compose -f docker-compose-dev.yml ps

# Consulter les logs
docker-compose -f docker-compose-dev.yml logs frontend-development
docker-compose -f docker-compose-dev.yml logs nginx-development

# Vérifier les ressources système
docker system df
docker system events
```

**Solutions** :
```bash
# Nettoyer les conteneurs arrêtés
docker container prune -f

# Reconstruire les images
docker-compose -f docker-compose-dev.yml build --no-cache

# Vérifier l'espace disque
df -h
docker system prune -f
```

### 2. Problèmes Docker

#### Permissions Docker
**Symptôme** : `permission denied while trying to connect to the Docker daemon socket`

**Solution** :
```bash
# Ajouter l'utilisateur au groupe docker
sudo usermod -aG docker $USER

# Redémarrer la session
newgrp docker
# Ou se déconnecter/reconnecter

# Vérifier les permissions
docker run hello-world
```

#### Images corrompues ou manquantes
**Symptôme** : Erreurs de build ou images introuvables

**Solution** :
```bash
# Lister les images
docker images

# Supprimer les images corrompues
docker rmi $(docker images -f "dangling=true" -q)

# Reconstruire complètement
docker-compose -f docker-compose-dev.yml down -v
docker-compose -f docker-compose-dev.yml build --no-cache
docker-compose -f docker-compose-dev.yml up -d
```

#### Problèmes de réseau Docker
**Symptôme** : Conteneurs ne peuvent pas communiquer

**Solution** :
```bash
# Vérifier les réseaux
docker network ls
docker network inspect matchid

# Recréer le réseau si nécessaire
docker network rm matchid
docker network create matchid

# Redémarrer les services
docker-compose -f docker-compose-dev.yml up -d
```

### 3. Problèmes Frontend

#### Erreurs de build NPM
**Symptôme** : Échec de `npm install` ou `npm run build`

**Diagnostic** :
```bash
# Accéder au conteneur
docker-compose -f docker-compose-dev.yml exec frontend-development sh

# Vérifier les logs NPM
npm config list
npm cache verify
```

**Solutions** :
```bash
# Nettoyer le cache NPM
docker-compose -f docker-compose-dev.yml exec frontend-development npm cache clean --force

# Supprimer node_modules et réinstaller
docker-compose -f docker-compose-dev.yml exec frontend-development rm -rf node_modules
docker-compose -f docker-compose-dev.yml exec frontend-development npm install

# Reconstruire l'image si nécessaire
docker-compose -f docker-compose-dev.yml build --no-cache frontend-development
```

#### Hot reload ne fonctionne pas
**Symptôme** : Modifications non prises en compte automatiquement

**Vérifications** :
```bash
# Vérifier les volumes montés
docker-compose -f docker-compose-dev.yml config

# Vérifier les permissions des fichiers
ls -la packages/dataprep-frontend/src/

# Vérifier le serveur Vite
curl http://localhost:8081
```

**Solutions** :
```bash
# Redémarrer le service frontend
docker-compose -f docker-compose-dev.yml restart frontend-development

# Vérifier la configuration Vite
cat packages/dataprep-frontend/vite.config.js

# Forcer le rechargement
docker-compose -f docker-compose-dev.yml exec frontend-development npm run dev
```

#### Erreurs de proxy API
**Symptôme** : Erreurs 502 ou 504 sur les appels API

**Diagnostic** :
```bash
# Vérifier la configuration Nginx
docker-compose -f docker-compose-dev.yml exec nginx-development cat /etc/nginx/conf.d/default.conf

# Tester la connectivité backend
docker-compose -f docker-compose-dev.yml exec nginx-development wget -O- http://backend:5000/health
```

**Solutions** :
```bash
# Vérifier les variables d'environnement
echo $BACKEND_HOST
echo $BACKEND_PORT

# Redémarrer Nginx
docker-compose -f docker-compose-dev.yml restart nginx-development

# Vérifier les logs Nginx
docker-compose -f docker-compose-dev.yml logs nginx-development
```

### 4. Problèmes de Configuration

#### Variables d'environnement manquantes
**Symptôme** : Erreurs de configuration ou comportements inattendus

**Diagnostic** :
```bash
# Vérifier les variables chargées
docker-compose -f docker-compose-dev.yml config

# Lister les variables d'environnement dans le conteneur
docker-compose -f docker-compose-dev.yml exec frontend-development env | grep -E "(APP|API|BACKEND)"
```

**Solutions** :
```bash
# Vérifier le fichier artifacts
cat artifacts

# Recharger les variables
source artifacts

# Redémarrer avec les nouvelles variables
docker-compose -f docker-compose-dev.yml down
docker-compose -f docker-compose-dev.yml up -d
```

#### Problèmes de proxy d'entreprise
**Symptôme** : Échecs de téléchargement ou timeouts

**Configuration** :
```bash
# Dans le fichier artifacts
export http_proxy=http://proxy.company.com:8080
export https_proxy=http://proxy.company.com:8080
export no_proxy=localhost,127.0.0.1,*.local,*.company.com

# Variables pour les conteneurs
export remote_http_proxy=$http_proxy
export remote_https_proxy=$https_proxy
export remote_no_proxy=$no_proxy
```

**Test** :
```bash
# Tester la connectivité
curl --proxy $http_proxy https://registry.npmjs.org/

# Vérifier dans le conteneur
docker-compose -f docker-compose-dev.yml exec frontend-development curl https://registry.npmjs.org/
```

### 5. Problèmes de Performance

#### Lenteur de l'interface
**Diagnostic** :
```bash
# Vérifier l'utilisation des ressources
docker stats

# Vérifier les logs pour les erreurs
docker-compose -f docker-compose-dev.yml logs --tail=100
```

**Solutions** :
```bash
# Augmenter les ressources Docker
# Dans Docker Desktop : Settings > Resources

# Optimiser les volumes
# Utiliser des volumes nommés pour node_modules
docker volume create matchid-node-modules

# Nettoyer les données inutiles
docker system prune -a
```

#### Problèmes de mémoire
**Symptôme** : Conteneurs tués par OOM (Out of Memory)

**Solutions** :
```bash
# Augmenter la limite mémoire
docker-compose -f docker-compose-dev.yml up -d --memory=2g

# Vérifier l'utilisation mémoire
docker stats --no-stream

# Optimiser la configuration Node.js
export NODE_OPTIONS="--max-old-space-size=2048"
```

### 6. Problèmes de Données

#### Erreurs Elasticsearch
**Symptôme** : Erreurs de connexion ou d'indexation

**Diagnostic** :
```bash
# Vérifier l'état d'Elasticsearch
curl http://localhost:9200/_cluster/health

# Vérifier les indices
curl http://localhost:9200/_cat/indices
```

**Solutions** :
```bash
# Redémarrer Elasticsearch
docker restart elasticsearch

# Vérifier les logs
docker logs elasticsearch

# Recréer l'index si nécessaire
curl -X DELETE http://localhost:9200/matchid
curl -X PUT http://localhost:9200/matchid
```

#### Problèmes de téléchargement Data.gouv.fr
**Symptôme** : Échecs de téléchargement ou checksums invalides

**Diagnostic** :
```bash
# Tester l'API Data.gouv.fr
curl -s https://www.data.gouv.fr/api/1/datasets/service-public-fr-annuaire-de-l-administration-base-de-donnees-locales/

# Vérifier les logs de téléchargement
make datagouv-get-files
```

**Solutions** :
```bash
# Nettoyer les fichiers partiels
rm -f data/*.tmp

# Forcer le re-téléchargement
rm -f data/*.sha1
make datagouv-get-files

# Vérifier la connectivité
curl -I https://www.data.gouv.fr/
```

### 7. Problèmes de Déploiement Cloud

#### Échecs de provisioning
**Symptôme** : Erreurs lors de la création d'instances

**Diagnostic** :
```bash
# Vérifier les credentials
make SCW-check-api  # Pour Scaleway
aws sts get-caller-identity  # Pour AWS

# Vérifier les quotas
# Consulter la console du provider cloud
```

**Solutions** :
```bash
# Vérifier la configuration
cat artifacts.SCW
source artifacts.SCW

# Nettoyer les ressources orphelines
make cloud-instance-down
make cloud-dir-delete

# Recréer proprement
make cloud-instance-up
```

#### Problèmes SSH
**Symptôme** : Impossible de se connecter aux instances

**Diagnostic** :
```bash
# Vérifier les clés SSH
ls -la ~/.ssh/id_rsa_matchid*

# Tester la connectivité
ssh -i ~/.ssh/id_rsa_matchid ubuntu@instance-ip
```

**Solutions** :
```bash
# Régénérer les clés SSH
rm ~/.ssh/id_rsa_matchid*
make ${CLOUD}-add-sshkey

# Vérifier les groupes de sécurité
# Autoriser le port 22 depuis votre IP
```

### 8. Outils de Diagnostic

#### Logs Centralisés
```bash
# Tous les logs
docker-compose -f docker-compose-dev.yml logs -f

# Logs spécifiques avec horodatage
docker-compose -f docker-compose-dev.yml logs -f -t frontend-development

# Logs depuis une date
docker-compose -f docker-compose-dev.yml logs --since="2024-01-01T00:00:00"
```

#### Monitoring en Temps Réel
```bash
# Utilisation des ressources
docker stats

# Événements Docker
docker events

# Processus dans les conteneurs
docker-compose -f docker-compose-dev.yml exec frontend-development ps aux
```

#### Tests de Connectivité
```bash
# Test API local
make local-test-api

# Test des endpoints
curl -v http://localhost:8080/matchID/
curl -v http://localhost:8080/matchID/api/v0/

# Test DNS et réseau
docker-compose -f docker-compose-dev.yml exec frontend-development nslookup backend
docker-compose -f docker-compose-dev.yml exec nginx-development ping frontend-development
```

### 9. Scripts de Dépannage

#### Script de Diagnostic Complet
```bash
#!/bin/bash
# diagnostic.sh

echo "=== MatchID Diagnostic ==="
echo "Date: $(date)"
echo

echo "=== Docker Status ==="
docker version
docker-compose version
docker system df

echo "=== Container Status ==="
docker-compose -f docker-compose-dev.yml ps

echo "=== Network Status ==="
docker network ls
netstat -tulpn | grep -E "(8080|8081|9200)"

echo "=== Environment Variables ==="
env | grep -E "(APP|DOCKER|PORT|BACKEND|ES_)" | sort

echo "=== Disk Space ==="
df -h

echo "=== Recent Logs ==="
docker-compose -f docker-compose-dev.yml logs --tail=20
```

#### Script de Réinitialisation
```bash
#!/bin/bash
# reset.sh

echo "=== Arrêt des services ==="
docker-compose -f docker-compose-dev.yml down -v

echo "=== Nettoyage Docker ==="
docker system prune -f
docker volume prune -f

echo "=== Reconstruction ==="
docker-compose -f docker-compose-dev.yml build --no-cache

echo "=== Redémarrage ==="
docker-compose -f docker-compose-dev.yml up -d

echo "=== Vérification ==="
sleep 30
docker-compose -f docker-compose-dev.yml ps
curl -s http://localhost:8080/matchID/ > /dev/null && echo "✓ Frontend OK" || echo "✗ Frontend KO"
```

### 10. Ressources d'Aide

#### Documentation
- [Guide de Démarrage Rapide](./quick-start.md)
- [Configuration Docker](../deployment/docker-setup.md)
- [Variables d'Environnement](../deployment/environment-variables.md)

#### Communauté
- **GitHub Issues** : https://github.com/matchid-project/matchID/issues
- **Documentation** : https://matchid.io/
- **Email** : matchid.project@gmail.com

#### Logs Utiles
```bash
# Logs système
journalctl -u docker
tail -f /var/log/syslog

# Logs applicatifs
docker-compose -f docker-compose-dev.yml logs -f --tail=100

# Logs Nginx
docker-compose -f docker-compose-dev.yml exec nginx-development tail -f /var/log/nginx/access.log
docker-compose -f docker-compose-dev.yml exec nginx-development tail -f /var/log/nginx/error.log
```

---

*Ce guide couvre les problèmes les plus fréquents. Pour des problèmes spécifiques, consultez les logs détaillés et n'hésitez pas à ouvrir une issue sur GitHub.*