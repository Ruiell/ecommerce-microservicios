# ecommerce-microservicios
# E-commerce — Plataforma de Microservicios

Proyecto final DuocUC — Arquitectura de 10 microservicios + Eureka + API Gateway,
desplegable en modo **local (Docker + Laragon)** y **remoto (Render + base de datos en la nube)**.

## Integrantes

| Nombre | Microservicios a cargo |
|---|---|
| Javier Barrios | ms-categorias, ms-inventario, ms-envios, ms-resenas, ms-notificaciones, eureka-server, api-gateway |
| Ignacio Sanchez | ms-carrito, ms-pagos, ms-pedidos, ms-productos, ms-usuarios |

## Repositorio

`https://github.com/TU_USUARIO/TU_REPOSITORIO` — reemplaza este link por el real una vez que subas el proyecto (ver sección GitHub más abajo).

---

## 1. Arquitectura

```
                         [ Postman / Cliente ]
                                  |
                                  v
                         [ api-gateway :8080 ]
                                  |
        -------------------------------------------------------
        |          |            |            |                |
   ms-usuarios ms-productos ms-categorias ms-inventario   ms-carrito   ...(10 en total)
     :8081        :8082        :8083         :8084          :8085
        |___________|_____________|_____________|______________|
                                  |
                        [ eureka-server :8761 ]
                     (registro y descubrimiento)
```

| Microservicio | Puerto | Base de datos (Laragon local) |
|---|---|---|
| eureka-server | 8761 | — |
| api-gateway | 8080 | — |
| ms-usuarios | 8081 | ecommerce_usuarios |
| ms-productos | 8082 | ecommerce_productos |
| ms-categorias | 8083 | ecommerce_categorias |
| ms-inventario | 8084 | ecommerce_inventario |
| ms-carrito | 8085 | ecommerce_carrito |
| ms-pedidos | 8086 | ecommerce_pedidos |
| ms-pagos | 8087 | ecommerce_pagos |
| ms-envios | 8088 | ecommerce_envios |
| ms-resenas | 8089 | ecommerce_resenas |
| ms-notificaciones | 8090 | ecommerce_notificaciones |

## 2. Qué incluye el proyecto (contenido del semestre)

- Arquitectura de microservicios independientes (1 dominio, 1 base de datos por servicio)
- Comunicación entre servicios vía WebClient / Feign
- Registro y descubrimiento de servicios con **Eureka**
- Enrutamiento centralizado con **Spring Cloud Gateway** (rutas dinámicas `lb://`)
- Configuración por **YAML** con perfiles (`dev`, `test`, `render`, `renderdb`, `aiven`)
- **HATEOAS** y **Swagger/OpenAPI** en los **10 microservicios** (rutas `/api/v2/...` con enlaces HAL,
  además de las rutas normales `/api/...`)
- Manejo global de excepciones, DTOs, validaciones (`jakarta.validation`)
- Tests unitarios con Mockito en los 10 microservicios
- **JaCoCo** configurado en los 10 `pom.xml` — reporte de cobertura en `target/site/jacoco/index.html`
  tras correr `mvn test`
- Carga de datos de prueba con **DataFaker** (`DataLoader`, perfil `dev`)
- Contenerización con **Docker** y **Docker Compose**
- Despliegue en **Render** (PaaS) con base de datos en la nube (opcional, ver sección 4)
- Colección de **Postman** lista para importar: `ecommerce-microservicios-local.postman_collection.json`
- Script `run-tests-coverage.ps1` / `.bat` — corre los tests de los 10 servicios y resume la cobertura
  de JaCoCo de cada uno en una sola tabla

---

## 3. Modo LOCAL (Docker + Laragon) — requisito 1

> **Nota sobre los requisitos:** local y remoto son alternativas válidas por separado (no es
> obligatorio hacer ambos), la base de datos puede ser local y/o remota, y lo único no
> negociable es que los microservicios corran en **contenedores Docker**. Este proyecto
> cumple de sobra solo con el modo local + Docker + Laragon.

### Requisitos previos
- Docker Desktop instalado y corriendo
- Laragon corriendo con MySQL activo
- Maven (`mvn -v` debe funcionar)

### Paso 1 — Crear las bases de datos en Laragon
Abre HeidiSQL/consola de Laragon y ejecuta una vez:
```
mysql -u root -p < init-databases.sql
```
Esto crea las 10 bases (`ecommerce_usuarios`, `ecommerce_productos`, etc.).

### Paso 2 — Compilar todos los microservicios
Desde la raíz del proyecto, cada microservicio se compila por separado (no hay pom.xml raíz multi-módulo):
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

### Paso 3 — Levantar todo con Docker Compose
```powershell
docker compose up --build
```
`docker-compose.yml` conecta cada contenedor a tu Laragon usando `host.docker.internal`
(la forma en que Docker Desktop en Windows alcanza servicios corriendo en el propio Windows),
por eso los datos quedan en tus bases de Laragon y no se pierden al reiniciar los contenedores.

### Paso 4 — Verificar
- Eureka: http://localhost:8761 → deben aparecer las 12 aplicaciones registradas.
- Gateway: http://localhost:8080/api/productos, `/api/usuarios`, etc.

### Paso 5 — Probar con Postman (evidencia recomendada)
Importa `ecommerce-microservicios-local.postman_collection.json` (incluida en la raíz del
repo) — trae 69 requests organizados en 10 carpetas, una por microservicio, con las URLs
y puertos ya configurados (`/api/...` normal y `/api/v2/...` HATEOAS). No requiere el Gateway;
apunta directo a cada servicio. Se sugiere guardar capturas o un video corriendo la colección
completa ("Run collection") como respaldo del trabajo.

