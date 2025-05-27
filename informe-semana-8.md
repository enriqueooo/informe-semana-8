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
# Paso 3: Crear el Dockerfile multi-stage para el backend

Este Dockerfile utiliza una construcción multi-stage para optimizar la imagen final del backend. Primero compila el proyecto con Maven y luego crea una imagen ligera solo con el archivo `.jar` resultante.

```dockerfile
# Stage 1: Build
FROM maven:3.8.6-openjdk-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# Stage 2: Run
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

```
# Paso 4: Construir y levantar los servicios

Ejecuta el siguiente comando para construir las imágenes (incluyendo la del backend) y levantar todos los servicios definidos en `docker-compose.yml`:

```bash
docker-compose up --build
```
# Paso 5: Verificación

1. **Acceder a pgAdmin**  
   Abre tu navegador y visita:  
   [http://localhost:8080](http://localhost:8080)  
   Inicia sesión con las credenciales definidas en las variables de entorno.

2. **Configurar conexión en pgAdmin**  
   - Añade un nuevo servidor o conexión.  
   - Configura los datos de conexión a PostgreSQL:  
     - Host: `postgres`  
     - Puerto: `5432`  
     - Usuario: `${enrique}`  
     - Contraseña: `${enrique123}`  
     - Base de datos: `${POSTGRES_DB}`

3. **Verificar backend**  
   - Accede a la aplicación backend en:  
     [http://localhost:8081](http://localhost:8081)  
   - Confirma que la aplicación está corriendo y se conecta correctamente a la base de datos PostgreSQL.

---
# 9. Resultados esperados

- Los contenedores de PostgreSQL, pgAdmin y backend se levantan correctamente sin errores.
- Los datos de PostgreSQL persisten entre reinicios gracias al volumen configurado.
- pgAdmin puede conectarse y administrar la base de datos PostgreSQL.
- El backend accede a la base de datos y responde a peticiones en el puerto configurado.

---

# 10. Bibliografía

- [Docker Docs](https://docs.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [PostgreSQL Official Documentation](https://www.postgresql.org/docs/)
- [pgAdmin Documentation](https://www.pgadmin.org/docs/)
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)

---

# 11. Enlace del audio

Audio de la práctica  
*(Aquí puedes agregar el enlace o indicar dónde encontrar el audio)*  


