# practica-docker-final

## Arquitectura

- **Frontend**: Angular (compilado y servido con Nginx). Expone `http://localhost:${FRONTEND_PUBLISH_PORT}`.
  - Ruteo SPA (Angular) en Nginx.
  - Proxy en Nginx:
    - `/api/*` → `backend:3000/api/*`
    - `/socket.io/*` → `backend:3000/socket.io/*` (websocket)
- **Backend**: NestJS + TypeORM + PostgreSQL. Expone `http://localhost:${BACKEND_PUBLISH_PORT}/api`.
- **Base de datos**: PostgreSQL. Persistencia con volumen `postgres_data`.

Estructura usada (del repo referenciado):

- `tesloshop-app/teslo-shop` (backend)
- `tesloshop-app/angular-tesloshop` (frontend)

## Requisitos

- Docker Desktop (Windows)
- Docker Compose v2 (`docker compose ...`)

## Variables de entorno

Se usa el archivo `.env` en la raíz (ya incluido). Puedes cambiar:

- **DB**: `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`, `POSTGRES_PUBLISH_PORT`
- **Backend**: `BACKEND_PUBLISH_PORT`, `JWT_SECRET`, `HOST_API`
- **Frontend**: `FRONTEND_PUBLISH_PORT`

## Explicación de servicios (docker-compose)

- **db (PostgreSQL)**:
  - **Imagen**: `postgres:16-alpine`
  - **Puerto**: `localhost:${POSTGRES_PUBLISH_PORT}` → `db:5432`
  - **Volumen**: `postgres_data` (persistencia de datos)
  - **Healthcheck**: `pg_isready` para asegurar que el backend espere a la BD

- **backend (NestJS)**:
  - **Build**: `./tesloshop-app/teslo-shop/Dockerfile` (multi-stage)
  - **Puerto**: `localhost:${BACKEND_PUBLISH_PORT}` → `backend:${PORT}` (por defecto 3000)
  - **Dependencia**: `depends_on` con `db` saludable
  - **Volumen**: `backend_static` (para `static/products` y archivos subidos)

- **frontend (Angular + Nginx)**:
  - **Build**: `./tesloshop-app/angular-tesloshop/Dockerfile` (multi-stage)
  - **Puerto**: `localhost:${FRONTEND_PUBLISH_PORT}` → `frontend:80`
  - **Proxy**: Nginx redirige `/api` y `/socket.io` hacia el servicio `backend`

## Nota de puertos (Windows / Wampserver)

Si tienes **Wampserver/Apache** usando el puerto **80**, el frontend no podrá publicar en 80.
Solución recomendada: en `.env` usar `FRONTEND_PUBLISH_PORT=8080` y abrir `http://localhost:8080`.

## Fase 2 – Construcción de imágenes (Dockerfile)

### Backend

```bash
cd tesloshop-app/teslo-shop
docker build -t backend .
```

### Frontend

```bash
cd ../angular-tesloshop
docker build -t frontend .
```

## Fase 3 – Orquestación con Docker Compose

Desde la raíz del repo:

```bash
docker compose up --build -d
```

Para ver estado:

```bash
docker compose ps
docker compose logs -f --tail=200
```

Para bajar todo:

```bash
docker compose down
```

## Fase 4 – Validación del sistema

### 1) Contenedores activos

```bash
docker compose ps
```

Debes ver `teslo-db`, `teslo-backend`, `teslo-frontend` en estado `running`.

### 2) Probar frontend

- Abrir `http://localhost:${FRONTEND_PUBLISH_PORT}`

### 3) Probar API backend

- Swagger: `http://localhost:${BACKEND_PUBLISH_PORT}/api`
- Health rápida (ejemplo): `http://localhost:${BACKEND_PUBLISH_PORT}/api/seed` (ver siguiente punto)

### 4) Seed / datos de prueba

El backend incluye endpoint de seed:

- GET `http://localhost:${BACKEND_PUBLISH_PORT}/api/seed`

## Fase 5 – Evidencias (capturas sugeridas)

- `docker compose ps` mostrando contenedores en ejecución
- Navegador con frontend cargando en `http://localhost:${FRONTEND_PUBLISH_PORT}` (ej. `8080`)
- Swagger del backend en `http://localhost:${BACKEND_PUBLISH_PORT}/api`
- Ejecución del seed (respuesta de `GET /api/seed`)

Sugerencia: guardar las capturas en una carpeta `evidencias/` (no obligatoria) con nombres como:

- `01-compose-ps.png`
- `02-frontend.png`
- `03-swagger.png`
- `04-seed.png`