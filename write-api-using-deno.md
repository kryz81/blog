In this post, I will show you how to create a small API using **Deno** - the newest runtime to run Javascript and Typescript, created by the author of Node.js - Ryan Dahl.

If you don't know what Deno is, check this article: **[Getting started with Deno](https://dev.to/wuz/getting-started-with-deno-e1m).**

**Our goal is to:**

* Create an API which manages users
* Provide GET, POST, PUT and DELETE routes
* Save created/updated users to a local JSON file
* Use a web framework to speed up the development process

The only tool you need to install is Deno itself. Deno supports Typescript out of the box. For this example, I used the 0.22 version. The Deno API is **still under a continuous development**, and this code may not work with other versions. Check your version using: **deno version** command in the terminal.

## Let's start

You can find the code below on Github: [github.com/kryz81/deno-api-example](https://github.com/kryz81/deno-api-example)

### Step 1: Program structure

```console
handlers
middlewares
models
services
config.ts
index.ts
routing.ts
```

As you see it looks like a small Node.js web application:

* **handlers** contains route handlers
* **middlewares** provide functions that run on every request
* **models** contain model definitions, in our case only User interface
* **services** contains... services
* **config.ts** contains global application configuration
* **index.ts** is the entry point of the application
* **routing.ts** contains API routes


### Step 2: Choose a web framework

There are many great web frameworks for Node.js. The most popular one is **Express**. There is also a modern version of Express - **Koa**. But Deno is not compatible with Node.js, and we cannot use Node.js libraries. In the case of Deno, the choice is currently much smaller, but there is a framework inspired by Koa - **Oak**. Let's use it for our example. If you've never used Koa, don't worry, it looks almost the same as Express.

### Step 3: Create the main file

_index.ts_

```typescript
import { Application } from "https://deno.land/x/oak/mod.ts";
import { APP_HOST, APP_PORT } from "./config.ts";
import router from "./routing.ts";
import notFound from "./handlers/notFound.ts";
import errorMiddleware from "./middlewares/error.ts";

const app = new Application();

app.use(errorMiddleware);
app.use(router.routes());
app.use(router.allowedMethods());
app.use(notFound);

console.log(`Listening on ${APP_PORT}...`);

await app.listen(`${APP_HOST}:${APP_PORT}`);
```

In the first line, we use the Deno feature - **importing modules directly from the internet**. Besides that, there is nothing special here. We create an application, add middleware, routes, and finally start the server. Just like in Express/Koa.


### Step 4: Create a configuration

_config.ts_

```typescript
const env = Deno.env();
export const APP_HOST = env.APP_HOST || "127.0.0.1";
export const APP_PORT = env.APP_PORT || 4000;
export const DB_PATH = env.DB_PATH || "./db/users.json";
```
Our configuration is flexible, settings are read from the environment, but we also provide default values used during development. **Deno.env()** is an equivalent of Node.js **process.env**.

### Step 5: Add user model

_models/user.ts_

```typescript
export interface User {
  id: string;
  name: string;
  role: string;
  jiraAdmin: boolean;
  added: Date;
}
```

We need this interface for proper typing.

### Step 6: Add routes

_routing.ts_

```typescript
import { Router } from "https://deno.land/x/oak/mod.ts";

import getUsers from "./handlers/getUsers.ts";
import getUserDetails from "./handlers/getUserDetails.ts";
import createUser from "./handlers/createUser.ts";
import updateUser from "./handlers/updateUser.ts";
import deleteUser from "./handlers/deleteUser.ts";

const router = new Router();

router
  .get("/users", getUsers)
  .get("/users/:id", getUserDetails)
  .post("/users", createUser)
  .put("/users/:id", updateUser)
  .delete("/users/:id", deleteUser);

export default router;
```

Again, nothing special, we create a router and add routes. It looks almost like a copy/paste from an Express.js application!

### Step 7: Add route handlers

_handlers/getUsers.ts_

```typescript
import { getUsers } from "../services/users.ts";

export default async ({ response }) => {
  response.body = await getUsers();
};
```

It returns all users. If you've never used Koa, the **response** object is like **res** in Express. The res object in Express has some methods like **json** or **send**,  to return a response. In Koa/Oak, we need to attach our response value to the **response.body** property.

_handlers/getUserDetails.ts_

```typescript
import { getUser } from "../services/users.ts";

export default async ({ params, response }) => {
  const userId = params.id;

  if (!userId) {
    response.status = 400;
    response.body = { msg: "Invalid user id" };
    return;
  }

  const foundUser = await getUser(userId);
  if (!foundUser) {
    response.status = 404;
    response.body = { msg: `User with ID ${userId} not found` };
    return;
  }

  response.body = foundUser;
};
```

It returns the user with the given id.

_handlers/createUser.ts_

```typescript
import { createUser } from "../services/users.ts";

export default async ({ request, response }) => {
  if (!request.hasBody) {
    response.status = 400;
    response.body = { msg: "Invalid user data" };
    return;
  }

  const {
    value: { name, role, jiraAdmin }
  } = await request.body();

  if (!name || !role) {
    response.status = 422;
    response.body = { msg: "Incorrect user data. Name and role are required" };
    return;
  }

  const userId = await createUser({ name, role, jiraAdmin });

  response.body = { msg: "User created", userId };
};
```

This handler manages user creation.

_handlers/updateUser.ts_

```typescript
import { updateUser } from "../services/users.ts";

export default async ({ params, request, response }) => {
  const userId = params.id;

  if (!userId) {
    response.status = 400;
    response.body = { msg: "Invalid user id" };
    return;
  }

  if (!request.hasBody) {
    response.status = 400;
    response.body = { msg: "Invalid user data" };
    return;
  }

  const {
    value: { name, role, jiraAdmin }
  } = await request.body();

  await updateUser(userId, { name, role, jiraAdmin });

  response.body = { msg: "User updated" };
};
```

The update handler checks if the user with the given ID exists and updates user data.

_handlers/deleteUser.ts_

```typescript
import { deleteUser, getUser } from "../services/users.ts";

export default async ({ params, response }) => {
  const userId = params.id;

  if (!userId) {
    response.status = 400;
    response.body = { msg: "Invalid user id" };
    return;
  }

  const foundUser = await getUser(userId);
  if (!foundUser) {
    response.status = 404;
    response.body = { msg: `User with ID ${userId} not found` };
    return;
  }

  await deleteUser(userId);
  response.body = { msg: "User deleted" };
};
```

This handler deletes a user.

We would also like to handle non-exiting routes and return an error message:

_handlers/notFound.ts_

```typescript
export default ({ response }) => {
  response.status = 404;
  response.body = { msg: "Not Found" };
};
```

### Step 8: Add services
Before we create the user service, we need to create two small **helper** services.

_services/createId.ts_

```typescript
import { v4 as uuid } from "https://deno.land/std/uuid/mod.ts";

export default () => uuid();
```

Each new user gets a unique id, and for that, we will use **uuid** module from the Deno standard library.

_services/db.ts_

```typescript
import { DB_PATH } from "../config.ts";
import { User } from "../models/user.ts";

export const fetchData = async (): Promise<User[]> => {
  const data = await Deno.readFile(DB_PATH);

  const decoder = new TextDecoder();
  const decodedData = decoder.decode(data);

  return JSON.parse(decodedData);
};

export const persistData = async (data): Promise<void> => {
  const encoder = new TextEncoder();
  await Deno.writeFile(DB_PATH, encoder.encode(JSON.stringify(data)));
};
```

This service helps us to interact with our fake users' storage, which is a local json file in our case. To fetch users, we read the file content. The **readFile** function returns an **Uint8Array** object, which needs to be converted to a **string** before parsing to **JSON**. Both Uint8Array and TextDecoder come from **core Javascript API**. Similarly, the data to persist needs to be converted from **string** to **Uint8Array**.

Finally, here is the main service responsible for managing user data:

_services/users.ts_

```typescript
import { fetchData, persistData } from "./db.ts";
import { User } from "../models/user.ts";
import createId from "../services/createId.ts";

type UserData = Pick<User, "name" | "role" | "jiraAdmin">;

export const getUsers = async (): Promise<User[]> => {
  const users = await fetchData();

  // sort by name
  return users.sort((a, b) => a.name.localeCompare(b.name));
};

export const getUser = async (userId: string): Promise<User | undefined> => {
  const users = await fetchData();

  return users.find(({ id }) => id === userId);
};

export const createUser = async (userData: UserData): Promise<string> => {
  const users = await fetchData();

  const newUser: User = {
    id: createId(),
    name: String(userData.name),
    role: String(userData.role),
    jiraAdmin: "jiraAdmin" in userData ? Boolean(userData.jiraAdmin) : false,
    added: new Date()
  };

  await persistData([...users, newUser]);

  return newUser.id;
};

export const updateUser = async (
  userId: string,
  userData: UserData
): Promise<void> => {
  const user = await getUser(userId);

  if (!user) {
    throw new Error("User not found");
  }

  const updatedUser = {
    ...user,
    name: userData.name !== undefined ? String(userData.name) : user.name,
    role: userData.role !== undefined ? String(userData.role) : user.role,
    jiraAdmin:
      userData.jiraAdmin !== undefined
        ? Boolean(userData.jiraAdmin)
        : user.jiraAdmin
  };

  const users = await fetchData();
  const filteredUsers = users.filter(user => user.id !== userId);

  persistData([...filteredUsers, updatedUser]);
};

export const deleteUser = async (userId: string): Promise<void> => {
  const users = await getUsers();
  const filteredUsers = users.filter(user => user.id !== userId);

  persistData(filteredUsers);
};
```

There is a lot of code here, but it's a standard typescript.

### Step 9: Add error handling middleware
What could be the worse that would happen if the user service gave an error? The whole program would crash. To avoid it, we could add **try/catch** block in each handler, but there is a better solution - **add a middleware before all routes and catch all unexpected errors there.**

_middlewares/error.ts_

```typescript
export default async ({ response }, next) => {
  try {
    await next();
  } catch (err) {
    response.status = 500;
    response.body = { msg: err.message };
  }
};
```

### Step 10: Add example data
Before we run our program we will add some example data.

_db/users.json_

```json
[
  {
    "id": "1",
    "name": "Daniel",
    "role": "Software Architect",
    "jiraAdmin": true,
    "added": "2017-10-15"
  },
  {
    "id": "2",
    "name": "Markus",
    "role": "Frontend Engineer",
    "jiraAdmin": false,
    "added": "2018-09-01"
  }
]
```

**That's all. Great! Now we are ready to run our API:**

```
deno -A index.ts
```

The "A" flag means that we don't need to grant permissions on the program run manually. For development purposes, we will allow all of them. Keep in mind that it wouldn't be safe to do it in the production environment.

You should see a lot of **Download** and **Compile** lines, finally we see:

```
Listening on 4000...
```

## Summary

**What did we use:**

* Global **Deno** object to write to and read files
* **uuid** from the Deno standard library to create a unique id
* **oak** - a third-party framework inspired by Node.js Koa framework
* The rest ist pure typescript, objects such as **TextEncoder** or **JSON** are standard Javascript objects

**How does this differ from Node.js:**

* We don't need to install and configure the typescript compiler or other tools like ts-node. We can just run the program using **deno index.ts**
* We import all external modules directly in the code and don't need to install them before we start to implement our application
* There is no package.json and package-lock.json
* There is no node_modules in the root directory of the program; our files are stored in a global cache

You can find the full source code here: [https://github.com/kryz81/deno-api-example](https://github.com/kryz81/deno-api-example)

Do you have any queries? If so, kindly leave a comment below. If you like the article, please tweet it.
