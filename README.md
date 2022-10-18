# Laboratorio contenedores

## Ejercicio 1

### Creación de la red `lemoncode-challenge`
```sh
$ docker network create lemoncode-challenge
```

### Preparar la aplicación .NET para "dockerizarla"

Antes de crear el contenedor para la aplicación .NET, necesitamos hacer los siguientes cambios en el código:

- En `Properties/launchSettings.json` cambiar todos los `localhost` por `0.0.0.0`. [Explicación en Stack Overflow](https://stackoverflow.com/questions/59657499/unable-to-bind-to-http-localhost5000-on-the-ipv6-loopback-interface-cannot#comment117481867_59662573)
- En `appsettings.json` cambiar la URL de conexión a mongo de `mongodb://localhost:27017` a `mongodb://some-mongo:27017`.

### Crear el Dockerfile para la aplicación .NET

Para la aplicación .NET no exponemos puertos puesto que no accederemos a ella directamente desde el host sino desde el contenedor de la aplicación frontend, que tendrá acceso a ella mediante la red compartida `lemoncode-challenge`.

Path `./backend/Dockerfile`

```Dockerfile

FROM mcr.microsoft.com/dotnet/sdk:3.1

WORKDIR /App
    
COPY . .

CMD ["dotnet", "run"]
```

### Crear el Dockerfile para la aplicación Node:

Para la aplicación Frontend exponemos el puerto 3000 en el que corre la aplicación Express para poder mapearlo al puerto 8080 que usaremos para acceder a ella en el navegador desde el host.

Path `./frontend/Dockerfile`:

```Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY ["package.json", "package-lock.json", "./"]

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

### Construir las imágenes a partir de los Dockerfiles

Construir la imagen para la app .NET:
```sh
$ cd backend
$ docker build -t lemoncode-challenge-backend .
```

Construir la imagen para la app Node:
```sh
$ cd frontend
$ docker build -t lemoncode-challenge-frontend .
```

### Arrancar el contenedor Mongo con un volumen mapeado

Para el contenedor de Mongo exponemos el puerto 27017 ya que nos conectaremos a él desde el host mediante el cliente gráfico MongoDB Compass.

```sh
$ docker run --name some-mongo \
  -p 27017:27017 \ 
  --network lemoncode-challenge \
  -v mongo_data:/data/db -d mongo
```

Una vez conectados a Mongo, creada la BBDD `TopicstoreDb` y la colección `Topics`, añadimos algunas entradas importando el archivo `Topics.json` situado en la raíz del proyecto.


### Arrancar los contenedores de las aplicaciones

Una vez que tenemos el contenedor Mongo corriendo, arrancamos los contenedores de las aplicaciones backend y frontend en este orden:

**Contenedor .NET**:
```sh
$ docker run -d --name topics-api \
  --network lemoncode-challenge \
  lemoncode-challenge-backend
```

**Contenedor Node**:
```sh
$ docker run -d --name frontend \
  -p 8080:3000 \
  --network lemoncode-challenge \
  -e API_URI=http://topics-api:5000/api/topics \
  lemoncode-challenge-frontend
```

La aplicación frontend ya es accesible desde http://localhost:8080

Notas: tanto para el contenedor de Mongo, como para los de las aplicaciones usamos la red `lemoncode-challenge`. En el caso de la aplicación Node pasamos también la variable de entorno `API_URI` con la URL de la aplicación .NET contenerizada.

## Ejercicio 2

Archivo [`docker-compose.yml`](./docker-compose.yml)

En este caso hemos usado un bind mount para el volumen usado en el contenedor Mongo. Al igual que en el caso del ejercicio 1, necesitaremos importar entradas manualmente a Mongo en la primera ejecución a partir del archivo `Topics.json`. En siguientes ejecuciones se usarán los datos que se hayan ido guardando en el directorio `mongo-data` mapeado a `/data/db` del contenedor Mongo.

Arrancar los contenedores:

```sh
$ docker compose up
```

Detener y eliminar los contenedores:

```sh
$ docker compose down
```

Detener y eliminar los contenedores y los volúmenes asociados a ellos:

```sh
$ docker compose down -v
```

