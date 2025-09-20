# Documentation MatchID

Cette documentation fournit une analyse complète du projet MatchID, incluant son architecture, sa configuration, et les procédures de développement local.

## 🚀 Démarrage Rapide

**Nouveau sur MatchID ?** Commencez par le [Guide de Démarrage Rapide](./development/quick-start.md) pour être opérationnel en moins de 10 minutes.

## 📚 Structure de la Documentation

### 📋 Architecture et Vue d'ensemble
- **[Architecture Globale](./architecture/overview.md)** - Vue d'ensemble du système et composants principaux
- **[Flux de Données et API](./architecture/data-flow.md)** - Pipeline de données et endpoints API
- **[Synthèse Technique](./technical/technical-summary.md)** - Résumé exécutif pour les équipes techniques

### 🔧 Configuration et Déploiement
- **[Configuration Docker](./deployment/docker-setup.md)** - Architecture containerisée et orchestration
- **[Variables d'Environnement](./deployment/environment-variables.md)** - Configuration complète multi-environnement

### 💻 Développement Local
- **[Guide de Démarrage Rapide](./development/quick-start.md)** - Installation et lancement (⭐ **Recommandé**)
- **[Guide de Dépannage](./development/troubleshooting.md)** - Solutions aux problèmes courants

### 🎨 Frontend Vue.js
- **[Architecture Frontend](./frontend/architecture.md)** - Structure Vue.js 3, composants et intégrations

### 🛠️ Outils et Automatisation
- **[Outils de Développement](./tools/development-tools.md)** - Makefile, déploiement cloud, monitoring

## 🏗️ Architecture du Projet

MatchID est une plateforme de préparation et d'appariement de données composée de :

- **Frontend Vue.js 3** - Interface utilisateur moderne avec Vite
- **Infrastructure Docker** - Containerisation avec Nginx et Node.js
- **Multi-Cloud** - Support Scaleway, AWS, OpenStack
- **Pipeline de Données** - Ingestion Data.gouv.fr → Transformation → Validation
- **Outils d'Orchestration** - Makefile complexe (1200+ lignes) pour l'automatisation

## 🔗 Liens Rapides

### Pour les Développeurs
- [Démarrage en 5 minutes](./development/quick-start.md#installation-rapide)
- [Configuration Docker](./deployment/docker-setup.md#développement-local)
- [Variables d'environnement](./deployment/environment-variables.md#développement-local-minimal)

### Pour les DevOps
- [Déploiement Cloud](./tools/development-tools.md#déploiement-cloud)
- [Monitoring et Logs](./tools/development-tools.md#outils-de-monitoring)
- [Variables de Production](./deployment/environment-variables.md#production-scaleway)

### Pour les Architectes
- [Vue d'ensemble technique](./technical/technical-summary.md)
- [Flux de données](./architecture/data-flow.md)
- [Architecture globale](./architecture/overview.md)

## 📋 Checklist de Démarrage

- [ ] **Prérequis installés** : Docker, Docker Compose, Git, Make
- [ ] **Projet cloné** : `git clone https://github.com/matchid-project/matchID.git`
- [ ] **Configuration créée** : Copier et adapter le fichier `artifacts`
- [ ] **Services démarrés** : `docker-compose -f docker-compose-dev.yml up -d`
- [ ] **Interface accessible** : http://localhost:8080/matchID/
- [ ] **Documentation lue** : [Guide de démarrage rapide](./development/quick-start.md)

## 🆘 Besoin d'Aide ?

1. **Problème technique** → [Guide de Dépannage](./development/troubleshooting.md)
2. **Configuration** → [Variables d'Environnement](./deployment/environment-variables.md)
3. **Architecture** → [Vue d'ensemble](./architecture/overview.md)
4. **Bug ou Question** → [GitHub Issues](https://github.com/matchid-project/matchID/issues)

## 📊 Métriques de la Documentation

- **10 documents** créés couvrant tous les aspects du projet
- **Architecture complète** analysée et documentée
- **350+ variables d'environnement** documentées
- **290+ commandes Make** référencées
- **Guide de démarrage** testé et validé

## 🔄 Mise à Jour de la Documentation

Cette documentation a été générée par analyse complète du projet le **16 septembre 2025**.

Pour contribuer :
1. Modifier les fichiers sources correspondants
2. Mettre à jour la documentation associée
3. Tester les procédures modifiées
4. Soumettre une pull request

---

*Documentation générée automatiquement - Dernière mise à jour : 2025-09-16*