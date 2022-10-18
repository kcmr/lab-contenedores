# Laboratorio contenedores

## Ejercicio 1

### Creación de la red `lemoncode-challenge`:
```sh
$ docker network create lemoncode-challenge
```

### Preparar la aplicación .NET para "dockerizarla"

Antes de crear el contenedor para la aplicación .NET, necesitamos hacer los siguientes cambios en el código:

- En `Properties/launchSettings.json` cambiar todos los `localhost` por `0.0.0.0`. [Explicación en Stack Overflow](https://stackoverflow.com/questions/59657499/unable-to-bind-to-http-localhost5000-on-the-ipv6-loopback-interface-cannot#comment117481867_59662573)
- En `appsettings.json` cambiar la URL de conexión a mongo de `mongodb://some-mongo:27017` a `mongodb://some-mongo:27017`.

### Crear el Dockerfile para la aplicación .NET

Path `./backend/Dockerfile`

```Dockerfile

FROM mcr.microsoft.com/dotnet/sdk:3.1

WORKDIR /App
    
COPY . .

CMD ["dotnet", "run"]
```

### Crear el Dockerfile para la aplicación Node:

Path `./frontend/Dockerfile`:

```Dockerfile
FROM node:18-alpine

WORKDIR /app

RUN apk update && \
    apk upgrade && \ 
    apk add curl

COPY ["package.json", "package-lock.json", "./"]

RUN npm ci

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
```

Nota: instalamos `curl` para poder crear algunas peticiones POST a la API y añadir más contenido a la BBDD.

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

### Arrancar los contenedores

Contenedor Mongo:
```sh
$ docker run --name some-mongo -p 27017:27017 --network lemoncode-challenge -v mongo_data:/etc/mongo -d mongo
```

Contenedor .NET:
```sh
$ docker run -d --name topics-api --network lemoncode-challenge lemoncode-challenge-backend
```

Contenedor Node:
```sh
$ docker run -d --name frontend -p 8080:3000 --network lemoncode-challenge -e API_URI=http://topics-api:5000/api/topics lemoncode-challenge-frontend
```

### Acceder a la aplicación frontend y añadir más entradas a la BBDD:

Si accedemos con el navegador a http://localhost:8080 veremos la aplicación frontend con el contenido que hayamos añadido inicialmente a la BBDD creada en Mongo.

Añadimos más entradas a la BBDD desde el contenedor de la aplicación Node:

```sh
$ docker exec -it frontend sh
$ curl -X POST -d '{"TopicName": "Docker volumes"}' -H 'Content-Type: application/json' http://topics-api:5000/api/topics
```

Si refrescamos el navegador veremos la nueva entrada.