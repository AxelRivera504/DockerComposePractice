# 🐳 Example Voting App – Guía práctica Docker & Docker Compose

## 📌 Descripción
En este ejercicio aprenderás a levantar una aplicación distribuida usando Docker paso a paso y luego simplificar todo el proceso con Docker Compose.

---

# 🔹 Parte 1: Uso de Docker (manual)

## 1. Clonar el repositorio

```bash
git clone https://github.com/dockersamples/example-voting-app
2. Construir imagen del servicio vote
docker build -t voting-app "D:\Cursos\DevOps\KodeKloud\Docker\GitHub\Nueva carpeta\example-voting-app\vote"

📌 Nota:
Las rutas con espacios deben ir entre comillas.

3. Ejecutar contenedor de vote
docker run -p 5000:80 voting-app

👉 Acceder en: http://localhost:5000

⚠️ Verás error al votar porque aún no existe Redis.

4. Ejecutar Redis
docker run -d --name redis redis
5. Ejecutar vote conectado a Redis
docker run -p 5000:80 --link redis:redis voting-app
6. Ejecutar PostgreSQL
docker run -d \
-e POSTGRES_HOST_AUTH_METHOD=trust \
--name db \
postgres:15-alpine
7. Construir imagen del worker (.NET)
docker build -t worker-app "D:\Cursos\DevOps\KodeKloud\Docker\GitHub\Nueva carpeta\example-voting-app\worker"
8. Ejecutar worker
docker run -d \
--link redis:redis \
--link db:db \
worker-app

📌 Este servicio mueve datos de Redis a PostgreSQL.

9. Construir imagen de result
docker build -t result-app "D:\Cursos\DevOps\KodeKloud\Docker\GitHub\Nueva carpeta\example-voting-app\result"
10. Ejecutar result app
docker run -d -p 5001:80 --link db:db result-app

👉 Acceder en: http://localhost:5001

🛑 Detener contenedores
docker stop <id1> <id2> <id3>

Ejemplo:

docker stop b9 67 28 ce 43
🔹 Parte 2: Docker Compose (automatizado)
📌 ¿Qué es Docker Compose?

# 🔹 Parte 2: Uso de Docker Compose

Permite levantar múltiples contenedores con un solo comando usando un archivo YAML.

1. Crear archivo docker-compose.yaml
version: "3.9"

services:
  redis:
    image: redis:latest

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_HOST_AUTH_METHOD: trust

  vote:
    image: voting-app
    ports:
      - "5000:80"
    depends_on:
      - redis

  worker:
    image: worker-app
    depends_on:
      - redis
      - db

  result:
    image: result-app
    ports:
      - "5001:80"
    depends_on:
      - db
2. Levantar todos los servicios
docker compose up -d
3. Ver contenedores activos
docker ps
4. Detener todo el stack
docker compose down
🚀 Conclusión
Docker permite controlar contenedores individualmente.
Docker Compose simplifica la gestión de múltiples servicios.
Es la forma recomendada para aplicaciones reales.
💡 Recomendaciones
Usar docker compose en lugar de docker-compose
Evitar --link (deprecated)
Usar depends_on
Construir imágenes antes si no existen

---

Si quieres después te lo dejo todavía más pro con:
- badges tipo GitHub  
- tabla de arquitectura  
- o versión tipo documentación oficial 🔥