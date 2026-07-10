# Proyecto Final - E-commerce con Microservicios

Este proyecto corresponde al trabajo final de la asignatura. El objetivo fue desarrollar una plataforma de e-commerce utilizando una arquitectura basada en microservicios con Spring Boot. Cada microservicio se encarga de una funcionalidad específica del sistema y posee su propia base de datos para mantener la independencia entre servicios.

## Integrantes

| Nombre          | Microservicios desarrollados                                                                       |
| --------------- | -------------------------------------------------------------------------------------------------- |
| Javier Barrios  | ms-categorias, ms-inventario, ms-envios, ms-resenas, ms-notificaciones, eureka-server, api-gateway |
| Ignacio Sanchez | ms-carrito, ms-pagos, ms-pedidos, ms-productos, ms-usuarios                                        |

## Repositorio

`https://github.com/Ruiell/ecommerce-microservicios`

---

# Arquitectura del proyecto

```
                         Cliente / Postman
                                |
                                v
                         API Gateway :8080
                                |
        -------------------------------------------------------
        |         |          |          |          |
  Usuarios   Productos  Categorías Inventario  Carrito ...
        |____________________________________________|
                        Eureka Server :8761
```

## Microservicios

| Microservicio     | Puerto | Base de datos            |
| ----------------- | -----: | ------------------------ |
| eureka-server     |   8761 | —                        |
| api-gateway       |   8080 | —                        |
| ms-usuarios       |   8081 | ecommerce_usuarios       |
| ms-productos      |   8082 | ecommerce_productos      |
| ms-categorias     |   8083 | ecommerce_categorias     |
| ms-inventario     |   8084 | ecommerce_inventario     |
| ms-carrito        |   8085 | ecommerce_carrito        |
| ms-pedidos        |   8086 | ecommerce_pedidos        |
| ms-pagos          |   8087 | ecommerce_pagos          |
| ms-envios         |   8088 | ecommerce_envios         |
| ms-resenas        |   8089 | ecommerce_resenas        |
| ms-notificaciones |   8090 | ecommerce_notificaciones |

---

# Tecnologías utilizadas

Durante el desarrollo del proyecto se trabajó con las siguientes tecnologías:

* Java 17
* Spring Boot
* Spring Cloud
* Eureka Server
* Spring Cloud Gateway
* MySQL
* Docker y Docker Compose
* Swagger / OpenAPI
* HATEOAS
* Maven
* JUnit 5
* Mockito
* JaCoCo
* DataFaker
* Postman
* Aiven

---

# Ejecución del proyecto en local

## Requisitos

Antes de ejecutar el proyecto es necesario tener instalado:

* Docker Desktop
* Laragon con MySQL activo
* Maven

## 1. Crear las bases de datos

Ejecutar el siguiente script desde Laragon:

```bash
mysql -u root -p < init-databases.sql
```

Este script crea las bases de datos necesarias para todos los microservicios.

## 2. Compilar los proyectos

Cada microservicio se compila por separado.

```powershell
mvn -f eureka-server/pom.xml clean package -DskipTests
mvn -f api-gateway/pom.xml clean package -DskipTests
mvn -f ms-usuarios/pom.xml clean package -DskipTests
mvn -f ms-productos/pom.xml clean package -DskipTests
mvn -f ms-categorias/pom.xml clean package -DskipTests
mvn -f ms-inventario/pom.xml clean package -DskipTests
mvn -f ms-carrito/pom.xml clean package -DskipTests
mvn -f ms-pedidos/pom.xml clean package -DskipTests
mvn -f ms-pagos/pom.xml clean package -DskipTests
mvn -f ms-envios/pom.xml clean package -DskipTests
mvn -f ms-resenas/pom.xml clean package -DskipTests
mvn -f ms-notificaciones/pom.xml clean package -DskipTests
```

## 3. Levantar el proyecto

```bash
docker compose up --build
```

Una vez iniciado Docker Compose, todos los servicios deberían registrarse automáticamente en Eureka.

## 4. Verificación

* Eureka: http://localhost:8761
* API Gateway: http://localhost:8080

Los distintos endpoints pueden probarse utilizando la colección de Postman incluida en el proyecto.

---

# Pruebas

El proyecto incluye una colección de Postman para probar los distintos microservicios.

También incorpora pruebas unitarias con JUnit y Mockito.

Para ejecutar los tests se puede utilizar el siguiente script:

```powershell
run-tests-coverage.bat
```

Al finalizar se genera el reporte de cobertura utilizando JaCoCo.

---

# Despliegue

Además del funcionamiento en ambiente local, el proyecto puede desplegarse en Render utilizando el archivo `render.yaml`.

La configuración también permite trabajar con bases de datos remotas utilizando los perfiles definidos en cada microservicio.

---

# Subir el proyecto a GitHub

```powershell
git init
git add .
git commit -m "Proyecto Final"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git
git push -u origin main
```

---

# Estructura del proyecto

```
proyecto-limpio/
├── docker-compose.yml
├── render.yaml
├── init-databases.sql
├── ecommerce-microservicios-local.postman_collection.json
├── run-tests-coverage.bat
├── run-tests-coverage.ps1
├── eureka-server/
├── api-gateway/
├── ms-usuarios/
├── ms-productos/
├── ms-categorias/
├── ms-inventario/
├── ms-carrito/
├── ms-pedidos/
├── ms-pagos/
├── ms-envios/
├── ms-resenas/
├── ms-notificaciones/
└── render/docker/
```

---

# Observaciones

El proyecto fue desarrollado siguiendo una arquitectura de microservicios, donde cada servicio funciona de manera independiente y administra su propia base de datos. La comunicación entre los servicios se realiza mediante Spring Cloud, utilizando Eureka para el registro de servicios y API Gateway para centralizar el acceso a los endpoints.

Para facilitar las pruebas se incluyó una colección de Postman y pruebas unitarias con reporte de cobertura mediante JaCoCo.

