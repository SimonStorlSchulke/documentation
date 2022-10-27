---
title: Unit and Route Testing - Strapi Developer Docs
description: Learn in this guide how you can run basic unit tests for a Strapi application using a testing framework.
sidebarDepth: 2
canonicalUrl: https://docs.strapi.io/developer-docs/latest/guides/unit-testing.html
---

# Unit and Route Testing

Testing code units and API routes in a Strapi application can be done with [Jest](https://jestjs.io/) and [Supertest](https://github.com/visionmedia/supertest), with an SQLite database. This documentation describes API endpoint unit test and an API endpoint unit test with authorization. Refer to the testing framework documentation for other use cases.

::: tip
In this example we will use  Testing Framework with a focus on simplicity and
 Super-agent driven library for testing node.js HTTP servers using a fluent API
:::

:::caution
Please note that this guide will not work if you are on Windows using the SQLite database due to how windows locks the SQLite file.
:::

## Install and configure the test tools

`Jest` contains a set of guidelines or rules used for creating and designing test cases - a combination of practices and tools that are designed to help testers test more efficiently.

`Supertest` allows you to test all the `api` routes as if they were instances of [http.Server](https://nodejs.org/api/http.md#http_class_http_server)

`better-sqlite3` is used to create an on-disk database that is created and deleted between tests.
<!-- TODO rewrite this intro section-->

### Install for JavaScript applications

1. Add the tools to the dev dependencies:

<code-group>
<code-block title=JAVASCRIPT>

```sh
yarn add jest --dev
yarn add supertest --dev 
yarn add better-sqlite3 --dev 

OR

npm install jest --save-dev
npm install supertest --save-dev
npm install better-sqlite3 --save-dev
  ```
  
  </code-block>

<code-block title=TYPESCRIPT>

<!-- TODO: test the TS commands-->
```sh
yarn add jest ts-jest @types/jest --dev
yarn add supertest --dev 
yarn add better-sqlite3 --dev 

OR

npm install jest ts-jest @types/jest --save-dev
npm install @types/supertest --save-dev
npm install better-sqlite3 --save-dev
```

</code-block>
</code-group>

2. Add `test` to the `package.json` file `scripts` section:

``` json{6}
  "scripts": {
    "develop": "strapi develop",
    "start": "strapi start",
    "build": "strapi build",
    "strapi": "strapi",
    "test": "jest --forceExit --detectOpenHandles"
  },
```

3. Add a `jest` section to the `package.json` file with the following code:

```json
//path: ./package.json

  "jest": {
    "testPathIgnorePatterns": [ //informs Jest to ignore these directories.
      "/node_modules/",
      ".tmp",
      ".cache"
    ],
    "testEnvironment": "node"
  }
```

4.(TypeScript only) Create a Jest-TypeScript configuration file:

<code-group>
<code-block title=YARN>

```sh
yarn ts-jest config:init
```

</code-block>
<code-block title=NPM>

```sh
npx ts-jest config:init
```

</code-block>
</code-group>


## Create a testing environment

The testing environment should test the application code without affecting the database, and should be able to run distinct units of the application to incrementally test the code functionality. A testing database and a testing Strapi instance are necessary to successfully run unit tests.

### Create a test environment database configuration file

The test framework must have a clean and empty environment to perform valid tests and to not interfere with the development database database.

Once `jest` is running it uses the `test` [environment](/developer-docs/latest/setup-deployment-guides/configurations/optional/environment.md) by switching `NODE_ENV` to `test`.

1. Create a new database configuration file for the test env: `./config/env/test/database.js`.
2. add the following code to `./config/env/test/database.js:

```js
// path: ./config/env/test/database.js

const path = require('path');

module.exports = ({ env }) => ({
  connection: {
    client: 'sqlite',
    connection: {
      filename: path.join(__dirname, '..', env('DATABASE_FILENAME', '.tmp/data.db')),
    },
    useNullAsDefault: true,
  },
});
```

### Create a `strapi` instance

In order to test anything we need to have a strapi instance that runs in the testing environment,
basically we want to get instance of strapi app as an object, similar to creating an instance for the [process manager](process-manager.md).

These tasks require adding some files - let's create a folder `tests` where all the tests will be put and inside it, next to folder `helpers` where main Strapi helper will be in file strapi.js.

1. Create a `tests` directory at the application root.
2. Create a `helpers` directory inside `tests`.
3. Create the file `strapi.js` in the `helpers` directory and add the following code:

```js
//path: ./tests/helpers/strapi.js
const Strapi = require("@strapi/strapi");
const fs = require("fs");

let instance;

async function setupStrapi() {
  if (!instance) {
    await Strapi().load();
    instance = strapi;
    
    await instance.server.mount();
  }
  return instance;
}

async function teardownStrapi() {
  const dbSettings = strapi.config.get("database.connection");

  //close server to release the db-file
  await strapi.server.httpServer.close();

  // close the connection to the database before deletion
  await strapi.db.connection.destroy();

  //delete test database after all tests have completed
  if (dbSettings && dbSettings.connection && dbSettings.connection.filename) {
    const tmpDbFile = dbSettings.connection.filename;
    if (fs.existsSync(tmpDbFile)) {
      fs.unlinkSync(tmpDbFile);
    }
  }
}

module.exports = { setupStrapi, teardownStrapi };
```

::: note
The command to close the database connection is not working, which results in an open handle in Jest. The `--force-exit` flag temporarily solves this problem.
:::

## Test the `strapi` instance

You need a main entry file for the tests, one that will also test the helper file. To do this create `app.test.js` in the `tests` directory and add the following code:

```js
//path: ./tests/app.test.js

const fs = require('fs');
const { setupStrapi, teardownStrapi } = require("./helpers/strapi");

beforeAll(async () => {
  await setupStrapi();
});

afterAll(async () => {
  await cleanupStrapi();
});

it("strapi is defined", () => {
  expect(strapi).toBeDefined();
});
```

Run the unit test to confirm it is working correctly:

<code-group>
<code-block title=YARN>

```sh
yarn test
```
</code-block>
<code-block title=NPM>

```sh
npm test
```
</code-block>
</code-group>

The test output in your terminal should be the following:

```bash
yarn run v1.22.18
$ jest --forceExit --detectOpenHandles
 PASS  tests/app.test.js
  ✓ strapi is defined (1 ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        2.043 s, estimated 3 s
Ran all test suites.
✨  Done in 2.90s.
```

:::tip
If you receive a timeout error for Jest, please add the following line right before the `beforeAll` method in the `app.test.js` file: `jest.setTimeout(15000)` and adjust the milliseconds value as you need.
:::

## Use the testing environment

With the testing environment set up, you can create and run tests on functions or API routes. To construct your own tests you need to:

1. Add the test criteria to the `app.test.js` file or write a new `your-file-name.test.js` file.
2. Add the file path for the path for the code to be tested to the `app.test.js`.
3. Run the test.

The following documentation provides examples for how to setup:

- unit tests,
- public API endpoints,
- authenticated API endpoints

### Run a unit test

Unit tests are designed to test individual units such as functions and methods. The following procedure sets up a unit test for a function to demonstrate the functionality:

1. Create the file `sum.js` at the application root.
2. Add the following code to the `sum.js` file:

    ```js
    function sum(a, b) {
    return a + b;
      }
      module.exports = sum;
  
    ```

3. Add the location of the code to be tested to the `app.test.js` file:

    ```js{4}

    // path: ./tests/app.test.js
    //...
    const sum = require('../sum');
    //...

4. Add the test criteria to the `app.test.js` file:

```js{4-6}

    // path: ./tests/app.test.js
    //...
    test('adds 1 + 2 to equal 3', () => {
    expect(sum(1, 2)).toBe(3);
    });
    //...

```

5. Save the files and run `yarn test` or `npm test` in the project root directory. 


### Test a public endpoint

:::prerequisite
<!--TODO: add links-->
This test requires a public endpoint. Create an API using the `strapi generate` CLI command and allow public access to the `get` route.

:::

The goal of this test is to evaluate if the endpoint works properly and if the route returns a specified string.

1. Create a test file `publicRoute.js` in `./tests`.
2. Add the following code to `publicRoute.js`:

    ```js

    // path: ./tests/hello/index.js

    const request = require('supertest');

    it("should return some text here", async () => {
      await request(strapi.server.httpServer)
        .get("/api/your-api-route") //add your API route here
        .expect(200) // Expect response http code 200
        .then((data) => {
          expect(data.text).toBe("some text here"); // expect the response text
        });
    });

    ```
3. Customize the `.get` route.
4. (optional) Customize the response text.
5. Add the following code to `./tests/app.test.js

    ```js
    require('./publicRoute');

    ```
6. Save your code changes.
7. run `yarn test` or `npm test`

:::tip
If you receive an error `Jest has detected the following 1 open handles potentially keeping Jest from exiting` check `jest` version as `26.6.3` works without an issue.
:::

### test an authenticated API endpoint

In this scenario we'll test authentication login endpoint with two tests

- Test `/auth/local` that should login user and return `jwt` token
- Test `/users/me` that should return users data based on `Authorization` header

<!--This code example does not work as-is-->


```js
// path: `./tests/user/index.js`

const request = require('supertest');

// user mock data
const mockUserData = {
  username: "tester",
  email: "tester@strapi.com",
  provider: "local",
  password: "1234abc",
  confirmed: true,
  blocked: null,
};

it("should login user and return jwt token", async () => {
  /** Creates a new user and save it to the database */
  await strapi.plugins["users-permissions"].services.user.add({
    ...mockUserData,
  });

  await request(strapi.server.httpServer) // app server is an instance of Class: http.Server
    .post("/api/auth/local")
    .set("accept", "application/json")
    .set("Content-Type", "application/json")
    .send({
      identifier: mockUserData.email,
      password: mockUserData.password,
    })
    .expect("Content-Type", /json/)
    .expect(200)
    .then((data) => {
      expect(data.body.jwt).toBeDefined();
    });
});

it('should return users data for authenticated user', async () => {
  /** Gets the default user role */
  const defaultRole = await strapi.query('plugin::users-permissions.role').findOne({}, []);

  const role = defaultRole ? defaultRole.id : null;

  /** Creates a new user an push to database */
  const user = await strapi.plugins['users-permissions'].services.user.add({
    ...mockUserData,
    username: 'tester2',
    email: 'tester2@strapi.com',
    role,
  });

  const jwt = strapi.plugins['users-permissions'].services.jwt.issue({
    id: user.id,
  });

  await request(strapi.server.httpServer) // app server is an instance of Class: http.Server
    .get('/api/users/me')
    .set('accept', 'application/json')
    .set('Content-Type', 'application/json')
    .set('Authorization', 'Bearer ' + jwt)
    .expect('Content-Type', /json/)
    .expect(200)
    .then(data => {
      expect(data.body).toBeDefined();
      expect(data.body.id).toBe(user.id);
      expect(data.body.username).toBe(user.username);
      expect(data.body.email).toBe(user.email);
    });
});
```

Then include this code to `./tests/app.test.js` at the bottom of that file

```js
require('./user');
```

All the tests above should return an console output like

```bash
➜  my-project git:(master) yarn test

yarn run v1.13.0
$ jest --forceExit --detectOpenHandles
[2020-05-27T08:30:30.811Z] debug GET /hello (10 ms) 200
[2020-05-27T08:30:31.864Z] debug POST /auth/local (891 ms) 200
 PASS  tests/app.test.js (6.811 s)
  ✓ strapi is defined (3 ms)
  ✓ should return hello world (54 ms)
  ✓ should login user and return jwt token (1049 ms)
  ✓ should return users data for authenticated user (163 ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        6.874 s, estimated 9 s
Ran all test suites.
✨  Done in 8.40s.
```