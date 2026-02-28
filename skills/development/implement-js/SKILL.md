---
name: implement-js
description: Implement JavaScript code based on user requirements.
globs: ["*.js", "package.json"]
---

When implementing JavaScript code, follow these guidelines.

## Code format guide:

 * Use tabs and double quotes for strings

 * Make sure to follow style guide from eslint or/and editorconfig if present in the project

 * Use Common.JS syntax for imports and exports

 * I do not like to use this format `const { recordEvent } = require("./event-log");`, instead, do this `const recordEvent = require("./event-log").recordEvent;`

 * Write exports directly in `module.exports = {};`

 * Expose constants at the top of the file, after imports, for easier configuration

 * Use JSDoc when function parameters are ambiguous (e.g., options objects with multiple properties)

 * Prefer if/else over ternaries for non-trivial logic - readability over cleverness

## Project structure

 * Make sure to always create folder for each module, and create index.js inside, it helps with organization and reduces chance of conflict

 * I like to keep code for talking to other services in `src/services` folder

 * I like to keep helper code that is specification specific in `src/utils` folder. If it is not logically part of application, just something that is needed to make things work, it should go to utils

 * I create entrypoints into project in `bin/` folder (e.g., `bin/pull.js`, `bin/sync.js`)

 * All entrypoints must be:
   * Started with `forever` (e.g., `forever bin/pull.js`)
   * Added to `package.json` scripts (e.g., `"pull": "forever bin/pull.js"`)
   * Added to `Procfile` for herokuish (e.g., `pull: yarn pull`)
   * Have a `-watch` variant using `nodemon` for development (e.g., `"pull-watch": "nodemon bin/pull.js"`)

 * If `nodemon` is not in package.json, install it as dev dependency (`yarn add -D nodemon`)

 * Scripts run in heroku/herokuish environment

## Technical guidelines

 * Use Promise instead of async/await for asynchronous operations

 * Chain promises sequentially with `.then()` rather than nesting:

   ```javascript
   return doFirst()
       .then(() => {
           return doSecond();
       })
       .then(() => {
           return doThird();
       });
   ```

 * Avoid using ES6+ features that are not widely supported in Node.js environments

 * I like functional JS, map/reduce/foreach/filter should be preferred over for loops

 * I like to use wrappers for repeating code, like rate limiting, cachine, etc.

 * Never use `var`, always `const`/`let`. Unless you really need to change value, use `const`

 * Never change parameters of functions, always create new variables if you need to change something

 * Handle errors at the last possible moment to avoid losing error data when rebuilding. Catch earlier only when it makes sense for the logic

## Architectural guidelines

 * Try to encapsulate complex logic in modules and expose logic so that final code looks almost like pseudocode

 * Name modules based off their functionality, not how they are implemented (for example, module for logging stuff to mongo should be called logger, not mongo)

 * Where it makes sense, if there is array getter, also implement getter that returns map, calling original getter and transforming result to map

## Service pattern

 * Services should be defined as objects with methods, not separate functions

 * Use `_client` for the HTTP client (private by convention), do not export it

 * Reference the service object by name inside methods

 * Example:

   ```javascript
   const got = require("got-verbose");
   const config = require("../../config");

   const myService = {
       _client: got.extend({
           prefixUrl: "https://api.example.com",
           headers: {
               authorization: "Bearer " + config.get("MY_API_KEY")
           }
       }),

       getItems: () => {
           return myService._client.get("items").then((response) => {
               return JSON.parse(response.body).items;
           });
       },

       getItemsMap: () => {
           return myService.getItems().then((items) => {
               const map = {};
               items.forEach((item) => {
                   map[item.id] = item;
               });
               return map;
           });
       }
   };

   module.exports = myService;
   ```

## Package manager

 * Always use `yarn` instead of `npm`, unless there is a `package-lock.json` present (then use `npm`)

 * Never guess npm package versions — use `yarn add <package>` (or `yarn add -D <package>` for dev dependencies) and let yarn resolve the correct version. Do not manually edit dependency versions in `package.json`.

## Version control

 * Make sure there is `.gitignore` file present

   * Make sure to ignore `node_modules`

## Libraries and tools

 * Lib I like to use for rate limiting (how many calls of method should happen at the same time) is queue-promised

    * Install it, package name is `queue-promised`

    * Import method `wrapper` from `queue-promised` (`const wrapper = require("queue-promised").wrapper;`)

    * Use it like this `const limitedFunction = wrapper(originalFunction, 5);` where 5 is how many calls can happen at the same time

 * I like to use `uuid` package for generating uuids. I like to use `v4` method from it. and I import it like this: `const uuid = require("uuid").v4;`

 * I like to use `node-cron` for cron jobs. I import it like this: `const cron = require("node-cron");`

 * I like to use `got-verbose`, which is a wrapper around `got` library for HTTP requests. It exposes identical API as `got`, but has built-in logging and error handling. I import it like this: `const got = require("got-verbose");`

   * **Note:** `got-verbose` does not work with streams. For streaming, use `got` directly (`const got = require("got");`)

   * Avoid using .json() method of got-verbose and json option, instead use .body and parse JSON manually, it does have some unexpected behavior

 * I like to use `forever` for running Node.js applications. Use it in the start script: `"start": "forever src/index.js"`

## Claude notice

 * Add note to CLAUDE.md that this skill should be always used

 * Suggest changes to this skill, when I suggest code style changes

 * If you notice a pattern in my instructions, suggest adding it to this skill

 * Try to keep CLAUDE.md in sync with project changes

 * If you are writing a pseudocode (calling method or API endpoint you are not sure exists), add TODO comment so I know I should fix it

## Building container

It is usual for me to use herokuish build like this

```
FROM gliderlabs/herokuish:latest

COPY . /app

WORKDIR /app

RUN /bin/herokuish buildpack build

ENV PORT 3000

EXPOSE 3000
```

## Cleanup time

When I tell you it is cleanup time, follow this checklist:

 * /implement-js

 * update docs (README.md)

 * update your docs (CLAUDE.md)

 * update missing jsdoc...

 * check tests

 * check linter
