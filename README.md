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