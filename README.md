# Entorno local con Docker Compose

Este proyecto levanta el entorno de prueba de OAuth y sus servicios relacionados:

- `postgres`: base de datos PostgreSQL 18.
- `auth-service`: servidor de autenticacion y emision de tokens.
- `msc-service`: servicio MSC protegido por OAuth.
- `msa-service`: servicio MSA protegido por OAuth, integrado con MSC.
- `msc-web`: aplicacion web y proxy de los servicios anteriores.

Los servicios se conectan a traves de la red Docker `cloud-rtravez`. Compose espera a
que las dependencias pasen sus healthchecks antes de iniciar cada servicio dependiente.

## Requisitos

- Docker Engine con Docker Compose v2 (`docker compose`).
- Los repositorios hermanos deben existir en estas rutas relativas al directorio de
	este archivo:

	```text
	../auth-server/
	../msc-root/msc-service/
	../msa-root/msa-service/
	../msc-web/
	```

Cada proyecto debe incluir el Dockerfile indicado en `docker-compose.yml`.

## Inicio

Desde este directorio, construir las imagenes e iniciar el entorno:

```bash
docker compose up --build -d
```

Comprobar el estado de los contenedores:

```bash
docker compose ps
```

Ver los logs de todos los servicios o de un servicio especifico:

```bash
docker compose logs -f
docker compose logs -f auth-service
```

## Puertos y endpoints

| Servicio | Puerto | Endpoint base |
| --- | ---: | --- |
| PostgreSQL | `5432` | `localhost:5432` |
| Auth | `8080` | `http://localhost:8080/authServices` |
| MSC | `8081` | `http://localhost:8081/mscServices` |
| MSA | `8082` | `http://localhost:8082/msaServices` |
| Web | `4200` | `http://localhost:4200` |

Los endpoints de healthcheck de Spring Actuator son:

```text
http://localhost:8080/authServices/actuator/health
http://localhost:8081/mscServices/actuator/health
http://localhost:8082/msaServices/actuator/health
```

## Detener y limpiar

Detener y eliminar los contenedores, conservando los datos de los volúmenes:

```bash
docker compose down
```

Eliminar tambien los volumenes `pgdata` y `springdata` (esto borra la base de datos
local):

```bash
docker compose down -v
```

## Configuracion de prueba

La configuracion actual usa los siguientes valores de prueba:

```text
Base de datos: db_test
Usuario:       postgres
Contrasena:    admin
Client secret: 12345
```

Estas credenciales estan definidas directamente en `docker-compose.yml` y no deben
usarse en entornos productivos. Para un despliegue real, reemplazarlas por secretos
gestionados externamente.

## Publicar imagenes

Despues de construir las imagenes, se pueden etiquetar y publicar en Docker Hub. Se
requiere haber iniciado sesion con `docker login` y tener permisos sobre el repositorio
`rtravez/oauth`:

```bash
docker tag postgres:18-alpine rtravez/oauth:postgres-service
docker tag auth-service:latest rtravez/oauth:auth-service
docker tag msc-service:latest rtravez/oauth:msc-service
docker tag msa-service:latest rtravez/oauth:msa-service
docker tag msc-web:latest rtravez/oauth:msc-web

docker push rtravez/oauth:postgres-service
docker push rtravez/oauth:auth-service
docker push rtravez/oauth:msc-service
docker push rtravez/oauth:msa-service
docker push rtravez/oauth:msc-web
```
