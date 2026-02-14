# 🚀 Evolution API "Add-On" for existing Odoo/Traefik Stack

Este repositorio está diseñado para **añadir** el servicio **Evolution API** a un servidor que **YA TIENE** Odoo, PostgreSQL y Traefik corriendo (como el tuyo).

Reutilizará:
1.  Tu **Traefik** existente (vía red `traefik-network`).
2.  Tu **PostgreSQL** existente (vía red `postgres-network`).

## 🛠️ Instrucciones de Instalación

1.  **Clonar:**
    ```bash
    git clone https://github.com/Mimbex/odoo-evolution-api evolution-addon
    cd evolution-addon
    cp .env.example .env
    nano .env
    ```

2.  **Configurar `.env`:**
    *   `SERVER_URL`: Tu dominio (ej. `https://whatsapp.midominio.com`).
    *   `AUTHENTICATION_API_KEY`: Una contraseña fuerte.
    *   `POSTGRES_HOST`: El nombre del contenedor o servicio de tu Postgres actual (ej. `postgresql-postgresql-1` o simplemente `db`).

3.  **Preparar Base de Datos (Solo una vez):**
    Entra a tu contenedor de Postgres existente y crea la base `evolution_db`:
    ```bash
    docker exec -it postgresql-postgresql-1 psql -U odoo -c "CREATE DATABASE evolution_db;"
    ```

4.  **Lanzar:**
    ```bash
    docker compose up -d
    ```

El contenedor se conectará a tu red `traefik-network` y tu red `postgres-network` automáticamente.

## ⚠️ Nota Importante sobre Redes

Este `docker-compose.yml` asume que tus redes se llaman:
*   `traefik-network`
*   `postgres-network`

Si tienen otros nombres (ej. `web`, `frontend`, `backend`), edita el final del `docker-compose.yml`:

```yaml
networks:
  proxy:
    external: true
    name: NOMBRE_REAL_DE_TU_RED_TRAEFIK
  postgres:
    external: true
    name: NOMBRE_REAL_DE_TU_RED_POSTGRES
```
