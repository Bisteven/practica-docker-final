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

### capturas
- Docker compose ps
<img width="503" height="253" alt="image" src="https://github.com/user-attachments/assets/084635eb-924b-48d4-9ab8-a01cc48ff662" />
- frontend
<img width="1408" height="757" alt="image" src="https://github.com/user-attachments/assets/73721715-9bba-49b1-bb75-753fdf4c430b" />
- backend api
<img width="1431" height="718" alt="image" src="https://github.com/user-attachments/assets/07f05496-9054-422d-af4a-74aac8654d35" />
- Api seed
<img width="339" height="102" alt="image" src="https://github.com/user-attachments/assets/4ced485a-4d8c-480c-8c93-083d8959222f" />
- Estructura backend
<img width="268" height="390" alt="image" src="https://github.com/user-attachments/assets/d3ed176e-174d-41ab-a905-8692b4a70a65" />
- Estructura Frontend
<img width="287" height="416" alt="image" src="https://github.com/user-attachments/assets/5bc5b7d8-b75a-4f88-8eb3-87a2bc0b85c3" />
- Api funcionando
<img width="1065" height="574" alt="image" src="https://github.com/user-attachments/assets/983d8f31-9121-40cd-99c3-0c74987d9acc" />
- docker compose up --build -d
<img width="544" height="205" alt="image" src="https://github.com/user-attachments/assets/0162f239-c9c2-41e1-bbef-e057629b2cb3" />




