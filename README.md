# Nest JS Interview Questions & Answers

[![Status](https://img.shields.io/badge/status-active-success.svg)]()

> Click :star:if you like the project. Pull Request are highly appreciated.

[@sudheerj](https://github.com/sudheerj) - Inspiration & Idea

[@manish8561](https://github.com/manish8561) - Initial work

---

### Table of Contents

<details open>
<summary>
Hide/Show table of contents
</summary>

| No  | Questions                                                                                                                                                                         |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | [What is Nest JS?](#what-is-nest-js)                                                                                                                                              |
| 2   | [What are interceptors? Write a code for create a interceptor to transform the response.](#what-are-interceptors-write-a-code-for-create-a-interceptor-to-transform-the-response) |
| 3   | [Write and explain a code snippet to achieve CRUD in nest js.](#write-and-explain-a-code-snippet-to-achieve-crud-in-nest-js) |
| 4   | [What are the relationships available in nest js?](#what-are-the-relationships-available-in-nest-js) |
| 5   | [What is mongodb? What is mongoose? How you connect nest js with mongo db? Write a sample code for this?](#what-is-mongodb-what-is-mongoose-how-you-connect-nest-js-with-mongo-db-write-a-sample-code-for-this) |
| 6   | [What is swagger? What is REST? What is rest api? How you create a rest api with the help of swagger? Write code snippet.](#what-is-swagger-what-is-rest-what-is-rest-api-how-you-create-a-rest-api-with-the-help-of-swagger-write-code-snippet) |
| 7   | [Explain how you use a rest api created using swagger in nest js to other software technology?](#explain-how-you-use-a-rest-api-created-using-swagger-in-nest-js-to-other-software-technology) |
| 8   | [What is nest factory?](#what-is-nest-factory) |
| 9   | [How to crud operations in nest js with GraphQL? Write a code snippet.](#how-to-crud-operations-in-nest-js-with-graphql-write-a-code-snippet) |
</details>

1. ### What is Nest JS?
    Nest (NestJS) is a framework for building efficient, scalable Node.js server-side applications. It uses progressive JavaScript, is built with and fully supports TypeScript (yet still enables developers to code in pure JavaScript) and combines elements of OOP (Object Oriented Programming), FP (Functional Programming), and FRP (Functional Reactive Programming).

    Under the hood, Nest makes use of robust HTTP Server frameworks like Express (the default) and optionally can be configured to use Fastify as well!

**[⬆ Back to Top](#table-of-contents)**

2. ### What are interceptors? Write a code for create a interceptor to transform the response.

    In NestJS, interceptors are middleware-like components used to intercept incoming requests or outgoing responses, allowing you to perform common tasks such as logging, modifying data, or transforming responses.

    Below is an example of how you can create an interceptor to transform the response in NestJS:

    First, let's create a new interceptor:

    ```ts
    // response-transform.interceptor.ts
    @Injectable()
    export class ResponseTransformInterceptor<T>
      implements NestInterceptor<T, any>
    {
      intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
        return next.handle().pipe(
          map((data) => {
            // Transform the response
            return {
              data: data,
              message: "Response transformed successfully",
            };
          })
        );
      }
    }
    ```

    In this interceptor:

    We implement the NestInterceptor interface provided by NestJS.
    We override the intercept method, which receives the ExecutionContext and a CallHandler.
    Inside the intercept method, we use RxJS's map operator to transform the response data.

    Now, let's apply this interceptor to a controller method:

    ```ts
    // app.controller.ts

    import { Controller, Get, UseInterceptors } from "@nestjs/common";
    import { ResponseTransformInterceptor } from "./response-transform.interceptor";

    @Controller()
    export class AppController {
      @Get("data")
      @UseInterceptors(ResponseTransformInterceptor)
      getData(): { name: string; age: number } {
        return { name: "John Doe", age: 30 };
      }
    }
    ```

    In this controller:

    We use the @UseInterceptors decorator to apply our ResponseTransformInterceptor to the getData method.
    When the getData method is called, the response will be intercepted by ResponseTransformInterceptor, and the response data will be transformed.

    Finally, make sure to include the interceptor and controller in your NestJS module:

    ```ts
    // app.module.ts

    import { Module } from "@nestjs/common";
    import { AppController } from "./app.controller";
    import { ResponseTransformInterceptor } from "./response-transform.interceptor";

    @Module({
      controllers: [AppController],
      providers: [ResponseTransformInterceptor],
    })
    export class AppModule {}
    ```

    With this setup, when you access the /data endpoint, the response will be transformed according to the logic defined in the ResponseTransformInterceptor.

**[⬆ Back to Top](#table-of-contents)**

3. ### Write and explain a code snippet to achieve CRUD in nest js.

    Let's create a "todo" module to manage tasks. We'll have a Todo entity with CRUD operations exposed through a controller.

    First, let's define the Todo entity:

    ```ts
    // todo.entity.ts

    export class Todo {
      id: number;
      title: string;
      description: string;
      completed: boolean;
    }
    ```

    Now, let's create a service to handle CRUD operations for the Todo entity:

    ```ts
    // todo.service.ts

    import { Injectable } from "@nestjs/common";
    import { Todo } from "./todo.entity";

    @Injectable()
    export class TodoService {
      private todos: Todo[] = [];

      getAll(): Todo[] {
        return this.todos;
      }

      getById(id: number): Todo {
        return this.todos.find((todo) => todo.id === id);
      }

      create(todo: Todo): Todo {
        this.todos.push(todo);
        return todo;
      }

      update(id: number, todo: Todo): Todo {
        const index = this.todos.findIndex((t) => t.id === id);
        if (index !== -1) {
          this.todos[index] = todo;
          return todo;
        }
        return null;
      }

      delete(id: number): Todo {
        const index = this.todos.findIndex((todo) => todo.id === id);
        if (index !== -1) {
          const deletedTodo = this.todos[index];
          this.todos.splice(index, 1);
          return deletedTodo;
        }
        return null;
      }
    }
    ```

    Next, let's create a controller to handle HTTP requests and interact with the TodoService:

    ```ts
    // todo.controller.ts

    import {
      Controller,
      Get,
      Post,
      Put,
      Delete,
      Param,
      Body,
    } from "@nestjs/common";
    import { TodoService } from "./todo.service";
    import { Todo } from "./todo.entity";

    @Controller("todos")
    export class TodoController {
      constructor(private readonly todoService: TodoService) {}

      @Get()
      getAll(): Todo[] {
        return this.todoService.getAll();
      }

      @Get(":id")
      getById(@Param("id") id: number): Todo {
        return this.todoService.getById(+id);
      }

      @Post()
      create(@Body() todo: Todo): Todo {
        return this.todoService.create(todo);
      }

      @Put(":id")
      update(@Param("id") id: number, @Body() todo: Todo): Todo {
        return this.todoService.update(+id, todo);
      }

      @Delete(":id")
      delete(@Param("id") id: number): Todo {
        return this.todoService.delete(+id);
      }
    }
    ```

    Finally, make sure to register the controller and service in the appropriate module:

    ```ts
    // todo.module.ts

    import { Module } from "@nestjs/common";
    import { TodoController } from "./todo.controller";
    import { TodoService } from "./todo.service";

    @Module({
      controllers: [TodoController],
      providers: [TodoService],
    })
    export class TodoModule {}
    ```

    With this setup, you have a basic CRUD implementation for managing Todo resources in NestJS. You can make HTTP requests to endpoints like /todos (GET for getting all todos, POST for creating a new todo), /todos/:id (GET for getting a specific todo, PUT for updating a todo, DELETE for deleting a todo).

**[⬆ Back to Top](#table-of-contents)**

4. ### What are the relationships available in nest js?
   In NestJS, you can manage relationships between entities using various techniques depending on the data persistence layer you're using (e.g., TypeORM, Sequelize, Mongoose). Here are some common types of relationships you can handle in NestJS:

- **One-to-One**: Each record in one entity is associated with exactly one record in another entity, and vice versa.

- **One-to-Many (or Many-to-One)**: Each record in one entity can be associated with multiple records in another entity, but each record in the other entity is associated with only one record in the first entity.

- **Many-to-Many**: Each record in one entity can be associated with multiple records in another entity, and vice versa.

  These relationships are typically defined using decorators provided by the ORM libraries used in NestJS. Here's a brief overview of how these relationships are defined using TypeORM as an example:

  Below is an example of how you can create an interceptor to transform the response in NestJS:

  ```ts
  // User entity
  import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
  import { Todo } from "./todo.entity";

  @Entity()
  export class User {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany((type) => Todo, (todo) => todo.user)
    todos: Todo[];
  }

  // Todo entity
  import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
  import { User } from "./user.entity";

  @Entity()
  export class Todo {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @ManyToOne((type) => User, (user) => user.todos)
    user: User;
  }
  ```

  In this example:

  The User entity has a one-to-many relationship with the Todo entity, defined by the `@OneToMany` decorator.

  The Todo entity has a many-to-one relationship with the User entity, defined by the `@ManyToOne` decorator.

  These relationships are bi-directional, meaning that changes made to one side of the relationship are automatically reflected on the other side.

  Remember that the specific syntax and decorators used for defining relationships may vary depending on the ORM library you're using (e.g., TypeORM, Sequelize, Mongoose).

**[⬆ Back to Top](#table-of-contents)**

5. ### What is mongodb? What is mongoose? How you connect nest js with mongo db? Write a sample code for this?
   MongoDB is a popular NoSQL database that stores data in a flexible, JSON-like format called BSON (Binary JSON). It's known for its scalability, flexibility, and performance, making it a popular choice for many types of applications, especially those with large amounts of unstructured or semi-structured data.

   Mongoose, on the other hand, is an Object Data Modeling (ODM) library for MongoDB and Node.js. It provides a straightforward schema-based solution to model your application data. With Mongoose, you can define schemas, create models, and perform CRUD operations with ease, all while leveraging the power and flexibility of MongoDB.

   To connect NestJS with MongoDB using Mongoose, you first need to install the required dependencies:

   ```bash
   npm install @nestjs/mongoose mongoose
   ```
   Then, you need to set up the connection in your NestJS application. Here's a sample code demonstrating how to do this:
   ```ts
    // app.module.ts

    import { Module } from '@nestjs/common';
    import { MongooseModule } from '@nestjs/mongoose';
    import { CatsModule } from './cats/cats.module';

    @Module({
      imports: [
        MongooseModule.forRoot('mongodb://localhost/nestjs_mongodb_example', {
          useNewUrlParser: true,
          useUnifiedTopology: true,
        }),
        CatsModule,
      ],
    })
    export class AppModule {}
   ```
   
    In this code:

  - We import `MongooseModule` from `@nestjs/mongoose` to integrate Mongoose with NestJS.
  - We use the `forRoot()` method of `MongooseModule` to establish a connection to the MongoDB database. You need to provide the connection URI as the first argument. In this example, we're connecting to a local MongoDB instance named `nestjs_mongodb_example`. The second argument is an options object where we specify options like `useNewUrlParser` and `useUnifiedTopology`.
  - `CatsModule` is just a placeholder for your application modules. Replace it with your actual modules.

    Now, let's create a sample module and model using Mongoose:

    ```ts
    // cats.module.ts

    import { Module } from '@nestjs/common';
    import { MongooseModule } from '@nestjs/mongoose';
    import { CatsController } from './cats.controller';
    import { CatsService } from './cats.service';
    import { CatSchema } from './schemas/cat.schema';

    @Module({
      imports: [
        MongooseModule.forFeature([{ name: 'Cat', schema: CatSchema }]),
      ],
      controllers: [CatsController],
      providers: [CatsService],
    })
    export class CatsModule {}

    ```
    ```ts
    // cat.schema.ts

    import { Schema } from 'mongoose';

    export const CatSchema = new Schema({
      name: String,
      age: Number,
      breed: String,
    });

    ```
    ```ts
    // cats.controller.ts

    import { Controller, Get } from '@nestjs/common';
    import { CatsService } from './cats.service';
    import { Cat } from './interfaces/cat.interface';

    @Controller('cats')
    export class CatsController {
      constructor(private readonly catsService: CatsService) {}

      @Get()
      async findAll(): Promise<Cat[]> {
        return this.catsService.findAll();
      }
    }
    ```
    ```ts
    // cats.service.ts

    import { Injectable } from '@nestjs/common';
    import { InjectModel } from '@nestjs/mongoose';
    import { Model } from 'mongoose';
    import { Cat } from './interfaces/cat.interface';

    @Injectable()
    export class CatsService {
      constructor(@InjectModel('Cat') private readonly catModel: Model<Cat>) {}

      async findAll(): Promise<Cat[]> {
        return this.catModel.find().exec();
      }
    }

    ```
    ```ts
    // cat.interface.ts

    export interface Cat {
      name: string;
      age: number;
      breed: string;
    }
    ```
    In this code:

  - We define a `CatSchema` using Mongoose's schema definition.
  - We create a `CatsModule` where we import `MongooseModule.forFeature()` to specify that we'll be using the Cat model within this module. We provide an array with an object that contains the name of the model `(Cat)` and the schema `(CatSchema)`.
  - We define a `CatsController` to handle HTTP requests related to cats.
  - We create a `CatsService` to encapsulate the business logic for managing cats. This service injects the `Cat` model using `@InjectModel('Cat')`.
  - We define a `Cat` interface to enforce a specific structure for cat objects.
  
    This example demonstrates a basic setup of NestJS with MongoDB using Mongoose for Object Data Modeling. You can expand upon this foundation to build more complex applications.
  
**[⬆ Back to Top](#table-of-contents)**

6. ### What is swagger? What is REST? What is rest api? How you create a rest api with the help of swagger? Write code snippet.
    `Swagger` is a set of open-source tools built around the OpenAPI Specification that helps developers design, build, document, and consume RESTful APIs. It provides a user-friendly interface for developers to visualize and interact with APIs, as well as generate interactive API documentation automatically.

    `REST (Representational State Transfer)`is an architectural style for designing networked applications. It relies on a stateless, client-server communication protocol, typically `HTTP`, and emphasizes principles such as resource identification through URIs, uniform interface, stateless communication, and caching.

    A `REST API (Representational State Transfer Application Programming Interface)` is an API that adheres to the principles of `REST`. It allows clients to interact with server resources by sending `HTTP` requests `(GET, POST, PUT, DELETE)` to perform various actions like retrieving, creating, updating, or deleting resources.

    To integrate Swagger with a NestJS application, you can use the `@nestjs/swagger` package. Here's how you can create a simple REST API with Swagger documentation using NestJS:

    First, install the necessary dependencies:
    ```bash
    npm install @nestjs/swagger swagger-ui-express
    ```
    Then, set up Swagger in your NestJS application:
    ```ts
    // main.ts

    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule);

      // Swagger setup
      const config = new DocumentBuilder()
        .setTitle('Sample API')
        .setDescription('Sample API with Swagger')
        .setVersion('1.0')
        .addTag('cats')
        .build();
      const document = SwaggerModule.createDocument(app, config);
      SwaggerModule.setup('api', app, document);

      await app.listen(3000);
    }
    bootstrap();
    ```
    Next, let's create a basic Cats module with a controller:
    ```ts
    // cats.module.ts

    import { Module } from '@nestjs/common';
    import { CatsController } from './cats.controller';

    @Module({
      controllers: [CatsController],
    })
    export class CatsModule {}
    ```
    ```ts
    // cats.controller.ts

    import { Controller, Get } from '@nestjs/common';
    import { ApiTags } from '@nestjs/swagger';

    @ApiTags('cats')
    @Controller('cats')
    export class CatsController {
      @Get()
      findAll(): string {
        return 'This action returns all cats';
      }
    }
    ```
    In the above code:
  - We import `ApiTags` decorator from `@nestjs/swagger` and apply it to the controller to add tags for Swagger documentation.
  - We define a `GET` endpoint for `/cats` that returns a simple string message.
  
    Now, when you run your NestJS application and navigate to **http://localhost:3000/api**, you should see the Swagger UI interface displaying documentation for your API, including the `/cats` endpoint. Users can interact with this interface to make requests to your API and view responses.

    This is just a basic example. You can further expand your API by adding more controllers, services, and models, and Swagger will automatically generate documentation for them.


**[⬆ Back to Top](#table-of-contents)**

7. ### Explain how you use a rest api created using swagger in nest js to other software technology?
    Once you've created a REST API with Swagger in NestJS, you can easily consume it from other software technologies, regardless of the programming language or framework being used. Here's how you can use the Swagger documentation to interact with the REST API from another technology:

    **API Documentation:** The Swagger UI provides a user-friendly interface for developers to explore the API endpoints, request/response payloads, and any required parameters or headers. Users can interact with the Swagger UI to understand how to make requests to the API and interpret the responses.

    **API Client Generation:** Swagger provides tools to generate API clients in various programming languages automatically. These clients abstract away the low-level HTTP communication details and provide a convenient interface for making requests to the API. Developers can generate client code for their preferred programming language using the Swagger documentation.

    **Manual Integration:** Alternatively, developers can manually integrate with the REST API using HTTP requests. They can use libraries or frameworks available in their technology stack to send HTTP requests (GET, POST, PUT, DELETE) to the API endpoints defined in the Swagger documentation. The documentation specifies the endpoint URLs, request payloads (if any), and expected response formats, which developers can use to construct their requests and handle the responses accordingly.

    Regardless of the approach chosen, the Swagger documentation serves as a comprehensive reference for developers to understand and interact with the REST API created using NestJS. It provides the necessary information for consuming the API from other software technologies, facilitating interoperability and integration across diverse systems.

**[⬆ Back to Top](#table-of-contents)**

8. ### What is nest factory?
    In NestJS, the NestFactory is a class provided by the NestJS framework that is responsible for creating instances of the Nest application. It is used to bootstrap your Nest application, allowing you to create an application instance and configure it before listening for incoming requests.

    Here are some key points about NestFactory:

    **Application Bootstrap:** `NestFactory` is typically used in the `main.ts` file of your NestJS application to bootstrap your application. You use it to create an instance of your application module (`AppModule`) and configure it with various options before starting the server.

    **Async Initialization:** The `create` method of `NestFactory` returns a Promise that resolves to the application instance. This allows you to perform asynchronous initialization tasks, such as setting up database connections or loading configuration settings, before your application starts listening for requests.

    **Configuration Options:** `NestFactory` provides methods to configure various aspects of your application, such as setting up middleware, enabling CORS (Cross-Origin Resource Sharing), specifying the HTTP server framework (e.g., Express, Fastify), and more. These methods allow you to customize your application's behavior according to your requirements.

    **Dependency Injection Container:** `NestFactory` utilizes the dependency injection container provided by NestJS to manage the application's dependencies. It automatically resolves and injects dependencies into your controllers, services, and other components, making it easy to build modular and testable applications.

    Here's a basic example of using NestFactory to bootstrap a NestJS application:

    ```ts
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      await app.listen(3000);
    }
    bootstrap();
    ```
    In this example:
  - We import the `NestFactory` class from `@nestjs/core`.
  - We import the root module of our Nest application, typically named `AppModule`.
  - We define an asynchronous function `bootstrap()` that uses `NestFactory.create()` to create an instance of our application module.
  - We call `app.listen()` to start the server and make it listen for incoming HTTP requests on port 3000.
    
    Overall, NestFactory simplifies the process of creating and configuring NestJS applications, allowing developers to focus on building robust and scalable backend solutions.

**[⬆ Back to Top](#table-of-contents)**

9. ### How to crud operations in nest js with GraphQL? Write a code snippet.
    To perform CRUD operations in NestJS with GraphQL, you typically use the `@nestjs/graphql` module along with other GraphQL-related libraries. Here's a basic example of how you can set up a GraphQL API in NestJS and implement CRUD operations:

    First, install the required dependencies:
    ```bash
    npm install @nestjs/graphql graphql type-graphql class-validator class-transformer
    ```
    Now, let's create a simple GraphQL schema and implement CRUD operations for a hypothetical Post entity:
    ```ts
    // post.entity.ts

    import { ObjectType, Field, ID } from '@nestjs/graphql';

    @ObjectType()
    export class Post {
      @Field(() => ID)
      id: string;

      @Field()
      title: string;

      @Field()
      content: string;
    }
    ```
    ```ts
    // post.service.ts

    import { Injectable } from '@nestjs/common';
    import { Post } from './post.entity';
    import { v4 as uuidv4 } from 'uuid';

    @Injectable()
    export class PostService {
      private posts: Post[] = [];

      findAll(): Post[] {
        return this.posts;
      }

      findById(id: string): Post {
        return this.posts.find(post => post.id === id);
      }

      create(title: string, content: string): Post {
        const post: Post = { id: uuidv4(), title, content };
        this.posts.push(post);
        return post;
      }

      update(id: string, title: string, content: string): Post {
        const index = this.posts.findIndex(post => post.id === id);
        if (index !== -1) {
          this.posts[index].title = title;
          this.posts[index].content = content;
          return this.posts[index];
        }
        return null;
      }

      delete(id: string): boolean {
        const index = this.posts.findIndex(post => post.id === id);
        if (index !== -1) {
          this.posts.splice(index, 1);
          return true;
        }
        return false;
      }
    }
    ```
    ```ts
    // post.resolver.ts

    import { Resolver, Query, Mutation, Args, ID } from '@nestjs/graphql';
    import { Post } from './post.entity';
    import { PostService } from './post.service';

    @Resolver(() => Post)
    export class PostResolver {
      constructor(private readonly postService: PostService) {}

      @Query(() => [Post])
      async posts(): Promise<Post[]> {
        return this.postService.findAll();
      }

      @Query(() => Post)
      async post(@Args('id', { type: () => ID }) id: string): Promise<Post> {
        return this.postService.findById(id);
      }

      @Mutation(() => Post)
      async createPost(
        @Args('title') title: string,
        @Args('content') content: string,
      ): Promise<Post> {
        return this.postService.create(title, content);
      }

      @Mutation(() => Post)
      async updatePost(
        @Args('id', { type: () => ID }) id: string,
        @Args('title') title: string,
        @Args('content') content: string,
      ): Promise<Post> {
        return this.postService.update(id, title, content);
      }

      @Mutation(() => Boolean)
      async deletePost(@Args('id', { type: () => ID }) id: string): Promise<boolean> {
        return this.postService.delete(id);
      }
    }
    ```
    ```ts
    // post.module.ts

    import { Module } from '@nestjs/common';
    import { PostService } from './post.service';
    import { PostResolver } from './post.resolver';

    @Module({
      providers: [PostService, PostResolver],
    })
    export class PostModule {}
    ```
    ```ts
    // app.module.ts

    import { Module } from '@nestjs/common';
    import { GraphQLModule } from '@nestjs/graphql';
    import { PostModule } from './post/post.module';

    @Module({
      imports: [
        GraphQLModule.forRoot({
          autoSchemaFile: true,
        }),
        PostModule,
      ],
    })
    export class AppModule {}
    ```
    In this code:

  - We define a **Post** entity with **id**, **title**, and **content** fields.
  - We create a **PostService** to handle CRUD operations for **Post** entities.
  - We implement resolver methods in **PostResolver** to expose GraphQL queries and mutations for CRUD operations.
  - We create a GraphQL module using `GraphQLModule.forRoot()` in `AppModule` to configure the GraphQL server and specify the schema file location (`autoSchemaFile`).
  - We import and include the **PostModule** in **AppModule** to make the **PostResolver** and **PostService** available in the application.
    
    With this setup, you can now run your NestJS application and use GraphQL to perform CRUD operations on Post entities.

**[⬆ Back to Top](#table-of-contents)**
