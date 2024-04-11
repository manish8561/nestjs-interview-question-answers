<h1>Nest JS Interview Questions & Answers</h1>

[![Status](https://img.shields.io/badge/status-active-success.svg)]()

> Click :star:if you like the project. Pull Request are highly appreciated. 

[@manish8561](https://github.com/manish8561) - Idea & Initial work

---

## 📝 Table of Contents

<details open>
<summary>
Hide/Show table of contents
</summary>

| No | Questions |
| --- | --- |
| 1 | [What is Nest JS?](#what-is-nest-js) |
| 2 | [What are interceptors? Write a code for create a interceptor to transform the response.](#what-are-interceptors)|
| 3 | [Write and explain a code snippet to achieve CRUD in nest js.](#crud-in-nestjs)|
| 4 | [What are the relationships available in nest js?](#what-are-relationships-nestjs)|
| 5 | [What are interceptors? Write a code for create a interceptor to transform the response.](#getting_started)|


3. 
4. What is mongodb? What is mongoose? How you connect nest js with mongo db? Write a sample code for this?
5. What is swagger? What is REST? What is rest api? How you create a rest api with the help of swagger? Write code snippet.
6. Explain how you use a rest appi created using swagger in nest js to other software technology?
7. What is nest factory?
8. How to crud operations in nest js with GraphQL? Write a code snippet.
9. What is serialisation? How you transform and sanitize the data?
10. Explain sign up setup in nest js.
11. What are exception filters?
12. How can you call mongodb as NoSQL db?
13. How NoSQL is better than RDMS?
14. How to create a collection? How to create database?
15. Explain CRUD sql statements? Write it.
16. What are the data types available in mongodb?
17. What is aggregation?

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
export class ResponseTransformInterceptor<T> implements NestInterceptor<T, any> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      map(data => {
        // Transform the response
        return {
          data: data,
          message: 'Response transformed successfully'
        };
      }),
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

import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { ResponseTransformInterceptor } from './response-transform.interceptor';

@Controller()
export class AppController {
  @Get('data')
  @UseInterceptors(ResponseTransformInterceptor)
  getData(): { name: string; age: number } {
    return { name: 'John Doe', age: 30 };
  }
}
```
In this controller:

We use the @UseInterceptors decorator to apply our ResponseTransformInterceptor to the getData method.
When the getData method is called, the response will be intercepted by ResponseTransformInterceptor, and the response data will be transformed.

Finally, make sure to include the interceptor and controller in your NestJS module:

```ts
// app.module.ts

import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { ResponseTransformInterceptor } from './response-transform.interceptor';

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

import { Injectable } from '@nestjs/common';
import { Todo } from './todo.entity';

@Injectable()
export class TodoService {
  private todos: Todo[] = [];

  getAll(): Todo[] {
    return this.todos;
  }

  getById(id: number): Todo {
    return this.todos.find(todo => todo.id === id);
  }

  create(todo: Todo): Todo {
    this.todos.push(todo);
    return todo;
  }

  update(id: number, todo: Todo): Todo {
    const index = this.todos.findIndex(t => t.id === id);
    if (index !== -1) {
      this.todos[index] = todo;
      return todo;
    }
    return null;
  }

  delete(id: number): Todo {
    const index = this.todos.findIndex(todo => todo.id === id);
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

import { Controller, Get, Post, Put, Delete, Param, Body } from '@nestjs/common';
import { TodoService } from './todo.service';
import { Todo } from './todo.entity';

@Controller('todos')
export class TodoController {
  constructor(private readonly todoService: TodoService) {}

  @Get()
  getAll(): Todo[] {
    return this.todoService.getAll();
  }

  @Get(':id')
  getById(@Param('id') id: number): Todo {
    return this.todoService.getById(+id);
  }

  @Post()
  create(@Body() todo: Todo): Todo {
    return this.todoService.create(todo);
  }

  @Put(':id')
  update(@Param('id') id: number, @Body() todo: Todo): Todo {
    return this.todoService.update(+id, todo);
  }

  @Delete(':id')
  delete(@Param('id') id: number): Todo {
    return this.todoService.delete(+id);
  }
}

```
Finally, make sure to register the controller and service in the appropriate module:

```ts
// todo.module.ts

import { Module } from '@nestjs/common';
import { TodoController } from './todo.controller';
import { TodoService } from './todo.service';

@Module({
  controllers: [TodoController],
  providers: [TodoService],
})
export class TodoModule {}
```
**[⬆ Back to Top](#table-of-contents)**