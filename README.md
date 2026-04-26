# 🐳 Example Voting App

> Guía práctica para levantar una aplicación distribuida con **Docker** y **Docker Compose**

![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![.NET](https://img.shields.io/badge/.NET-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-5FA04E?style=for-the-badge&logo=nodedotjs&logoColor=white)

---

## 📖 Descripción

En esta práctica aprenderás a:

1. Levantar una aplicación distribuida de **5 servicios** usando Docker de forma manual.
2. Simplificar todo el proceso con un único archivo `docker-compose.yaml`.

---

## 🏗️ Arquitectura

```
                  ┌───────────┐
  ┌──────┐       │           │       ┌────────┐
  │ Vote ├──────►│   Redis   │◄──────┤ Worker │
  │ :5000│       │           │       │ (.NET) │
  └──────┘       └───────────┘       └───┬────┘
  (Python)                               │
                                         ▼
                 ┌───────────┐       ┌────────┐
                 │  Result   │◄──────┤  DB    │
                 │  :5001    │       │(Postgres)
                 └───────────┘       └────────┘
                  (Node.js)
```

| Servicio | Tecnología | Puerto | Función |
|----------|------------|--------|---------|
| **vote** | Python (Flask) | `5000` | Interfaz web para votar |
| **redis** | Redis | — | Cola de mensajes temporal |
| **worker** | .NET | — | Procesa votos y los persiste |
| **db** | PostgreSQL 15 | — | Almacenamiento persistente |
| **result** | Node.js | `5001` | Muestra resultados en tiempo real |

---

## 📋 Requisitos previos

- [Docker](https://docs.docker.com/get-docker/) instalado
- [Docker Compose](https://docs.docker.com/compose/install/) (incluido en Docker Desktop)
- Git

---

## 🔹 Parte 1 — Docker (modo manual)

En esta sección levantaremos cada contenedor por separado para entender cómo se conectan entre sí.

### 1. Clonar el repositorio

```bash
git clone https://github.com/dockersamples/example-voting-app
cd example-voting-app
```

### 2. Construir y ejecutar el servicio `vote`

```bash
docker build -t voting-app ./vote
docker run -p 5000:80 voting-app
```

👉 Abre [http://localhost:5000](http://localhost:5000) en tu navegador.

> ⚠️ Al intentar votar verás un error porque **Redis aún no existe**.

### 3. Levantar Redis y reconectar `vote`

```bash
docker run -d --name redis redis
docker run -p 5000:80 --link redis:redis voting-app
```

### 4. Levantar PostgreSQL

```bash
docker run -d \
  -e POSTGRES_HOST_AUTH_METHOD=trust \
  --name db \
  postgres:15-alpine
```

### 5. Construir y ejecutar el `worker`

```bash
docker build -t worker-app ./worker
docker run -d \
  --link redis:redis \
  --link db:db \
  worker-app
```

> 📌 El worker mueve los votos desde Redis hacia PostgreSQL.

### 6. Construir y ejecutar `result`

```bash
docker build -t result-app ./result
docker run -d -p 5001:80 --link db:db result-app
```

👉 Abre [http://localhost:5001](http://localhost:5001) para ver los resultados.

### 7. Detener todos los contenedores

```bash
docker stop $(docker ps -q)
```

---

## 🔹 Parte 2 — Docker Compose (modo automatizado)

Docker Compose permite definir y levantar **todos los servicios con un solo comando** a partir de un archivo YAML.

### 1. Crear `docker-compose.yaml`

```yaml
version: "3.9"

services:
  redis:
    image: redis:latest

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust

  vote:
    build: ./vote
    ports:
      - "5000:80"
    depends_on:
      - redis

  worker:
    build: ./worker
    depends_on:
      - redis
      - db

  result:
    build: ./result
    ports:
      - "5001:80"
    depends_on:
      - db
```

### 2. Levantar el stack completo

```bash
docker compose up -d
```

### 3. Verificar los contenedores

```bash
docker compose ps
```

### 4. Ver los logs en tiempo real

```bash
docker compose logs -f
```

### 5. Detener y eliminar todo

```bash
docker compose down
```

---

## 🆚 Docker manual vs Docker Compose

| | Docker manual | Docker Compose |
|---|---|---|
| **Comandos necesarios** | Uno por contenedor (~10) | Un solo comando |
| **Networking** | `--link` (deprecado) | Red interna automática |
| **Orden de inicio** | Manual | `depends_on` |
| **Reproducibilidad** | Baja | Alta (archivo versionable) |

---

## 💡 Buenas prácticas

- Usa `docker compose` (con espacio) en lugar del antiguo `docker-compose`.
- Evita `--link`, ya que está **deprecado**. Compose crea una red interna automáticamente.
- Define `depends_on` para controlar el orden de arranque.
- Usa `build:` en el Compose para construir las imágenes directamente en lugar de hacerlo por separado.
- Agrega `volumes` para persistir los datos de PostgreSQL en producción.

---

## 🚀 Conclusión

- **Docker** te da control granular sobre cada contenedor.
- **Docker Compose** simplifica la orquestación de múltiples servicios.
- Para aplicaciones con más de un contenedor, **Compose es la forma recomendada** de trabajar.

---

## 📚 Recursos

- [Repositorio original — example-voting-app](https://github.com/dockersamples/example-voting-app)
- [Documentación de Docker Compose](https://docs.docker.com/compose/)
- [Docker Compose file reference](https://docs.docker.com/compose/compose-file/)
