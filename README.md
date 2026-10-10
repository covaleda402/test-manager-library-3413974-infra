# Manager Library — Infraestructura y Orquestación

> **Prueba técnica · Ficha ADSO 3413974**  
> Orquestación de servicios en contenedores **Docker** mediante **Docker Compose**, configuración de redes, almacenamiento persistente y base de datos relacional PostgreSQL.

---

## 1. Topología de Servicios

La solución define 4 contenedores interconectados bajo una red puente privada (`library_net`):

| Servicio | Contenedor | Imagen / Base | Puerto Host | Descripción |
|---|---|---|---|---|
| `db` | `library_db` | `postgres:16-alpine` | `5432` | Motor PostgreSQL 16 que arranca vacío, sin scripts DDL manuales (cumpliendo Art. V y ADR-001). Healthcheck activo en `pg_isready`. |
| `api` | `library_api` | `php:8.2-cli` | `8000` | Backend en Laravel 11. Ejecuta migraciones, seeding inicial y atiende el contrato REST en `/api/v1`. Monta volumen persistente para portadas. |
| `app` | `library_app` | `nginx:alpine` | `5173` | Frontend SPA en React 18 / Vite compilado y servido por Nginx con enrutamiento HTML5. |
| `page` | `library_page` | `nginx:alpine` | `8080` | Sitio web público estático de presentación de la biblioteca. |

---

## 2. Volúmenes y Persistencia

1. **`db_data`**: Volumen nombrado de Docker que almacena los datos de PostgreSQL de manera persistente en `./data/postgres`.
2. **`covers_data`**: Volumen que persiste los archivos binarios de las portadas de libros en `/var/covers` dentro del contenedor del backend.

---

## 3. Principio Innegociable de Base de Datos (ADR-001)

El contenedor de base de datos **no** contiene scripts de inicialización SQL (`/docker-entrypoint-initdb.d` está deliberadamente vacío).  
Toda la estructura relacional, claves foráneas, restricciones `CHECK` y la semilla de las 5 categorías oficiales son propiedad exclusiva y se aplican mediante las migraciones de la API en el arranque (`php artisan migrate --force`).

---

## 4. Instrucciones de Despliegue

### 4.1 Levantar todo el stack
Desde la raíz del proyecto o desde el directorio de infraestructura:

```bash
docker compose -f docker-compose.yml up -d --build
```

### 4.2 Verificar estado de los servicios
```bash
docker compose -f docker-compose.yml ps
```

Todos los servicios deben figurar en estado `Up` (y `library_db` como `healthy`).

### 4.3 URLs de acceso local
* **Aplicación Web (React):** [http://localhost:5173](http://localhost:5173)
* **API REST (Laravel):** [http://localhost:8000/api/v1/health](http://localhost:8000/api/v1/health)
* **Sitio Web Público:** [http://localhost:8080](http://localhost:8080)
* **Motor PostgreSQL:** `localhost:5432` (Usuario: `library`, BD: `library`)

### 4.4 Detener los servicios
```bash
docker compose -f docker-compose.yml down
```
