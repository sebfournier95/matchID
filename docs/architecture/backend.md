# Architecture Backend - MatchID

## Vue d'ensemble

L'architecture backend de MatchID est composée de deux services principaux développés dans des technologies différentes pour répondre à des besoins spécifiques : **dataprep-backend** (Python/Flask) pour la préparation de données et **deces-backend** (Node.js/TypeScript) pour les services de décès et d'appariement.

## Services Backend

### 1. DataPrep Backend (Python/Flask)

**Localisation** : [`packages/dataprep-backend/`](../../packages/dataprep-backend/)
**Technologie** : Python 3.9+ avec Flask
**Rôle** : Préparation, transformation et validation des données

#### Stack Technique
```python
# Frameworks et bibliothèques principales
Flask==2.3.3                    # Framework web
Flask-CORS==4.0.0              # Support CORS
Werkzeug==2.3.7                # WSGI utilities

# Base de données et ORM
SQLAlchemy==2.0.21             # ORM
psycopg2-binary==2.9.7         # Driver PostgreSQL
alembic==1.12.0                # Migrations

# Elasticsearch
elasticsearch==8.6.1           # Client Elasticsearch 8.x
elasticsearch-dsl==8.5.0       # DSL pour requêtes

# Traitement de données
pandas==2.1.1                  # Manipulation de données
numpy==1.24.3                  # Calculs numériques
pyyaml==6.0.1                  # Parsing YAML
```

#### Architecture des Modules

##### API Routes (`/api/v0/`)
```python
# Structure des endpoints
/api/v0/projects               # Gestion des projets
/api/v0/datasets               # Gestion des datasets
/api/v0/recipes                # Gestion des recettes
/api/v0/jobs                   # Gestion des tâches
/api/v0/validation             # Interface de validation
/api/v0/es                     # Proxy Elasticsearch
```

##### Core Modules
```
dataprep-backend/
├── app/
│   ├── __init__.py           # Application Flask
│   ├── models/               # Modèles SQLAlchemy
│   │   ├── project.py
│   │   ├── dataset.py
│   │   ├── recipe.py
│   │   └── job.py
│   ├── routes/               # Routes API
│   │   ├── projects.py
│   │   ├── datasets.py
│   │   ├── recipes.py
│   │   └── validation.py
│   ├── services/             # Logique métier
│   │   ├── data_processor.py
│   │   ├── elasticsearch_service.py
│   │   └── validation_service.py
│   └── utils/                # Utilitaires
│       ├── yaml_parser.py
│       ├── data_validator.py
│       └── file_handler.py
```

##### Configuration Docker
```dockerfile
# Dockerfile multi-étapes
FROM python:3.9-slim as base

# Installation des dépendances système
RUN apt-get update && apt-get install -y \
    gcc \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Installation des dépendances Python
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Application
COPY . /app
WORKDIR /app
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "app:create_app()"]
```

#### Fonctionnalités Principales

##### Gestion des Projets
```python
@bp.route('/projects', methods=['GET', 'POST'])
def projects():
    """Gestion des projets de préparation de données"""
    if request.method == 'POST':
        project = Project(
            name=request.json['name'],
            description=request.json.get('description', ''),
            config=request.json.get('config', {})
        )
        db.session.add(project)
        db.session.commit()
        return jsonify(project.to_dict())
    
    return jsonify([p.to_dict() for p in Project.query.all()])
```

##### Traitement des Données
```python
class DataProcessor:
    """Service de traitement des données"""
    
    def __init__(self, elasticsearch_client):
        self.es = elasticsearch_client
    
    def process_dataset(self, dataset_config):
        """Traitement d'un dataset selon sa configuration"""
        # Chargement des données
        data = self.load_data(dataset_config['source'])
        
        # Application des transformations
        for transform in dataset_config.get('transforms', []):
            data = self.apply_transform(data, transform)
        
        # Indexation dans Elasticsearch
        self.index_data(data, dataset_config['index'])
        
        return data
```

##### Interface de Validation
```python
@bp.route('/validation/<dataset_id>/pairs', methods=['GET'])
def get_validation_pairs(dataset_id):
    """Récupération des paires à valider"""
    pairs = ValidationService.get_pending_pairs(dataset_id)
    return jsonify({
        'pairs': pairs,
        'total': len(pairs),
        'validated': ValidationService.get_validated_count(dataset_id)
    })

@bp.route('/validation/<dataset_id>/validate', methods=['POST'])
def validate_pair(dataset_id):
    """Validation d'une paire"""
    pair_id = request.json['pair_id']
    decision = request.json['decision']  # 'match', 'no_match', 'uncertain'
    
    ValidationService.validate_pair(pair_id, decision)
    return jsonify({'status': 'validated'})
```

### 2. Deces Backend (Node.js/TypeScript)

**Localisation** : [`packages/deces-backend/`](../../packages/deces-backend/)
**Technologie** : Node.js 18+ avec TypeScript et Express
**Rôle** : Services de recherche de décès et API publique

