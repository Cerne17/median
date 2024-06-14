# Table of Content

TODO: add the table of contents

# 1. Introduction:
We are going to create a Medium Clone - Median.
## 1.1. Technologies:
- Nestjs: Backend Framework
- Prisma: Object-Relational Model (ORM)
- PostgreSQL: Database
- Swagger: API Documentation
- Typescript: Programming Language

## 1.2. Dependencies:
- [Nodejs](https://developer.fedoraproject.org/tech/languages/nodejs/nodejs.html)
- [Docker](https://docs.docker.com/engine/install/fedora/) or [PostgreSQL](https://docs.fedoraproject.org/en-US/quick-docs/postgresql/)
- Access to a Unix shell

# 2. Generating the Nest Project
## 2.1. Installing Dependencies:
Firstly, you will need to install the Nestjs' CLI
```console
npm i -g @nestjs/cli
```

## 2.2. Initializing the Project:
After the CLI's installation, create the Project's boilerplate scripts:
```console
nest new median
```
or even:
```console
npx @nestjs/cli new median
```
During the initilization you will be prompted to choose a package manager (npm, yarn and pnpm). For safety, use npm.
Finally, we need to enter the project's directory:
```console
cd median
```
## 2.3. Understanding the Project's Structure:
All Nestjs projects feature some default files, the main ones are:
- `src/app.module.ts`: This is the root module of the application
- `src/app.controler.ts`: This creates the main route of the application `/`. This returns - as default - a `'Hello World!'` page.
- `src/main.ts`: The entry point of the application, is the first file to be ran. It bootstraps the application.

## 2.4. Running the project:
To run the project and make it autoupdate with code changes, we can run the following command:
```console
npm run start:dev
```
This command watches the files for change, recompiles them on-demand and run the server on the `http://localhost:3000/` URL.
This should display the `'Hello World!'` page.

# 3. Creating the PostgreSQL Instance
To create a PostgreSQL Instance (our database), we're going to use Docker Containers.
## 3.1. Configuring the Docker Files:
### 3.1.1. Create the Docker Compose file:
Create this file on the main directory of your project:
```console
touch docker-compose.yml
```
### 3.2. Write the configuration file:
Inside the docker.compose file, insert the configuration as follows:
```dockerfile
version: '3.8'
services:

    postgres:
        image: postgres:13.5
        restart: always
        environment:
            - POSTGRES_USER=myuser
            - POSTGRES_PASSWORD=mypassword
        volumes:
            - postgres:/var/lib/postgresql/data
        ports:
            - '5432:5432'
volumes:
    postgres:
```
- image: tells Docker which image to look for (postgres - version 13.5)
- environment: Sets the environment variables.
- volumes: creates the data persistence in the Docker Container
- ports: maps ports from the host machine to the container (`'host-port:container-port'`)
- The postgreSQL database will run on the 5432 port, this port should not be used by any other process.
### Running Docker:
On a new Terminal, in the project's main directory run the following command to run the docker-compose file.
```console
docker-compose up
```
This will run the postgres database while running.
If you encounter a Permission error, try the following command:
```console
sudo docker-compose up
```
# 4. Setting up Prisma:
## 4.1. Installing Prisma:
Install Prisma CLI as a development dependency:
```console
npm install -D prisma
```
## 4.2. Initializing a Prisma Project:
```console
npx prisma init
```
This command creates the `schema.prisma` file. This is the main configuration file that will contain our database schema. 
This command also creates the `.env` file in our project. This file is never uploaded to online repositories, thus, we will create a `.env.example` file with dummie values to emulate a real `.env`.

## References:
- [Prisma Blog](https://www.prisma.io/blog/nestjs-prisma-rest-api-7D056s1BmOL0)
- [Prisma Docs](https://www.prisma.io/docs) 
- [Nestjs Docs](https://docs.nestjs.com/) 
