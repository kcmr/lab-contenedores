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