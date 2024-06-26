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
This command also creates the `.env` file in our project. This file is never uploaded to online repositories, thus, we will create a `.env.example` file with dummy values to emulate a real `.env`.

### 4.2.1. The Prisma Syntax:
- Data source: Specifies the database connection. The default configuration to this project describes that the database provider is PostgreSQL and the db connection string is found under the `DATABASE_URL` environment variable.
- Generator: Sets the assets to be created in the moment of running the `prisma generate` command
- Data Model: Defines your db's models (or entities). Each model will have it's own table on the database. In the current state, our schema does not have any Data Model.

## 4.3. Modeling the Data:
For this Medium simplified clone, you will only need the `Article` model to represent each article on the blog.
### 4.3.1. Creating the Article on schema:
In the `prisma/schema.prisma` file add a new model to the schema, named `Article`.
```prisma
model Article {
  id          Int      @id @default(autoincrement()) // @id indicates it's the PK
  title       String   @unique
  description String? // The `?` indicates it is an optional argument
  body        String
  published   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt // @updatedAt updates the field with the current timestamp when the Article is modified
}
```
## 4.4. Migrating the Database:
With the schema created, we need to migrate it, to actually create all the tables in the database. To do so, run the following command:
```console
npx prisma migrate dev --name "init"
```
### 4.4.1. Explaining the migration command:
1. Save the migration: Generates the SQL commands equivalent to the `schema.prisma` file;
2. Execute the migration: Execute the SQL commands in the migration file to create the tables in the database;
3. Generate the Prisma Client: (Requires another dependency: `@prisma/client`) Prisma installs automatically the `@prisma/client` dependency if lacking and generates the Prisma Client based on the current Migration. The Prisma Client is a query builder auto-generated from the Prisma schema. Can be used to send queries to the database.

If successful, inside the `prisma` directory, we should have a new folder: `migrations`.

## 4.5. Seeding the Database:
Currently, the database is empty. Thus, we will create a seed script to populate the database with some dummy data.
### 4.5.1 Creating the Seeding file:
Inside `prisma/` create the `seed.ts` file:
```console
touch prisma/seed.ts
```
Then, add the following code inside it:
```typescript
import { PrismaClient } from '@prisma/client';


// initialize Prisma Client
const prisma = new PrismaClient();

async function main() {

  // create two dummy articles
  const post1 = await prisma.article.upsert({
    where: { title: 'Prisma Adds Support for MongoDB' },
    update: {},
    create: {
      title: 'Prisma Adds Support for MongoDB',
      body: 'Support for MongoDB has been one of the most requested features since the initial release of...',
      description:
        "We are excited to share that today's Prisma ORM release adds stable support for MongoDB!",
      published: false,
    },
  });

  const post2 = await prisma.article.upsert({
    where: { title: "What's new in Prisma? (Q1/22)" },
    update: {},
    create: {
      title: "What's new in Prisma? (Q1/22)",
      body: 'Our engineers have been working hard, issuing new releases with many improvements...',
      description:
        'Learn about everything in the Prisma ecosystem and community from January to March 2022.',
      published: true,
    },
  });

  console.log({ post1, post2 });
}

// execute the main function
main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    // close Prisma Client at the end
    await prisma.$disconnect();
  });
```
Here, we use the `upsert` command, which is similar to `create`. There main difference is that `upsert` first checks the database for an instance with the same identifier as the one we are currently inserting and blocks if it would cause a dupe. On the other hand, `create` only inserts the new instance in the database, being a little more dangerous.

