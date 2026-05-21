# innovatech-backend

Backend del sistema Innovatech basado en Node.js, Express y MySQL, preparado para contenedorización con Docker, persistencia de datos, despliegue en AWS EC2 y automatización CI/CD con GitHub Actions.

## Descripción general

Este repositorio contiene los componentes backend del proyecto Innovatech:

- `backend/`: API REST desarrollada con Node.js y Express.
- `db/`: Base de datos MySQL contenerizada.
- `docker-compose.yml`: Archivo para levantar el backend y la base de datos como servicios Docker.

La API permite gestionar productos mediante operaciones CRUD y se conecta a una base de datos MySQL usando variables de entorno.

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
├── docker-compose.yml
├── .gitignore
└── README.md
````

## Tecnologías utilizadas

* Node.js
* Express
* MySQL
* Docker
* Docker Compose
* AWS EC2
* Amazon ECR
* GitHub Actions

## Backend

El backend corresponde a una API REST desarrollada con Node.js y Express. Se ejecuta en el puerto `3001`.

Endpoints principales:

```txt
GET    /api/productos
GET    /api/productos/:id
POST   /api/productos
PUT    /api/productos/:id
DELETE /api/productos/:id
GET    /api/health
```

## Variables de entorno

El backend utiliza las siguientes variables de entorno para conectarse a la base de datos:

```txt
PORT=3001
DB_HOST=db
DB_USER=alumno
DB_PASSWORD=alumno123
DB_NAME=tienda_perritos
DB_PORT=3306
```

Dentro de Docker Compose, el backend se conecta a la base de datos usando el nombre del servicio:

```txt
DB_HOST=db
```

Esto permite que ambos contenedores se comuniquen dentro de la red interna de Docker.

## Base de datos

La base de datos utiliza MySQL 8 y se inicializa mediante el archivo:

```txt
db/init.sql
```

El contenedor de base de datos define las siguientes credenciales:

```txt
MYSQL_ROOT_PASSWORD=admin123
MYSQL_DATABASE=tienda_perritos
MYSQL_USER=alumno
MYSQL_PASSWORD=alumno123
```

## Dockerfile del backend

El Dockerfile del backend utiliza una imagen liviana de Node.js:

```txt
node:18-alpine
```

Su función es:

1. Definir el directorio de trabajo `/app`.
2. Copiar los archivos `package.json` y `package-lock.json` si existe.
3. Instalar dependencias de producción.
4. Copiar el código fuente.
5. Exponer el puerto `3001`.
6. Ejecutar la aplicación con `npm start`.

## Dockerfile de base de datos

El Dockerfile de la base de datos utiliza la imagen oficial:

```txt
mysql:8
```

Además, copia el archivo `init.sql` en:

```txt
/docker-entrypoint-initdb.d/
```

Esto permite que el script SQL se ejecute automáticamente la primera vez que se crea el contenedor de MySQL.

## Docker Compose

El archivo `docker-compose.yml` permite levantar el backend y la base de datos de forma conjunta.

Servicios definidos:

```txt
db       → Base de datos MySQL
backend  → API Node.js / Express
```

El archivo configura:

* Variables de entorno.
* Puertos expuestos.
* Red interna Docker.
* Volumen persistente para MySQL.
* Dependencia entre backend y base de datos.

## Ejecución con Docker Compose

Desde la raíz del repositorio:

```bash
docker compose up --build -d
```

Ver contenedores activos:

```bash
docker ps
```

Ver logs del backend:

```bash
docker logs innovatech-backend
```

Ver logs de la base de datos:

```bash
docker logs innovatech-db
```

Detener los contenedores:

```bash
docker compose down
```

No se debe usar `docker compose down -v` si se desea conservar la información de la base de datos, ya que esa opción elimina los volúmenes.

## Puertos utilizados

```txt
Backend: 3001
MySQL:   3306
```

## Persistencia de datos

La persistencia se implementa mediante un volumen nombrado:

```txt
mysql_data
```

Este volumen se monta en:

```txt
/var/lib/mysql
```

Esto permite que los datos de MySQL se mantengan aunque los contenedores se detengan o se reinicien.

## Despliegue en AWS

El despliegue proyectado considera:

* Una instancia EC2 para ejecutar backend y base de datos.
* Docker instalado en la instancia EC2.
* Imágenes Docker publicadas en Amazon ECR.
* Despliegue automatizado mediante GitHub Actions.
* Activación del pipeline al realizar push sobre la rama `deploy`.

## Registro de imágenes

Se utilizará Amazon ECR como registro de imágenes Docker, ya que el despliegue se realizará dentro del ecosistema AWS.

Esta decisión permite mantener las imágenes dentro de la misma infraestructura cloud utilizada por el proyecto, facilitando la integración entre GitHub Actions, ECR y EC2.

## Rama de despliegue

La rama utilizada para el despliegue será:

```txt
deploy
```

Los workflows de GitHub Actions serán configurados para ejecutarse al hacer push sobre esta rama.

## Seguridad y red

La base de datos no debe exponerse públicamente a Internet. La comunicación entre backend y base de datos ocurre mediante la red interna de Docker.

En AWS, se recomienda que:

* El puerto `3001` del backend solo sea accesible desde la instancia frontend o su Security Group.
* El puerto `3306` de MySQL no sea abierto públicamente.
* El acceso administrativo a EC2 se realice mediante SSH restringido o AWS Systems Manager Session Manager.

## Estado actual del proyecto

Actualmente el repositorio contiene:

* Código base del backend.
* Configuración de base de datos.
* Dockerfile para backend.
* Dockerfile para base de datos.
* Docker Compose para levantar backend y MySQL.
* Documentación técnica inicial.