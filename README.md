# Docker & Docker Compose

How to use docker and docker-compose to run the application.

## Install Docker

- [Windows](https://docs.docker.com/desktop/install/windows-install/)
- [MacOS](https://docs.docker.com/desktop/install/mac-install/)
- [Linux](https://docs.docker.com/desktop/install/linux-install/)

## Create Server Image

- Create `Dockerfile` in the root of the server project
- Create `.dockerignore` in the root of the server project

### Server Dockerfile

```dockerfile
FROM node:16

WORKDIR /app

COPY . .

RUN npm install

RUN npm run build:docker

EXPOSE 8080

CMD [ "npm", "run", "start:docker" ]
```

### Server .dockerignore

Add files that should not be copied to the image.

```git
# Bundles
dist
node_modules

# Secrets
.vscode
.env
```

### Build Server Image

```bash
docker build -t [image name] [path to Dockerfile]
```

```bash
docker build -t server .
```

### Run Server Image

```bash
docker run --name [container name] -p [host port]:[container port] [image name]
```

```bash
docker run --name server -p 3000:8080 server
```

## TODO: Create Website Image

## Docker Compose

- Create `docker-compose.yml` in the root of the project
- Create `.server.env` in the root of the project (for the server service environment variables)
- Create `.env` in the root of the project (for the postgres password)

### docker-compose.yml

```yaml
version: "3.8"

# The services of your app
services:

  # Database service: postgres:14
  database:
    # PostgreSQL image
    image: postgres:14
    # Restart the container if it crashes
    restart: always
    # Environment variables
    environment:
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD}"
    # Mount the database-data volume
    volumes:
      - database-data:/var/lib/postgresql/data

  # Server service: ./server
  server:
    # Build the image from the Dockerfile
    build: ./server
    # Restart the container if it crashes
    restart: on-failure
    # Environment variables
    env_file:
      - .server.env
    environment:
      DB_PASS: "${POSTGRES_PASSWORD}"
    # Mount the server-data volume
    volumes:
      - server-data:/app/storage
    # Expose port 3000 to the host
    ports:
      - 3000:8080
    # Link the database service
    depends_on:
      - database

  # Website service: ./website
  website:
    # Build the image from the Dockerfile
    build: ./website
    # Restart the container if it crashes
    restart: on-failure
    # Expose port 80 to the host
    ports:
      - 80:8080
    # Link the server service
    depends_on:
      - server

# Volumes store your app's data
volumes:
  database-data:
  server-data:
```

### .server.env

```env
# App [ENV: dev,test,prod]
ENV=dev

# Database
DB_USER=postgres
DB_HOST=database
DB_NAME=postgres
DB_TEST_NAME=school_test
DB_PORT=5432

# Bcrypt
BCRYPT_SECRET=
BCRYPT_SALT=10

# JWT
JWT_SECRET=

# Mail
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=587
MAIL_USER=
MAIL_PASS=
```

### .env

```env
POSTGRES_PASSWORD=
```