### 4.5.2. Adding the seed command to our package.json:
In order to be able to run the seeding script easier, we can insert it in the package.json file, to use it with the prisma cli:
```json
// package.json

// ...
  "scripts": {
    // ...
  },
  (...)
  "jest" : {
    // ...
  },
  "prisma": {

    "seed": "ts-node prisma/seed.ts"

  }
```
By adding the `prisma` field, we should be able to run the `seed.ts` script using the following command on the terminal:
```console
npx prisma db seed
```
## 4.6. Creating a Prisma Service:
This is done mearly for best practices. We create a service called `PrismaService` responsible to instantiate a `PrismaClient` in order to abstract the Prisma Client API from our Application.
To instantiate this Prisma Service, we will use the [Nest's CLI](https://docs.nestjs.com/cli/usages):
```command
npx nest generate module prisma
npx nest generate service prisma
```
### 4.6.1. The prisma.service.ts file:
Inside the `prisma/prisma.service.ts` file, we should have the following code:
```typescript
import { INestApplication, Injectable } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient {}
```
### 4.6.2. The prisma.module.ts file:
In terms of [Architecture](https://refactoring.guru/design-patterns), the `prisma.module.ts` file is responsible to create a [singleton](https://refactoring.guru/pt-br/design-patterns/singleton) instance of the `PrismaService` to be shared accross the application. For doing so, we add the `PrismaService` to the `exports` array in the `prisma.module.ts` file.
```typescript
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';

@Module({
  providers: [PrismaService],
  exports: [PrismaService],
})
export class PrismaModule {}
```
Essentially, all the files that import the `PrismaModule` now have access to the `PrismaService` too.
This is a common pattern and a good coding and architecture practice in Nest projects.
# 5. Setting up Swagger:
[Swagger](https://swagger.io/docs/) is a documentation tool to document APIs following the OpenAI's specifications.
## 5.1. Installing Swagger:
To install Swagger, use the following command:
```console
npm install --save @nestjs/swagger swagger-ui-express
```
## 5.2. Initializing Swagger on the Application:
To use Swagger, we need to initialize it in our application.
To do so, we need to open the `main.ts` file and inistialize Swagger using the `SwaggerModule` class:
```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  const config = new DocumentBuilder()
    .setTitle('Median')
    .setDescription('Median API Description')
    .setVersion('0.1')
    .build()

  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(3000);
}
bootstrap();
```
Now, we added the Swagger's setup to our application and the [`http://localhost:3000/api`](http://localhost:3000/api) endpoint is available to access our API's documentation. There, we can see the Swagger's default documentation page.
# 6. Implementing CRUD Operations for Article:
Now, we are going to implement our first [CRUD](https://www.codecademy.com/article/what-is-crud) operations (Create, Read, Update, Delete) for the `Article` model we created earlier. Together, we should implement any [business logic](https://www.techtarget.com/whatis/definition/business-logic) for our application.
## 6.1. Generating the Nest Resources:
Before creating any of the CRUD operations, we need to create the REST resources using the Nest CLI for Article, since we just created the Article model in the Database but didn't instantiate it inside Nest quite yet.
```console
npx nest generate resource
```
Here, you will be prompted some questions:
1. `What name would you like to use for this resource (plural, e.g., "users")?` *articles*
2. `What transport layer do you use?` *REST API*
3. `Would you like to generate CRUD entry points?` *Yes*

Now, we just created the Boilerplate for our Article Model inside Nestjs!
We should have a new directory inside our `src/` folder: `src/articles/` and we should have the new articles' endpoints appearing in our Swagger's documentation page under [`http://localhost:3000/api`](http://localhost:3000/api) url.

## 6.2. Adding PrismaClient to the Articles module:
As antecipated earlier, we need to add the Prisma Client Singleton to the Articles module for it to be able to use it.
To do so, we simply need to add a new import to our Article module file under `src/articles/articles.module.ts`:
```typescript
import { Module } from '@nestjs/common';
import { ArticlesService } from './articles.service';
import { ArticlesController } from './articles.controller';
import { PrismaModule } from 'src/prisma/prisma.module';


@Module({
  controllers: [ArticlesController],
  providers: [ArticlesService],
  imports: [PrismaModule],

})
export class ArticlesModule {}
```

Now, we can inject the `PrismaService` inside the `ArticlesService` and use it to access the database. To do this, add a constructor to `articles.service.ts`:
```typescript
import { Injectable } from '@nestjs/common';
import { CreateArticleDto } from './dto/create-article.dto';
import { UpdateArticleDto } from './dto/update-article.dto';
import { PrismaService } from 'src/prisma/prisma.service';

@Injectable()
export class ArticlesService {
  // Injecting the PrismaClient to access the database
  constructor(private prisma: PrismaService) {}   

  // CRUD Operations
}
```
## 6.3. Creating the Endpoints:
### 6.3.1. GET /articles:
The controller for this endpoint is `findAll`. This endpoint is supposed to return all published articles in the database.
This controller is implemented like so in the `articles.service.ts` file:
```typescript
@Get()
findAll() {
  return this.prisma.article.findMany({ where: { published: true } });
}
```
This query will resolve for all the article's instances with the `published` atribute set to `true`.
### 6.3.2. GET /articles/drafts:
This endpoint will return all the article's instances with the `published` attribute set to `false`. 
For doing so, we first declare the Endpoint in the `articles.controller.ts` file, then we create the method in `articles.service.ts`.
1. Articles' controller file:
```typescript
@Get('drafts')
findDrafts() {
  return this.articlesService.findDrafts();
}
```
2. Articles' service file:
```typescript
findDrafts() {
  return this.prisma.article.findMany({ where: { published: false } });
}
```
### 6.3.3. GET /articles/:id:
This endpoint will return an article given it's `id`. Since this is a default endpoint, we won't need to set anything on the controller side. But the implementation on `service` will be done using the `findUnique` method from `prisma` as follows:
```typescript
findOne(id: number) {
  return this.prisma.article.findUnique({ where: { id } });
}
```
### 6.3.4. POST /articles:
This endpoint is used for adding a new article instance to the database:
For doing so, we will need to create a `CreateArticleDto` type. A DTO is a Data Transfer Object, which is, basically an object that defines how the data will be sent over to the network.
To declare the CreateArticleDto class, we do it in the `src/articles/dto/create-article-dto.ts`:
```typescript
import { ApiProperty } from '@nestjs/swagger';

export class CreateArticleDto {

  @ApiProperty()
  title: string;

  @ApiProperty({ required: false })
  description?: string;

  @ApiProperty()
  body: string;

  @ApiProperty({ required: false, default: false })
  published?: boolean = false;
}
```
We use the `@ApiProperty` decorators to make the class properties visible to the `SwaggerModule` as described in [NestJS Docs](https://docs.nestjs.com/openapi/types-and-parameters).
Now, the CreateArticleDto should be visible under the Schemas on the Swagger API page.


# Quick Tips:
## Errors when running the Dockerfile:
### Error Fetching server API version:
If you find an error when trying to run the `docker-compose up` command, check if Docker Service is running:
```console
systemctl status docker
```
If the Service is not running, then start it:
```console
systemctl start docker
```
And then try to run again. It should fix the errors.
```console
docker-compose up
```
### Authentication Error:
If the case is an Authentication Error, execute the command as administrator/sudo:
```console
sudo docker-compose up
```
If the error is just Authentication, using this command should work.

## Errors Generating Modules and Services while running the Nest Server:
If you find any errors running the Nest's CLI while Nest is already running, such as: `Cannot find module './app.controller'`, run the following command from the terminal:
```console
rm -rf dist
```
And then restart the server.

# References:
- [Prisma Blog](https://www.prisma.io/blog/nestjs-prisma-rest-api-7D056s1BmOL0)
- [Prisma Docs](https://www.prisma.io/docs) 
- [Nestjs Docs](https://docs.nestjs.com/) 
