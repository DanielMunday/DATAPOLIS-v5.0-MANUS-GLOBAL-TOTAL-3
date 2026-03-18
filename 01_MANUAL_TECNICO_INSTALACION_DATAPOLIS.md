# MANUAL TÉCNICO DE INSTALACIÓN Y CONFIGURACIÓN
## DATAPOLIS v4.0 - Plataforma Integral de Análisis Territorial y Financiero

**Versión:** 1.0  
**Fecha:** 26 de Febrero de 2026  
**Clasificación:** Documentación Técnica Operativa  
**Audiencia:** Equipos de DevOps, Arquitectos de Sistemas, Desarrolladores

---

## TABLA DE CONTENIDOS

1. [Requisitos del Sistema](#requisitos-del-sistema)
2. [Instalación del Entorno Base](#instalación-del-entorno-base)
3. [Configuración de Bases de Datos](#configuración-de-bases-de-datos)
4. [Instalación del Backend](#instalación-del-backend)
5. [Instalación del Frontend](#instalación-del-frontend)
6. [Configuración de Módulos](#configuración-de-módulos)
7. [Guías de Integración Inter-Módulos](#guías-de-integración-inter-módulos)
8. [Testing y Validación](#testing-y-validación)
9. [Deployment en Producción](#deployment-en-producción)
10. [Troubleshooting](#troubleshooting)

---

## 1. REQUISITOS DEL SISTEMA

### 1.1 Hardware Mínimo

```
Servidor de Desarrollo:
├── CPU: 8 cores (Intel Xeon o equivalente)
├── RAM: 32 GB
├── Almacenamiento: 500 GB SSD
├── Red: 1 Gbps
└── Conectividad: Internet público

Servidor de Producción:
├── CPU: 16+ cores (Intel Xeon E5-2680 v4 o superior)
├── RAM: 64+ GB
├── Almacenamiento: 2+ TB SSD (RAID 10)
├── Red: 10 Gbps (recomendado)
└── Redundancia: Cluster de 3+ nodos
```

### 1.2 Software Requerido

| Componente | Versión | Propósito |
|-----------|---------|----------|
| **OS** | Ubuntu 22.04 LTS | Sistema operativo base |
| **Docker** | 24.0+ | Containerización |
| **Docker Compose** | 2.20+ | Orquestación local |
| **Kubernetes** | 1.28+ | Orquestación producción |
| **Python** | 3.11+ | Backend FastAPI |
| **Node.js** | 20+ | Frontend/Build tools |
| **PostgreSQL** | 16+ | Base de datos relacional |
| **Redis** | 7+ | Cache y pub/sub |
| **Git** | 2.40+ | Control de versiones |

### 1.3 Dependencias de Red

```
Puertos Requeridos:
├── 8000: FastAPI Backend
├── 3000: Next.js Frontend Dev
├── 5432: PostgreSQL
├── 6379: Redis
├── 9200: Elasticsearch (opcional)
├── 27017: MongoDB (opcional)
├── 8080: Kubernetes Dashboard (producción)
└── 443: HTTPS (producción)

Firewall Rules:
├── Entrada: 443 (HTTPS), 22 (SSH)
├── Salida: 443 (HTTPS), 53 (DNS), 123 (NTP)
└── Interno: Todos los puertos entre nodos
```

---

## 2. INSTALACIÓN DEL ENTORNO BASE

### 2.1 Preparación del Sistema Operativo

```bash
# Actualizar sistema
sudo apt-get update && sudo apt-get upgrade -y

# Instalar dependencias base
sudo apt-get install -y \
    build-essential \
    curl \
    wget \
    git \
    vim \
    htop \
    net-tools \
    openssl \
    libssl-dev \
    libffi-dev \
    python3-dev \
    python3-pip \
    python3-venv

# Crear usuario de aplicación
sudo useradd -m -s /bin/bash datapolis
sudo usermod -aG sudo datapolis
sudo usermod -aG docker datapolis

# Crear directorios de aplicación
sudo mkdir -p /opt/datapolis/{backend,frontend,data,logs,config}
sudo chown -R datapolis:datapolis /opt/datapolis
```

### 2.2 Instalación de Docker y Docker Compose

```bash
# Instalar Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Instalar Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" \
    -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verificar instalación
docker --version
docker-compose --version
```

### 2.3 Instalación de Python y Node.js

```bash
# Instalar Python 3.11
sudo apt-get install -y python3.11 python3.11-venv python3.11-dev

# Crear entorno virtual
cd /opt/datapolis/backend
python3.11 -m venv venv
source venv/bin/activate

# Instalar Node.js 20
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verificar versiones
python3.11 --version
node --version
npm --version
```

---

## 3. CONFIGURACIÓN DE BASES DE DATOS

### 3.1 PostgreSQL 16 + PostGIS

```bash
# Instalar PostgreSQL y PostGIS
sudo apt-get install -y postgresql-16 postgresql-16-postgis-3

# Iniciar servicio
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Crear usuario y base de datos
sudo -u postgres psql << EOF
CREATE USER datapolis_user WITH PASSWORD 'secure_password_here';
CREATE DATABASE datapolis_db OWNER datapolis_user;

-- Conectar a la base de datos y habilitar extensiones
\c datapolis_db
CREATE EXTENSION IF NOT EXISTS postgis;
CREATE EXTENSION IF NOT EXISTS postgis_topology;
CREATE EXTENSION IF NOT EXISTS uuid-ossp;
CREATE EXTENSION IF NOT EXISTS json;

-- Otorgar permisos
GRANT ALL PRIVILEGES ON DATABASE datapolis_db TO datapolis_user;
GRANT ALL PRIVILEGES ON SCHEMA public TO datapolis_user;
EOF

# Configurar postgresql.conf para producción
sudo nano /etc/postgresql/16/main/postgresql.conf
# Cambios recomendados:
# shared_buffers = 16GB (25% de RAM)
# effective_cache_size = 48GB (75% de RAM)
# work_mem = 32MB
# maintenance_work_mem = 4GB
# max_connections = 200
# max_parallel_workers = 8
```

### 3.2 Redis 7

```bash
# Instalar Redis
sudo apt-get install -y redis-server

# Iniciar servicio
sudo systemctl start redis-server
sudo systemctl enable redis-server

# Configurar Redis para producción
sudo nano /etc/redis/redis.conf
# Cambios recomendados:
# maxmemory 8gb
# maxmemory-policy allkeys-lru
# appendonly yes
# appendfsync everysec
# requirepass your_redis_password

# Reiniciar Redis
sudo systemctl restart redis-server

# Verificar conexión
redis-cli ping
```

### 3.3 ChromaDB (Vector Store)

```bash
# ChromaDB se instalará como dependencia Python
# Crear directorio de datos
mkdir -p /opt/datapolis/data/chromadb

# Configurar en .env
CHROMADB_PATH=/opt/datapolis/data/chromadb
```

### 3.4 Inicialización de Esquemas

```bash
# Conectar a PostgreSQL
psql -U datapolis_user -d datapolis_db -h localhost

# Ejecutar scripts de inicialización
\i /opt/datapolis/backend/database/init_schemas.sql
\i /opt/datapolis/backend/database/init_compliance.sql
\i /opt/datapolis/backend/database/init_due_diligence.sql
\i /opt/datapolis/backend/database/init_geospatial.sql

# Verificar tablas creadas
\dt
```

---

## 4. INSTALACIÓN DEL BACKEND

### 4.1 Clonar Repositorio y Dependencias

```bash
# Cambiar a usuario datapolis
su - datapolis

# Clonar repositorio
cd /opt/datapolis/backend
git clone https://github.com/datapolis/backend.git .

# Crear entorno virtual
python3.11 -m venv venv
source venv/bin/activate

# Instalar dependencias
pip install --upgrade pip setuptools wheel
pip install -r requirements.txt

# Verificar instalación
pip list | grep -E "fastapi|sqlalchemy|pydantic"
```

### 4.2 Configuración de Variables de Entorno

```bash
# Crear archivo .env
cat > /opt/datapolis/backend/.env << 'EOF'
# ============================================================================
# DATAPOLIS BACKEND - CONFIGURATION
# ============================================================================

# Application
APP_NAME=DATAPOLIS
APP_VERSION=4.0.0
DEBUG=false
ENVIRONMENT=production

# Database
DATABASE_URL=postgresql+asyncpg://datapolis_user:secure_password_here@localhost:5432/datapolis_db
DATABASE_POOL_SIZE=20
DATABASE_MAX_OVERFLOW=40
DATABASE_ECHO=false

# Redis
REDIS_URL=redis://:your_redis_password@localhost:6379/0
REDIS_CACHE_TTL=3600

# ChromaDB
CHROMADB_PATH=/opt/datapolis/data/chromadb
CHROMADB_PERSIST=true

# Security
SECRET_KEY=your-super-secret-key-change-this-in-production
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
REFRESH_TOKEN_EXPIRE_DAYS=7

# CORS
CORS_ORIGINS=["http://localhost:3000","https://yourdomain.com"]
CORS_ALLOW_CREDENTIALS=true
CORS_ALLOW_METHODS=["*"]
CORS_ALLOW_HEADERS=["*"]

# Logging
LOG_LEVEL=INFO
LOG_FORMAT=json
LOG_FILE=/opt/datapolis/logs/backend.log

# External APIs
GOOGLE_MAPS_API_KEY=your_google_maps_key
OPENSTREETMAP_API_KEY=your_osm_key
BLOOMBERG_API_KEY=your_bloomberg_key

# Email (para notificaciones)
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password

# S3 (para almacenamiento de archivos)
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
AWS_REGION=us-east-1
AWS_BUCKET_NAME=datapolis-files

# LLM Configuration
OLLAMA_BASE_URL=http://localhost:11434
OPENAI_API_KEY=your_openai_key
LLM_MODEL=gpt-4-turbo

# Monitoring
PROMETHEUS_ENABLED=true
SENTRY_DSN=your_sentry_dsn

# Feature Flags
FEATURE_PAE_ENABLED=true
FEATURE_COMPLIANCE_ENABLED=true
FEATURE_DUE_DILIGENCE_ENABLED=true
FEATURE_HEDONIC_PRICING_ENABLED=true

EOF

# Establecer permisos
chmod 600 /opt/datapolis/backend/.env
```

### 4.3 Inicialización de Base de Datos

```bash
# Activar entorno virtual
cd /opt/datapolis/backend
source venv/bin/activate

# Ejecutar migraciones (si usa Alembic)
alembic upgrade head

# O ejecutar scripts SQL directamente
psql -U datapolis_user -d datapolis_db -f database/migrations/001_initial_schema.sql

# Verificar tablas
psql -U datapolis_user -d datapolis_db -c "\dt"
```

### 4.4 Instalación de Servicios Adicionales

```bash
# Instalar Ollama (para LLM local)
curl https://ollama.ai/install.sh | sh

# Descargar modelo (ej: Mistral 7B)
ollama pull mistral

# Instalar Elasticsearch (opcional, para full-text search)
docker run -d \
  --name elasticsearch \
  -e discovery.type=single-node \
  -e xpack.security.enabled=false \
  -p 9200:9200 \
  docker.elastic.co/elasticsearch/elasticsearch:8.11.0
```

### 4.5 Inicio del Backend

```bash
# Opción 1: Desarrollo local
cd /opt/datapolis/backend
source venv/bin/activate
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# Opción 2: Producción con Gunicorn
gunicorn -w 4 -k uvicorn.workers.UvicornWorker \
  --bind 0.0.0.0:8000 \
  --access-logfile /opt/datapolis/logs/access.log \
  --error-logfile /opt/datapolis/logs/error.log \
  main:app

# Opción 3: Docker
docker-compose -f docker-compose.yml up -d backend

# Verificar que está corriendo
curl http://localhost:8000/api/v1/health
```

---

## 5. INSTALACIÓN DEL FRONTEND

### 5.1 Clonar y Configurar

```bash
# Clonar repositorio frontend
cd /opt/datapolis/frontend
git clone https://github.com/datapolis/frontend.git .

# Instalar dependencias
npm install

# Crear archivo .env.local
cat > /opt/datapolis/frontend/.env.local << 'EOF'
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
NEXT_PUBLIC_WS_URL=ws://localhost:8000/ws

# Maps
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your_google_maps_key

# Analytics
NEXT_PUBLIC_GA_ID=your_google_analytics_id

# Feature Flags
NEXT_PUBLIC_FEATURE_PAE=true
NEXT_PUBLIC_FEATURE_COMPLIANCE=true
NEXT_PUBLIC_FEATURE_DD=true

EOF
```

### 5.2 Build y Desarrollo

```bash
# Desarrollo local con hot reload
npm run dev
# Accesible en http://localhost:3000

# Build para producción
npm run build

# Iniciar servidor de producción
npm run start

# Verificar que está corriendo
curl http://localhost:3000
```

### 5.3 Configuración de Nginx (Producción)

```bash
# Instalar Nginx
sudo apt-get install -y nginx

# Crear configuración
sudo cat > /etc/nginx/sites-available/datapolis << 'EOF'
upstream backend {
    server localhost:8000;
}

upstream frontend {
    server localhost:3000;
}

server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    # Redirigir HTTP a HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;
    
    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    
    # Frontend
    location / {
        proxy_pass http://frontend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    
    # Backend API
    location /api/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # WebSocket
    location /ws {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
EOF

# Habilitar sitio
sudo ln -s /etc/nginx/sites-available/datapolis /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 6. CONFIGURACIÓN DE MÓDULOS

### 6.1 Módulo M01: Ficha Propiedad

```python
# /opt/datapolis/backend/routers/ficha_propiedad.py

from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from typing import List, Optional
from datetime import datetime
from decimal import Decimal
import uuid

router = APIRouter(prefix="/api/v1/properties", tags=["Property"])

# Configuración específica del módulo
MODULE_CONFIG = {
    "name": "Ficha Propiedad",
    "version": "1.0.0",
    "enabled": True,
    "cache_ttl": 3600,
    "validation_rules": {
        "address_required": True,
        "location_required": True,
        "owner_required": True
    }
}

@router.post("/properties")
async def create_property(
    data: dict,
    db: AsyncSession = Depends(get_db)
):
    """
    Crear nueva ficha de propiedad.
    
    Campos requeridos:
    - address: str
    - location: GeoJSON
    - owner_id: UUID
    - property_type: enum[house, apartment, land, commercial]
    """
    try:
        property_id = str(uuid.uuid4())
        
        # Validación
        if not data.get("address"):
            raise HTTPException(status_code=400, detail="Address required")
        
        # Crear registro
        property_record = {
            "id": property_id,
            "address": data["address"],
            "location": data["location"],
            "owner_id": data["owner_id"],
            "property_type": data["property_type"],
            "created_at": datetime.utcnow(),
            "status": "active"
        }
        
        # Guardar en BD
        # db.add(property_record)
        # await db.commit()
        
        # Publicar evento
        await publish_event("property.created", property_record)
        
        return {"status": "success", "property_id": property_id}
    
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))

@router.get("/properties/{property_id}")
async def get_property(property_id: str, db: AsyncSession = Depends(get_db)):
    """Obtener ficha de propiedad."""
    pass

@router.put("/properties/{property_id}")
async def update_property(property_id: str, data: dict, db: AsyncSession = Depends(get_db)):
    """Actualizar ficha de propiedad."""
    pass
```

### 6.2 Módulo M03: Credit Score

```python
# /opt/datapolis/backend/routers/credit_score.py

from fastapi import APIRouter
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
import numpy as np
import shap

router = APIRouter(prefix="/api/v1/credit", tags=["Credit"])

MODULE_CONFIG = {
    "name": "Credit Score",
    "version": "1.0.0",
    "model_path": "/models/credit_score_model.pkl",
    "features": [
        "income", "debt_ratio", "payment_history", 
        "credit_utilization", "age", "employment_years"
    ]
}

@router.post("/score")
async def calculate_credit_score(data: dict):
    """
    Calcular score crediticio con explicabilidad SHAP.
    
    Input:
    {
        "income": 50000,
        "debt_ratio": 0.35,
        "payment_history": 0.95,
        "credit_utilization": 0.45,
        "age": 35,
        "employment_years": 5
    }
    """
    try:
        # Preparar features
        features = np.array([[
            data["income"],
            data["debt_ratio"],
            data["payment_history"],
            data["credit_utilization"],
            data["age"],
            data["employment_years"]
        ]])
        
        # Normalizar
        scaler = StandardScaler()
        features_scaled = scaler.fit_transform(features)
        
        # Cargar modelo
        model = load_model(MODULE_CONFIG["model_path"])
        
        # Predecir
        score = model.predict_proba(features_scaled)[0][1] * 100
        
        # Explicabilidad SHAP
        explainer = shap.TreeExplainer(model)
        shap_values = explainer.shap_values(features_scaled)
        
        return {
            "score": round(score, 2),
            "risk_level": categorize_risk(score),
            "shap_explanation": shap_values.tolist(),
            "feature_importance": dict(zip(
                MODULE_CONFIG["features"],
                np.abs(shap_values[0]).tolist()
            ))
        }
    
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### 6.3 Módulo M04: Valorización

```python
# /opt/datapolis/backend/routers/valorizacion.py

from fastapi import APIRouter
from decimal import Decimal
from typing import Dict, List

router = APIRouter(prefix="/api/v1/valuation", tags=["Valuation"])

MODULE_CONFIG = {
    "name": "Valorización",
    "version": "1.0.0",
    "methods": ["comparable", "cost", "dcf", "hedonic"],
    "default_method": "comparable"
}

@router.post("/valuate")
async def valuate_property(data: dict):
    """
    Valorizar propiedad usando múltiples métodos.
    
    Input:
    {
        "property_id": "uuid",
        "method": "comparable|cost|dcf|hedonic",
        "comparable_properties": [...],
        "construction_cost": 1000000,
        "fcf": [100000, 110000, 120000],
        "discount_rate": 0.08
    }
    """
    
    results = {}
    
    # Método 1: Comparables
    if "comparable_properties" in data:
        comparable_value = calculate_comparable_value(
            data["comparable_properties"]
        )
        results["comparable"] = comparable_value
    
    # Método 2: Costo
    if "construction_cost" in data:
        cost_value = calculate_cost_value(
            data["construction_cost"],
            data.get("depreciation_rate", 0.02)
        )
        results["cost"] = cost_value
    
    # Método 3: DCF
    if "fcf" in data:
        dcf_value = calculate_dcf(
            data["fcf"],
            data.get("discount_rate", 0.08),
            data.get("terminal_growth", 0.02)
        )
        results["dcf"] = dcf_value
    
    # Método 4: Hedonic (si está disponible)
    if "hedonic_features" in data:
        hedonic_value = calculate_hedonic_value(
            data["hedonic_features"]
        )
        results["hedonic"] = hedonic_value
    
    # Promedio ponderado
    weighted_value = calculate_weighted_average(results)
    
    return {
        "property_id": data["property_id"],
        "valuations": results,
        "final_valuation": weighted_value,
        "confidence": 0.85,
        "timestamp": datetime.utcnow().isoformat()
    }

def calculate_comparable_value(comparables: List[Dict]) -> Dict:
    """Valorización por comparables."""
    prices = [c["price"] for c in comparables]
    areas = [c["area"] for c in comparables]
    
    price_per_m2 = [p/a for p, a in zip(prices, areas)]
    avg_price_m2 = sum(price_per_m2) / len(price_per_m2)
    
    return {
        "method": "comparable",
        "price_per_m2": avg_price_m2,
        "total_value": avg_price_m2 * comparables[0]["area"],
        "confidence": 0.90
    }

def calculate_dcf(fcf: List[float], discount_rate: float, terminal_growth: float) -> Dict:
    """Valorización por DCF."""
    pv_fcf = sum(f / ((1 + discount_rate) ** (i+1)) for i, f in enumerate(fcf))
    terminal_fcf = fcf[-1] * (1 + terminal_growth)
    terminal_value = terminal_fcf / (discount_rate - terminal_growth)
    pv_terminal = terminal_value / ((1 + discount_rate) ** len(fcf))
    
    total_value = pv_fcf + pv_terminal
    
    return {
        "method": "dcf",
        "pv_fcf": pv_fcf,
        "pv_terminal": pv_terminal,
        "total_value": total_value,
        "confidence": 0.75
    }
```

### 6.4 Configuración de Integración Entre Módulos

```yaml
# /opt/datapolis/backend/config/module_integration.yml

modules:
  m01_ficha_propiedad:
    name: "Ficha Propiedad"
    enabled: true
    dependencies: []
    publishes:
      - event: "property.created"
        subscribers: ["m04_valorizacion", "m05_arriendos", "m22_agora"]
      - event: "property.updated"
        subscribers: ["m04_valorizacion", "compliance_suite"]
    
  m03_credit_score:
    name: "Credit Score"
    enabled: true
    dependencies: []
    publishes:
      - event: "credit.scored"
        subscribers: ["m01_ficha_propiedad", "compliance_suite"]
    
  m04_valorizacion:
    name: "Valorización"
    enabled: true
    dependencies: ["m01_ficha_propiedad"]
    subscribes:
      - event: "property.created"
        handler: "on_property_created"
      - event: "property.updated"
        handler: "on_property_updated"
    publishes:
      - event: "property.valued"
        subscribers: ["m05_arriendos", "m06_plusvalia", "m22_agora"]
    
  m05_arriendos:
    name: "Arriendos"
    enabled: true
    dependencies: ["m01_ficha_propiedad", "m04_valorizacion"]
    subscribes:
      - event: "property.valued"
        handler: "on_property_valued"
    
  compliance_suite:
    name: "Compliance Suite"
    enabled: true
    dependencies: []
    subscribes:
      - event: "property.created"
        handler: "validate_property_compliance"
      - event: "credit.scored"
        handler: "validate_credit_compliance"
    publishes:
      - event: "compliance.checked"
        subscribers: ["m11_pae"]

event_bus:
  type: "redis"
  host: "localhost"
  port: 6379
  db: 0
  timeout: 30
  retry_policy:
    max_retries: 3
    backoff_factor: 2

cache:
  type: "redis"
  ttl: 3600
  key_prefix: "datapolis:"
```

---

## 7. GUÍAS DE INTEGRACIÓN INTER-MÓDULOS

### 7.1 Patrón de Publicación de Eventos

```python
# /opt/datapolis/backend/core/event_bus.py

import redis
import json
from typing import Dict, Any, Callable
from datetime import datetime

class EventBus:
    def __init__(self, redis_url: str):
        self.redis = redis.from_url(redis_url)
        self.subscribers: Dict[str, list] = {}
    
    async def publish(self, event_type: str, data: Dict[str, Any]):
        """Publicar evento en el bus."""
        event_payload = {
            "type": event_type,
            "data": data,
            "timestamp": datetime.utcnow().isoformat(),
            "source": "datapolis"
        }
        
        # Publicar en Redis
        channel = f"datapolis:events:{event_type}"
        self.redis.publish(channel, json.dumps(event_payload))
        
        print(f"Event published: {event_type}")
    
    async def subscribe(self, event_type: str, handler: Callable):
        """Suscribirse a un evento."""
        channel = f"datapolis:events:{event_type}"
        
        if event_type not in self.subscribers:
            self.subscribers[event_type] = []
        
        self.subscribers[event_type].append(handler)
        
        print(f"Subscribed to: {event_type}")
    
    async def listen(self):
        """Escuchar eventos en el bus."""
        pubsub = self.redis.pubsub()
        
        # Suscribirse a todos los canales
        for event_type in self.subscribers:
            channel = f"datapolis:events:{event_type}"
            pubsub.subscribe(channel)
        
        for message in pubsub.listen():
            if message['type'] == 'message':
                event_data = json.loads(message['data'])
                event_type = event_data['type']
                
                # Ejecutar handlers
                if event_type in self.subscribers:
                    for handler in self.subscribers[event_type]:
                        await handler(event_data['data'])

# Uso en módulos
event_bus = EventBus(settings.REDIS_URL)

# En M04 Valorización - Publicar evento
@router.post("/valuate")
async def valuate_property(data: dict):
    # ... lógica de valorización ...
    await event_bus.publish("property.valued", {
        "property_id": data["property_id"],
        "valuation": result
    })

# En M05 Arriendos - Suscribirse a evento
@app.on_event("startup")
async def startup():
    await event_bus.subscribe("property.valued", on_property_valued)

async def on_property_valued(data: dict):
    """Handler cuando una propiedad es valorizada."""
    property_id = data["property_id"]
    valuation = data["valuation"]
    
    # Actualizar yield de arriendos
    await update_rental_yield(property_id, valuation)
```

### 7.2 Patrón de Compartir Datos (Ontología)

```python
# /opt/datapolis/backend/core/ontology.py

from pydantic import BaseModel
from typing import Optional, Dict, Any
from datetime import datetime
from decimal import Decimal
import json

class PropertyOntology(BaseModel):
    """Ontología compartida para propiedades."""
    
    # Identificación
    id: str
    address: str
    location: Dict[str, Any]  # GeoJSON
    
    # Datos básicos
    property_type: str  # house, apartment, land, commercial
    area_m2: Decimal
    year_built: int
    
    # Valorización
    valuation: Optional[Dict] = None
    valuation_method: Optional[str] = None
    valuation_date: Optional[datetime] = None
    
    # Financiero
    rental_income: Optional[Decimal] = None
    operating_expenses: Optional[Decimal] = None
    net_yield: Optional[float] = None
    
    # Compliance
    compliance_status: Optional[str] = None  # compliant, warning, critical
    normative_violations: Optional[list] = None
    
    # Riesgos
    seismic_risk: Optional[float] = None
    flood_risk: Optional[float] = None
    
    class Config:
        json_schema_extra = {
            "example": {
                "id": "prop-123",
                "address": "Av. Providencia 1234, Santiago",
                "location": {
                    "type": "Point",
                    "coordinates": [-70.5, -33.4]
                },
                "property_type": "apartment",
                "area_m2": 120,
                "year_built": 2015,
                "valuation": {
                    "comparable": 450000000,
                    "dcf": 480000000,
                    "final": 465000000
                },
                "rental_income": 1500000,
                "net_yield": 0.048
            }
        }

# Uso en módulos
def serialize_property_to_ontology(db_record) -> PropertyOntology:
    """Convertir registro de BD a ontología."""
    return PropertyOntology(
        id=db_record.id,
        address=db_record.address,
        location=db_record.location,
        property_type=db_record.property_type,
        area_m2=db_record.area_m2,
        year_built=db_record.year_built,
        valuation=db_record.valuation,
        rental_income=db_record.rental_income,
        compliance_status=db_record.compliance_status
    )

def deserialize_ontology_to_db(ontology: PropertyOntology):
    """Convertir ontología a formato de BD."""
    return {
        "id": ontology.id,
        "address": ontology.address,
        "location": ontology.location,
        "property_type": ontology.property_type,
        "area_m2": ontology.area_m2,
        "year_built": ontology.year_built,
        "valuation": ontology.valuation,
        "rental_income": ontology.rental_income,
        "compliance_status": ontology.compliance_status
    }
```

### 7.3 Patrón de Flujo de Datos Multi-Módulo

```python
# /opt/datapolis/backend/workflows/property_workflow.py

from typing import Dict, Any
from datetime import datetime

class PropertyWorkflow:
    """Orquestador de flujo de datos para propiedades."""
    
    def __init__(self, event_bus, db):
        self.event_bus = event_bus
        self.db = db
    
    async def process_new_property(self, property_data: Dict[str, Any]):
        """
        Flujo completo de procesamiento de nueva propiedad:
        1. Crear ficha (M01)
        2. Valorizar (M04)
        3. Calcular arriendos (M05)
        4. Analizar plusvalía (M06)
        5. Validar compliance (Compliance Suite)
        6. Evaluar riesgos (M17)
        7. Actualizar mapa territorial (M22)
        """
        
        try:
            # Paso 1: Crear ficha de propiedad
            property_id = await self._create_property(property_data)
            print(f"✓ Propiedad creada: {property_id}")
            
            # Paso 2: Valorizar propiedad
            valuation = await self._valuate_property(property_id, property_data)
            print(f"✓ Propiedad valorizada: ${valuation['final_valuation']}")
            
            # Paso 3: Calcular arriendos
            rental_yield = await self._calculate_rental_yield(property_id, valuation)
            print(f"✓ Yield de arriendos: {rental_yield}%")
            
            # Paso 4: Analizar plusvalía
            capital_gains = await self._analyze_capital_gains(property_id)
            print(f"✓ Plusvalía analizada")
            
            # Paso 5: Validar compliance
            compliance_status = await self._validate_compliance(property_id)
            print(f"✓ Compliance validado: {compliance_status}")
            
            # Paso 6: Evaluar riesgos sísmicos
            seismic_risk = await self._evaluate_seismic_risk(property_id)
            print(f"✓ Riesgo sísmico evaluado: {seismic_risk}")
            
            # Paso 7: Actualizar mapa territorial
            await self._update_territorial_map(property_id)
            print(f"✓ Mapa territorial actualizado")
            
            return {
                "status": "success",
                "property_id": property_id,
                "valuation": valuation,
                "rental_yield": rental_yield,
                "compliance_status": compliance_status,
                "seismic_risk": seismic_risk
            }
        
        except Exception as e:
            print(f"✗ Error en workflow: {str(e)}")
            raise
    
    async def _create_property(self, data: Dict) -> str:
        """Crear propiedad en M01."""
        # Llamar a M01 API
        pass
    
    async def _valuate_property(self, property_id: str, data: Dict) -> Dict:
        """Valorizar propiedad en M04."""
        # Llamar a M04 API
        pass
    
    # ... más métodos ...
```

---

## 8. TESTING Y VALIDACIÓN

### 8.1 Unit Tests

```bash
# Instalar pytest
pip install pytest pytest-asyncio pytest-cov

# Crear estructura de tests
mkdir -p /opt/datapolis/backend/tests/{unit,integration,e2e}

# Ejecutar tests
pytest tests/unit -v --cov=routers --cov-report=html

# Verificar cobertura
coverage report
```

### 8.2 Integration Tests

```python
# /opt/datapolis/backend/tests/integration/test_property_workflow.py

import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

@pytest.mark.asyncio
async def test_create_property():
    """Test creación de propiedad."""
    async with AsyncClient(app=app, base_url="http://test") as client:
        response = await client.post(
            "/api/v1/properties",
            json={
                "address": "Test Address",
                "location": {"type": "Point", "coordinates": [-70.5, -33.4]},
                "owner_id": "test-owner",
                "property_type": "apartment"
            }
        )
        assert response.status_code == 200
        assert "property_id" in response.json()

@pytest.mark.asyncio
async def test_property_valuation_workflow():
    """Test flujo completo de valorización."""
    # Crear propiedad
    property_response = await create_test_property()
    property_id = property_response["property_id"]
    
    # Valorizar
    valuation_response = await valuate_property(property_id)
    assert valuation_response["status"] == "success"
    assert "final_valuation" in valuation_response
    
    # Verificar que se publicó evento
    assert await check_event_published("property.valued", property_id)
```

### 8.3 Performance Testing

```bash
# Instalar herramientas de load testing
pip install locust

# Crear test de carga
cat > /opt/datapolis/backend/tests/load/locustfile.py << 'EOF'
from locust import HttpUser, task, between

class DatapolisUser(HttpUser):
    wait_time = between(1, 3)
    
    @task
    def get_property(self):
        self.client.get("/api/v1/properties/test-id")
    
    @task
    def create_property(self):
        self.client.post("/api/v1/properties", json={
            "address": "Test",
            "location": {"type": "Point", "coordinates": [-70.5, -33.4]},
            "owner_id": "test",
            "property_type": "apartment"
        })
EOF

# Ejecutar test de carga
locust -f tests/load/locustfile.py --host=http://localhost:8000
```

---

## 9. DEPLOYMENT EN PRODUCCIÓN

### 9.1 Docker Compose (Desarrollo/Staging)

```yaml
# /opt/datapolis/docker-compose.yml

version: '3.8'

services:
  postgres:
    image: postgis/postgis:16-3.4
    environment:
      POSTGRES_USER: datapolis_user
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: datapolis_db
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U datapolis_user"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql+asyncpg://datapolis_user:${POSTGRES_PASSWORD}@postgres:5432/datapolis_db
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379/0
      SECRET_KEY: ${SECRET_KEY}
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    volumes:
      - ./backend:/app
      - backend_logs:/app/logs

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    environment:
      NEXT_PUBLIC_API_URL: http://backend:8000/api/v1
    ports:
      - "3000:3000"
    depends_on:
      - backend

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - backend
      - frontend

volumes:
  postgres_data:
  redis_data:
  backend_logs:

networks:
  default:
    name: datapolis_network
```

### 9.2 Kubernetes (Producción)

```yaml
# /opt/datapolis/k8s/deployment.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: datapolis-backend
  namespace: datapolis
spec:
  replicas: 3
  selector:
    matchLabels:
      app: datapolis-backend
  template:
    metadata:
      labels:
        app: datapolis-backend
    spec:
      containers:
      - name: backend
        image: datapolis/backend:4.0.0
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: datapolis-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: datapolis-secrets
              key: redis-url
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /api/v1/health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /api/v1/ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: datapolis-backend-service
  namespace: datapolis
spec:
  selector:
    app: datapolis-backend
  ports:
  - protocol: TCP
    port: 8000
    targetPort: 8000
  type: LoadBalancer
```

### 9.3 CI/CD con GitHub Actions

```yaml
# /opt/datapolis/.github/workflows/deploy.yml

name: Deploy DATAPOLIS

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgis/postgis:16-3.4
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
    
    - name: Install dependencies
      run: |
        cd backend
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Run tests
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/datapolis_db
        REDIS_URL: redis://localhost:6379/0
      run: |
        cd backend
        pytest tests/ -v --cov=routers
    
    - name: Build Docker image
      run: |
        docker build -t datapolis/backend:${{ github.sha }} ./backend
        docker tag datapolis/backend:${{ github.sha }} datapolis/backend:latest
    
    - name: Push to registry
      run: |
        echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
        docker push datapolis/backend:${{ github.sha }}
        docker push datapolis/backend:latest
    
    - name: Deploy to Kubernetes
      if: github.ref == 'refs/heads/main'
      run: |
        kubectl set image deployment/datapolis-backend \
          backend=datapolis/backend:${{ github.sha }} \
          --record
```

---

## 10. TROUBLESHOOTING

### 10.1 Problemas Comunes

| Problema | Síntoma | Solución |
|----------|---------|----------|
| **BD no conecta** | `psycopg2.OperationalError` | Verificar `DATABASE_URL`, credenciales, firewall |
| **Redis timeout** | `redis.exceptions.ConnectionError` | Verificar `REDIS_URL`, puerto 6379 abierto |
| **Módulo no carga** | `ImportError: No module named...` | `pip install -r requirements.txt` |
| **Evento no se publica** | Módulos no se sincronizan | Verificar conexión Redis, logs de event_bus |
| **Performance lenta** | Queries lentas | Crear índices en PostgreSQL, revisar EXPLAIN PLAN |
| **Frontend no carga** | `ERR_CONNECTION_REFUSED` | Verificar `NEXT_PUBLIC_API_URL`, backend running |

### 10.2 Logs y Debugging

```bash
# Ver logs del backend
tail -f /opt/datapolis/logs/backend.log

# Ver logs de Docker
docker-compose logs -f backend

# Ver logs de PostgreSQL
sudo tail -f /var/log/postgresql/postgresql-16-main.log

# Ver logs de Redis
redis-cli MONITOR

# Debugging con pdb
python -m pdb /opt/datapolis/backend/main.py

# Profiling de performance
python -m cProfile -s cumulative main.py
```

### 10.3 Checklist de Validación

```bash
#!/bin/bash
# /opt/datapolis/scripts/validate_installation.sh

echo "=== DATAPOLIS Installation Validation ==="

# 1. Verificar Python
python3.11 --version && echo "✓ Python 3.11" || echo "✗ Python 3.11"

# 2. Verificar PostgreSQL
psql -U datapolis_user -d datapolis_db -c "SELECT 1" && echo "✓ PostgreSQL" || echo "✗ PostgreSQL"

# 3. Verificar Redis
redis-cli ping && echo "✓ Redis" || echo "✗ Redis"

# 4. Verificar Backend
curl http://localhost:8000/api/v1/health && echo "✓ Backend" || echo "✗ Backend"

# 5. Verificar Frontend
curl http://localhost:3000 && echo "✓ Frontend" || echo "✗ Frontend"

# 6. Verificar módulos
python3.11 -c "from routers import *; print('✓ Módulos')" || echo "✗ Módulos"

echo "=== Validation Complete ==="
```

---

## CONCLUSIÓN

Este manual proporciona una guía exhaustiva para la instalación, configuración e integración de DATAPOLIS v4.0 en un entorno local o de producción. Siguiendo estos pasos, se garantiza una implementación correcta y operativa de la plataforma con todos sus módulos integrados.

**Próximos pasos recomendados:**
1. Completar la instalación siguiendo cada sección
2. Ejecutar los tests de validación
3. Configurar el monitoreo y alertas
4. Documentar configuraciones específicas del entorno
5. Establecer procedimientos de backup y recuperación

**Soporte técnico:** Para consultas técnicas adicionales, referirse a la documentación de API en `/api/v1/docs` (Swagger UI).

---

**Documento Generado:** 26 de Febrero de 2026  
**Versión:** 1.0  
**Clasificación:** Documentación Técnica Operativa  
**Autor:** Análisis Técnico Automatizado
