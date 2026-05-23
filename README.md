# innovatech-backend

Backend y base de datos del sistema Innovatech, implementados con Node.js, Express, MySQL, Docker, Amazon EC2, Amazon ECR y GitHub Actions.

Este repositorio contiene la API backend del sistema, la configuración de base de datos MySQL y los archivos necesarios para su contenedorización, despliegue en AWS y automatización mediante CI/CD.

## Descripción general

El proyecto forma parte de una arquitectura de tres capas:

```txt
Internet → Frontend → Backend → Base de datos
````

En esta arquitectura, el backend se encarga de recibir las solicitudes provenientes del frontend, procesarlas y comunicarse con la base de datos MySQL. La base de datos almacena la información de productos utilizada por la aplicación.

El backend fue desplegado en una instancia EC2 privada, mientras que la base de datos fue desplegada en una instancia EC2 privada independiente. La comunicación entre ambos componentes se controla mediante Security Groups.

## Estructura del repositorio

```txt
innovatech-backend/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   └── server.js
│
├── db/
│   ├── Dockerfile
│   └── init.sql
│
├── .github/
│   └── workflows/
│       ├── deploy-backend.yml
│       └── deploy-db.yml
│
├── docker-compose.yml
├── .gitignore
└── README.md
```

## Tecnologías utilizadas

* Node.js
* Express
* MySQL
* Docker
* Docker Compose
* Amazon EC2
* Amazon ECR
* AWS Systems Manager
* GitHub Actions

## Backend

El backend corresponde a una API REST desarrollada con Node.js y Express. Se ejecuta en el puerto `3001` y permite gestionar productos mediante operaciones CRUD.

Endpoints principales:

```txt
GET    /api/productos
GET    /api/productos/:id
POST   /api/productos
PUT    /api/productos/:id
DELETE /api/productos/:id
GET    /api/health
```

## Variables de entorno del backend

El backend utiliza variables de entorno para conectarse a la base de datos:

```txt
PORT=3001
DB_HOST=<IP_PRIVADA_EC2_DB>
DB_USER=alumno
DB_PASSWORD=alumno123
DB_NAME=tienda_perritos
DB_PORT=3306
```

Durante el despliegue en AWS, `DB_HOST` corresponde a la IP privada de la instancia EC2 donde se ejecuta la base de datos.

## Base de datos

La base de datos utiliza MySQL y se inicializa mediante el archivo:

```txt
db/init.sql
```

Este archivo permite crear la estructura inicial de la base de datos y cargar datos de prueba para validar el funcionamiento de la aplicación.

Credenciales utilizadas en el entorno de despliegue:

```txt
MYSQL_ROOT_PASSWORD=admin123
MYSQL_DATABASE=tienda_perritos
MYSQL_USER=alumno
MYSQL_PASSWORD=alumno123
```

## Dockerfile del backend

El Dockerfile del backend utiliza una imagen base de Node.js:

```txt
node:18-alpine
```

Su función principal es:

1. Crear el directorio de trabajo `/app`.
2. Copiar los archivos de dependencias.
3. Instalar dependencias de producción.
4. Copiar el código fuente.
5. Exponer el puerto `3001`.
6. Ejecutar la aplicación mediante `npm start`.

## Dockerfile de base de datos

El Dockerfile de la base de datos utiliza la imagen oficial de MySQL:

```txt
mysql:8
```

Además, copia el archivo `init.sql` en:

```txt
/docker-entrypoint-initdb.d/
```

Esto permite que el script SQL se ejecute automáticamente al iniciar por primera vez el contenedor de MySQL.

## Docker Compose

El archivo `docker-compose.yml` permite levantar backend y base de datos en un entorno contenerizado.

Servicios definidos:

```txt
db       → Base de datos MySQL
backend  → API Node.js / Express
```

El archivo configura:

* Servicios Docker.
* Variables de entorno.
* Puertos.
* Red interna.
* Volumen persistente.
* Dependencia entre backend y base de datos.

## Persistencia de datos

La persistencia de MySQL se implementa mediante el volumen Docker:

```txt
mysql_data
```

Este volumen se monta en:

```txt
/var/lib/mysql
```

De esta forma, los datos se conservan aunque el contenedor sea detenido o reiniciado.

## Ejecución con Docker Compose

Desde la raíz del repositorio:

```bash
docker compose up --build -d
```

Ver contenedores activos:

```bash
docker ps
```

Detener contenedores:

```bash
docker compose down
```

> No usar `docker compose down -v` si se desea conservar la información de la base de datos, ya que esta opción elimina los volúmenes.

## Despliegue manual en AWS

El despliegue manual fue realizado utilizando instancias EC2 separadas para backend y base de datos.

### EC2 DB

En la instancia `ec2-db` se construyó y ejecutó el contenedor MySQL:

```bash
docker build -t innovatech-db .
docker volume create mysql_data
docker run -d --name innovatech-db -p 3306:3306 -v mysql_data:/var/lib/mysql innovatech-db
```

Se verificó la correcta inicialización de la base de datos ingresando al contenedor y ejecutando:

```sql
SHOW TABLES;
```

Con ello se confirmó la existencia de la tabla `productos`.

### EC2 Backend

En la instancia `ec2-backend` se construyó y ejecutó el contenedor backend:

```bash
docker build -t innovatech-backend .
docker run -d \
  --name innovatech-backend \
  -p 3001:3001 \
  -e PORT=3001 \
  -e DB_HOST=<IP_PRIVADA_EC2_DB> \
  -e DB_USER=alumno \
  -e DB_PASSWORD=alumno123 \
  -e DB_NAME=tienda_perritos \
  -e DB_PORT=3306 \
  innovatech-backend