#### Stack Technique
```json
{
  "dependencies": {
    "express": "^4.18.2",
    "typescript": "^5.2.2",
    "@types/node": "^20.6.0",
    "@types/express": "^4.17.17",
    
    "elasticsearch": "^8.18.2",
    "@elastic/elasticsearch": "^8.18.2",
    
    "bullmq": "^5.21.2",
    "ioredis": "^5.3.2",
    
    "helmet": "^7.0.0",
    "cors": "^2.8.5",
    "express-rate-limit": "^6.10.0",
    
    "winston": "^3.10.0",
    "joi": "^17.10.1"
  }
}
```

#### Architecture TypeScript

##### Structure des Modules
```
deces-backend/
├── src/
│   ├── app.ts                # Application Express
│   ├── server.ts             # Serveur HTTP
│   ├── config/               # Configuration
│   │   ├── database.ts
│   │   ├── elasticsearch.ts
│   │   └── redis.ts
│   ├── controllers/          # Contrôleurs
│   │   ├── SearchController.ts
│   │   ├── ValidationController.ts
│   │   └── StatsController.ts
│   ├── services/             # Services métier
│   │   ├── ElasticsearchService.ts
│   │   ├── QueueService.ts
│   │   └── CacheService.ts
│   ├── models/               # Modèles TypeScript
│   │   ├── Person.ts
│   │   ├── SearchQuery.ts
│   │   └── ValidationResult.ts
│   ├── middleware/           # Middlewares
│   │   ├── auth.ts
│   │   ├── rateLimit.ts
│   │   └── validation.ts
│   └── utils/                # Utilitaires
│       ├── logger.ts
│       ├── validators.ts
│       └── formatters.ts
├── tests/                    # Tests
└── docker/                   # Configuration Docker
```

##### Configuration Express
```typescript
// app.ts
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import rateLimit from 'express-rate-limit';

const app = express();

// Sécurité
app.use(helmet());
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || ['http://localhost:8080'],
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limite par IP
  message: 'Trop de requêtes, réessayez plus tard'
});
app.use('/api/', limiter);

// Routes
app.use('/api/v1/search', searchRoutes);
app.use('/api/v1/validation', validationRoutes);
app.use('/api/v1/stats', statsRoutes);

export default app;
```

##### Service Elasticsearch
```typescript
// services/ElasticsearchService.ts
import { Client } from '@elastic/elasticsearch';

export class ElasticsearchService {
  private client: Client;

  constructor() {
    this.client = new Client({
      node: process.env.ES_HOST || 'http://localhost:9200',
      auth: {
        username: process.env.ES_USERNAME || 'elastic',
        password: process.env.ES_PASSWORD || 'changeme'
      }
    });
  }

  async searchPersons(query: SearchQuery): Promise<SearchResult[]> {
    const searchParams = {
      index: 'deces',
      body: {
        query: {
          bool: {
            must: [
              { match: { nom: query.nom } },
              { match: { prenom: query.prenom } }
            ],
            filter: [
              { range: { date_naissance: { gte: query.dateNaissanceMin } } }
            ]
          }
        },
        highlight: {
          fields: {
            nom: {},
            prenom: {}
          }
        }
      }
    };

    const response = await this.client.search(searchParams);
    return this.formatResults(response.body.hits.hits);
  }

  private formatResults(hits: any[]): SearchResult[] {
    return hits.map(hit => ({
      id: hit._id,
      score: hit._score,
      person: hit._source,
      highlights: hit.highlight
    }));
  }
}
```

##### Gestion des Files d'Attente (BullMQ)
```typescript
// services/QueueService.ts
import { Queue, Worker } from 'bullmq';
import IORedis from 'ioredis';

const connection = new IORedis({
  host: process.env.REDIS_HOST || 'localhost',
  port: parseInt(process.env.REDIS_PORT || '6379'),
  maxRetriesPerRequest: 3
});

export class QueueService {
  private indexingQueue: Queue;
  private validationQueue: Queue;

  constructor() {
    this.indexingQueue = new Queue('indexing', { connection });
    this.validationQueue = new Queue('validation', { connection });

    this.setupWorkers();
  }

  private setupWorkers() {
    // Worker pour l'indexation
    new Worker('indexing', async (job) => {
      const { dataset, data } = job.data;
      await this.processIndexing(dataset, data);
    }, { connection });

    // Worker pour la validation
    new Worker('validation', async (job) => {
      const { pairs } = job.data;
      await this.processValidation(pairs);
    }, { connection });
  }

  async addIndexingJob(dataset: string, data: any[]) {
    await this.indexingQueue.add('index-data', { dataset, data });
  }

  async addValidationJob(pairs: ValidationPair[]) {
    await this.validationQueue.add('validate-pairs', { pairs });
  }
}
```

#### Configuration Docker
```dockerfile
# Dockerfile pour deces-backend
FROM node:18-alpine as builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM node:18-alpine as runtime

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

EXPOSE 3000
CMD ["node", "dist/server.js"]
```

