# Evolution API Integration for Odoo (Existing Stack)

This repository serves as a template on **how to integrate Evolution API** into an **EXISTING Odoo Deployment** (which already runs Odoo, PostgreSQL, and Traefik).

The goal is to **reuse resources** (PostgreSQL, Traefik Network) instead of creating a duplicate stack.

---

## 🏗️ Architecture

Instead of launching a separate Evolution stack, we simply add:
1.  **Evolution API Service:** Connects to your existing Traefik network.
2.  **Redis Service:** Lightweight cache required by Evolution.
3.  **Database Integration:** We reuse your existing PostgreSQL container by creating a dedicated database (`evolution_db`) for the API.

---

## 🚀 How to Add to Your Docker Compose

Copy the following services into your main `docker-compose.yml`:

### 1. Add Services (Evolution API + Redis)

```yaml
version: '3.8'

services:
  # ... your existing odoo, postgres, traefik ...

  evolution_api:
    image: atendai/evolution-api:v2.1.1
    container_name: evolution_api
    restart: always
    volumes:
      - evolution_store:/evolution/store
    environment:
      - SERVER_URL=https://whatsapp.yourdomain.com
      - AUTHENTICATION_API_KEY=CHANGE_THIS_TO_STRONG_PASSWORD
      - DATABASE_ENABLED=true
      - DATABASE_PROVIDER=postgresql
      # Point to your EXISTING postgres service (e.g., 'db' or 'postgres')
      - DATABASE_CONNECTION_URI=postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/evolution_db
      - DATABASE_CLIENT_NAME=evolution_v2
      - CACHE_REDIS_ENABLED=true
      - CACHE_REDIS_URI=redis://evolution_redis:6379/0
      - CACHE_REDIS_PREFIX=evolution:
      - DEL_INSTANCE_ON_LOGOUT=false
      - QB_LIMIT=30
    networks:
      - default
      - proxy # Ensure this matches your Traefik network
    depends_on:
      - db # Your existing Postgres service
      - evolution_redis
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.evolution.rule=Host(`whatsapp.yourdomain.com`)"
      - "traefik.http.routers.evolution.entrypoints=websecure"
      - "traefik.http.routers.evolution.tls.certresolver=myresolver"
      - "traefik.http.services.evolution.loadbalancer.server.port=8080"

  evolution_redis:
    image: redis:7-alpine
    container_name: evolution_redis
    restart: always
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - evolution_redis_data:/data
    networks:
      - default

volumes:
  # ... existing volumes ...
  evolution_store:
  evolution_redis_data:

networks:
  proxy:
    external: true
```

### 2. Prepare the Database

Since we are reusing your existing Postgres, we need to create the `evolution_db` database inside it.

Run this command **once** on your existing Postgres container:

```bash
# Replace 'postgres_container_name' generally 'db' or 'odoo-db'
docker exec -it postgres_container_name psql -U odoo -c "CREATE DATABASE evolution_db;"
```

### 3. Deploy

```bash
docker compose up -d
```

---

## ✅ Result

- **Odoo** runs as usual.
- **Evolution API** runs alongside Odoo, using the SAME database server (saving RAM/CPU).
- **Traefik** routes `odoo.yourdomain.com` to Odoo and `whatsapp.yourdomain.com` to Evolution API.