```

Se validó el funcionamiento mediante:

```bash
curl http://localhost:3001/api/health
curl http://localhost:3001/api/productos
```

## Amazon ECR

Se crearon repositorios privados en Amazon ECR para almacenar las imágenes Docker:

```txt
innovatech-backend
innovatech-db
```

Las imágenes fueron etiquetadas y publicadas en ECR con la etiqueta:

```txt
latest
```

Esto permite que las instancias EC2 puedan descargar las imágenes desde el registro privado de AWS durante los procesos de despliegue.

## CI/CD con GitHub Actions

Este repositorio cuenta con workflows de GitHub Actions ubicados en:

```txt
.github/workflows/
```

Workflows configurados:

```txt
deploy-backend.yml
deploy-db.yml
```

Estos workflows se ejecutan al realizar cambios sobre la rama:

```txt
deploy
```

El flujo automatizado realiza las siguientes acciones:

1. Obtiene el código del repositorio.
2. Configura credenciales temporales de AWS.
3. Inicia sesión en Amazon ECR.
4. Construye la imagen Docker correspondiente.
5. Etiqueta la imagen con la URI de ECR.
6. Publica la imagen en Amazon ECR.
7. Ejecuta comandos en EC2 mediante AWS Systems Manager.
8. Detiene el contenedor anterior.
9. Descarga la nueva imagen.
10. Levanta el contenedor actualizado.

## Secrets utilizados en GitHub Actions

Para el funcionamiento de los workflows se configuraron los siguientes secrets en GitHub:

```txt
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
AWS_REGION
AWS_ACCOUNT_ID
ECR_BACKEND_REPOSITORY
ECR_DB_REPOSITORY
EC2_BACKEND_INSTANCE_ID
EC2_DB_INSTANCE_ID
DB_HOST
```

Estos valores permiten que GitHub Actions pueda autenticarse en AWS, publicar imágenes en ECR y desplegar los contenedores en EC2 mediante SSM.

## Seguridad

La arquitectura implementada utiliza Security Groups para controlar la comunicación entre capas:

```txt
Frontend → Backend → Base de datos
```

Reglas principales:

* El backend acepta tráfico en el puerto `3001` únicamente desde el frontend.
* La base de datos acepta tráfico MySQL en el puerto `3306` únicamente desde el backend.
* La base de datos no se expone directamente a Internet.
* El acceso administrativo se realiza mediante AWS Systems Manager Session Manager.

## Estado final del proyecto

El backend y la base de datos fueron desplegados correctamente en AWS. La API quedó operativa en la instancia EC2 backend y conectada a la base de datos MySQL ubicada en una instancia EC2 privada.

El sistema fue validado mediante pruebas de endpoints, consulta de productos y funcionamiento completo del CRUD desde el frontend.

Además, las imágenes Docker fueron publicadas en Amazon ECR y los workflows de GitHub Actions quedaron configurados para automatizar el proceso de construcción, publicación y despliegue.

## Rama de despliegue

La rama utilizada para despliegue es:

```txt
deploy
```

Los cambios realizados sobre esta rama activan los workflows de CI/CD del proyecto.