### Paso 6 — Tests y cobertura (JaCoCo)
Para no entrar carpeta por carpeta, corre el script incluido desde la raíz del proyecto:
```powershell
run-tests-coverage.bat
```
Corre `mvn clean test` en los 10 microservicios y al final imprime una tabla con resultado
de tests y % de cobertura de cada uno (lee `target/site/jacoco/jacoco.csv`). El reporte HTML
detallado de cada servicio queda en `<servicio>/target/site/jacoco/index.html`.

---

## 4. Modo REMOTO (Render + base de datos en la nube) — opcional (requisitos 2, 5 y 6)

> Este modo es **opcional**: el requisito permite entregar solo local, solo remoto, o ambos.
> Si vas a entregar solo local, puedes saltarte esta sección completa.
>
> **Nota:** el perfil `renderdb` (Postgres nativo de Render) ya está listo en 8 de los 10
> servicios. A `ms-pagos` y `ms-pedidos` les falta ese perfil — si decides sí desplegar a
> Render, avísame y lo agrego antes de que hagas el Blueprint.

Usamos la **base de datos PostgreSQL nativa de Render** (gratuita, vive en la misma cuenta,
más liviana que depender de un proveedor externo como Aiven).

### Paso 1 — Subir el proyecto a GitHub (ver sección 5)

### Paso 2 — Crear el Blueprint en Render
1. Entra a render.com → **New > Blueprint**.
2. Conecta tu repositorio de GitHub.
3. Antes de desplegar, reemplaza **todas** las apariciones de `TUPREFIJO` en `render.yaml`
   por un identificador corto tuyo (ej. tus iniciales), para que los nombres de servicio
   sean únicos en Render.
4. Presiona **Deploy Blueprint**.

> **Importante (requisito 5):** si el plan gratuito no te permite tener los 10 microservicios
> arriba al mismo tiempo, deja funcionando como mínimo 3 (recomendado: `ms-productos`,
> `ms-usuarios`, `ms-categorias`) comentando el resto de los bloques `- type: web` en
> `render.yaml`. Los 10 SIEMPRE deben seguir funcionando en modo local — eso no es opcional.

### Paso 3 — Verificar
```
https://TUPREFIJO-ms-productos.onrender.com/actuator/health
https://TUPREFIJO-ms-productos.onrender.com/api/productos
```
(reemplaza por el nombre real de cada servicio desplegado)

### Paso 4 — Probar con Postman
Igual que en local, pero usando las URLs públicas `https://TUPREFIJO-<servicio>.onrender.com`.
El primer request puede tardar 30-60 segundos (el plan free "duerme" servicios inactivos).

### Nota sobre Eureka en Render
Render Free no permite red privada entre Web Services, así que en modo remoto cada
microservicio corre con `EUREKA_ENABLED=false` (no se registra en Eureka) y se accede
directo por su URL pública. Eureka/Gateway dinámico se demuestra en el modo LOCAL.

### Perfiles de base de datos disponibles (por si Render nativo no te resulta)
| Perfil | Base de datos | Uso |
|---|---|---|
| `dev` | MySQL Laragon | Desarrollo local |
| `render` | H2 en memoria | Demo rápida sin BD externa (datos no persisten) |
| `renderdb` | PostgreSQL nativo de Render | **Recomendado para el examen** |
| `aiven` | MySQL Aiven (nube externa) | Alternativa si Render nativo falla |

---

## 5. Subir a GitHub (requisito 7)

```powershell
git init
git add .
git commit -m "Proyecto final - 10 microservicios"
git branch -M main
git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git
git push -u origin main
```
El repositorio debe quedar **público** para que el docente pueda clonarlo. Verifica en
GitHub → Settings → General → "Danger Zone" que dice "Public", no "Private".

---

## 6. Checklist antes de entregar (requisitos 3 y 4)

- [ ] Los 10 microservicios levantan sin error en local (`docker compose up`)
- [ ] Eureka muestra las 12 aplicaciones registradas
- [ ] El Gateway enruta correctamente a cada microservicio
- [ ] (Solo si entregas también remoto) Como mínimo 3 microservicios responden en Render
- [ ] Colección de Postman probada — capturas o video como evidencia (sugerido, no obligatorio)
- [ ] `run-tests-coverage.bat` corrido al menos una vez, sin tests fallidos
- [ ] Repositorio de GitHub público, con este README actualizado
- [ ] Revisar la rúbrica del examen entregada por el docente y marcar cada punto contra
      este proyecto (súbeme la rúbrica si quieres que la revisemos juntos punto por punto)

**Recordatorio de las reglas flexibles confirmadas por el docente:**
- Local y/o remoto: cualquiera de las dos (o ambas) es válida.
- Docker es **obligatorio** siempre, sin excepción.
- La base de datos puede ser solo local, solo remota, o ambas.
- Evidencia en Postman (capturas/video) es sugerida como respaldo, no obligatoria.

---

## 7. Estructura del repositorio

```
proyecto-limpio/
├── docker-compose.yml              # Local: Laragon + Docker (los 10 + eureka + gateway)
├── docker-compose.render-local.yml # Prueba local del modo "sin Eureka" (solo ms-productos)
├── render.yaml                     # Blueprint de Render (los 10 + base de datos nativa)
├── init-databases.sql              # Crea las 10 bases de datos en Laragon
├── ecommerce-microservicios-local.postman_collection.json  # Colección de Postman (69 requests)
├── run-tests-coverage.bat          # Doble clic: corre tests + JaCoCo en los 10 servicios
├── run-tests-coverage.ps1          # Script real detrás del .bat
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
└── render/docker/                  # Dockerfiles multietapa (uno por microservicio) para Render
```