## Communication Inter-Services

### API Gateway Pattern
```nginx
# Configuration Nginx pour le routage
upstream dataprep-backend {
    server dataprep-backend:5000;
}

upstream deces-backend {
    server deces-backend:3000;
}

server {
    listen 80;
    
    # Routes vers dataprep-backend
    location /matchID/api/v0/ {
        proxy_pass http://dataprep-backend/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    # Routes vers deces-backend
    location /api/v1/ {
        proxy_pass http://deces-backend/api/v1/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Partage de Données
```python
# dataprep-backend - Publication d'événements
class EventPublisher:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def publish_dataset_updated(self, dataset_id):
        event = {
            'type': 'dataset_updated',
            'dataset_id': dataset_id,
            'timestamp': datetime.utcnow().isoformat()
        }
        self.redis.publish('matchid_events', json.dumps(event))
```

```typescript
// deces-backend - Écoute d'événements
class EventSubscriber {
  private redis: IORedis;

  constructor() {
    this.redis = new IORedis({
      host: process.env.REDIS_HOST || 'localhost'
    });
    
    this.redis.subscribe('matchid_events');
    this.redis.on('message', this.handleEvent.bind(this));
  }

  private async handleEvent(channel: string, message: string) {
    const event = JSON.parse(message);
    
    switch (event.type) {
      case 'dataset_updated':
        await this.refreshDatasetCache(event.dataset_id);
        break;
    }
  }
}
```

## Monitoring et Observabilité

### Logging Centralisé
```python
# dataprep-backend - Configuration logging
import logging
from pythonjsonlogger import jsonlogger

def setup_logging():
    logger = logging.getLogger()
    handler = logging.StreamHandler()
    formatter = jsonlogger.JsonFormatter(
        '%(asctime)s %(name)s %(levelname)s %(message)s'
    )
    handler.setFormatter(formatter)
    logger.addHandler(handler)
    logger.setLevel(logging.INFO)
```

```typescript
// deces-backend - Winston logger
import winston from 'winston';

export const logger = winston.createLogger({
  level: 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    new winston.transports.File({ filename: 'logs/combined.log' })
  ]
});
```

### Health Checks
```python
# dataprep-backend
@bp.route('/health')
def health_check():
    checks = {
        'database': check_database_connection(),
        'elasticsearch': check_elasticsearch_connection(),
        'redis': check_redis_connection()
    }
    
    status = 'healthy' if all(checks.values()) else 'unhealthy'
    return jsonify({'status': status, 'checks': checks})
```

```typescript
// deces-backend
app.get('/health', async (req, res) => {
  const checks = {
    elasticsearch: await checkElasticsearch(),
    redis: await checkRedis(),
    memory: process.memoryUsage(),
    uptime: process.uptime()
  };
  
  const healthy = checks.elasticsearch && checks.redis;
  res.status(healthy ? 200 : 503).json({
    status: healthy ? 'healthy' : 'unhealthy',
    checks
  });
});
```

## Sécurité

### Authentification et Autorisation
```python
# dataprep-backend - JWT middleware
from functools import wraps
import jwt

def token_required(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        token = request.headers.get('Authorization')
        if not token:
            return jsonify({'message': 'Token manquant'}), 401
        
        try:
            data = jwt.decode(token, app.config['SECRET_KEY'], algorithms=['HS256'])
            current_user = User.query.filter_by(id=data['user_id']).first()
        except:
            return jsonify({'message': 'Token invalide'}), 401
        
        return f(current_user, *args, **kwargs)
    return decorated
```

```typescript
// deces-backend - Middleware d'authentification
import jwt from 'jsonwebtoken';

export const authenticateToken = (req: Request, res: Response, next: NextFunction) => {
  const authHeader = req.headers['authorization'];
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.sendStatus(401);
  }

  jwt.verify(token, process.env.JWT_SECRET!, (err: any, user: any) => {
    if (err) return res.sendStatus(403);
    req.user = user;
    next();
  });
};
```

### Rate Limiting Avancé
```typescript
// deces-backend - Rate limiting par utilisateur
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import IORedis from 'ioredis';

const redisClient = new IORedis({
  host: process.env.REDIS_HOST || 'localhost'
});

export const createRateLimit = (windowMs: number, max: number) => {
  return rateLimit({
    store: new RedisStore({
      sendCommand: (...args: string[]) => redisClient.call(...args),
    }),
    windowMs,
    max,
    message: 'Trop de requêtes, réessayez plus tard',
    standardHeaders: true,
    legacyHeaders: false,
  });
};

// Limites spécifiques par endpoint
export const searchRateLimit = createRateLimit(60 * 1000, 100); // 100 req/min
export const validationRateLimit = createRateLimit(60 * 1000, 50); // 50 req/min
```

---

*Cette architecture backend robuste permet de gérer efficacement les besoins de préparation de données (Python/Flask) et de services publics (Node.js/TypeScript) avec une communication fluide entre les services.*