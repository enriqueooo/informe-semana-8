# Semana 8 - Automatización del Backend con Docker

## 1. Título  
**Automatización del despliegue de una aplicación backend con Docker, PostgreSQL y pgAdmin**

## 2. Tiempo de duración  
**90 minutos**

## 3. Fundamentos  

En esta práctica se aprendió a automatizar el despliegue local de una aplicación backend desarrollada con Spring Boot, acompañada de una base de datos PostgreSQL y su panel de administración pgAdmin, todo ejecutado mediante Docker y Docker Compose.

Se configuraron servicios con persistencia de datos, redes personalizadas para asegurar la comunicación entre los contenedores, y se utilizó la técnica de multi-stage builds para optimizar la imagen Docker del backend.

## 4. Conocimientos previos

- Fundamentos de Docker y Docker Compose.
- Conocimiento básico de Spring Boot.
- Uso básico de bases de datos PostgreSQL.
- Manejo de terminal y variables de entorno.

## 5. Objetivos a alcanzar

- Automatizar la creación y despliegue local de servicios PostgreSQL y pgAdmin con volúmenes y redes configuradas.
- Construir la imagen Docker del backend con un Dockerfile multi-stage para optimizar tamaño y rendimiento.
- Configurar un entorno de variables de entorno en `.env` para gestionar parámetros sensibles.
- Verificar la conexión correcta entre pgAdmin y PostgreSQL, así como la conexión del backend a la base de datos.

## 6. Equipo necesario

- Docker y Docker Compose instalados.
- JDK (Java Development Kit) para construir la aplicación.
- Conexión a internet.
- Editor de texto o IDE.

## 7. Material de apoyo

- [Docker Docs](https://docs.docker.com/)
- [Docker Compose Docs](https://docs.docker.com/compose/)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [pgAdmin Docs](https://www.pgadmin.org/docs/)
- Repositorio base: https://github.com/maguaman2/tendencias-mar22-security.git

---

## 8. Procedimiento

### Paso 1: Crear el archivo `.env`

Definir las variables de entorno necesarias para la base de datos y pgAdmin.

```env
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin123
POSTGRES_DB=tendenciasdb
PGADMIN_DEFAULT_EMAIL=admin@admin.com
PGADMIN_DEFAULT_PASSWORD=admin123
```
# Configuración de `docker-compose.yml`

En este paso definiremos los servicios para PostgreSQL, pgAdmin y el backend, utilizando volúmenes y redes personalizadas para asegurar la persistencia de datos y la comunicación entre contenedores.

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - backend-network

  pgadmin:
    image: dpage/pgadmin4
    environment:
      PGADMIN_DEFAULT_EMAIL: ${PGADMIN_DEFAULT_EMAIL}
      PGADMIN_DEFAULT_PASSWORD: ${PGADMIN_DEFAULT_PASSWORD}
    ports:
      - "8080:80"
    depends_on:
      - postgres
    networks:
      - backend-network

  backend:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      SPRING_DATASOURCE_USERNAME: ${POSTGRES_USER}
      SPRING_DATASOURCE_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - "8081:8080"
    depends_on:
      - postgres
    networks:
      - backend-network

volumes:
  pgdata:

networks:
  backend-network:
    driver: bridge
```
