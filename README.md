# \_api

## Table of Contents

- [\_api](#_api)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Foreword](#foreword)
    - [Todo](#todo)
  - [Getting started](#getting-started)
  - [CRUD API](#crud-api)
    - [Endpoints](#endpoints)
      - [Find](#find)
      - [Create](#create)
      - [Update](#update)
      - [Remove](#remove)
  - [Middleware](#middleware)
  - [Helpers](#helpers)
    - [isJSONString()](#isjsonstring)
    - [getTotalCollections()](#gettotalcollections)
    - [responseBuilder()](#responsebuilder)
    - [routerQueries()](#routerqueries)
  - [License](#license)

## Introduction

\_api is an autogenerated CRUD API built on [LowDB](https://github.com/typicode/lowdb) and [ExpressJS](https://expressjs.com/). All you need to do is edit a configuration file and you will have a basic CRUP API ready to use!

### Foreword

\_api came about due to sheer curiosity. It's important to understand however, \_api was not built for large-scale applications, it's underlying database layer [LowDB](https://github.com/typicode/lowdb) is built on pure JSON objects for storage and [Lodash](https://lodash.com) for parsing. This doesn't mean that \_api isn't worth your time, the biggest positive that I see with \_api is the sheer simplicity of both the concept and the code written thereby simplifying development and maintenance.

### Todo

- ~~Improve README.md.~~
- ~~Rewrite API endpoint logic to implement functions such as sort, limit, and operators.~~
- Implement \_gte, \_gt, \_lte, and \_lt comparators to the API for find, update, and remove endpoints.
- Implement schema validation functionality.
- Implement an admin interface.

## Getting started

Download or clone this repository and then:

1. Enter the root directory of \_api.

```bash
cd _api/
```

2. Install the required node modules:

```bash
npm install
```

3. Open up `src/config/db/index.js` in your favorite text editor and start editing:

```js
SECRET: 'MY_SUPER_SECRET_KEY',
COLLECTIONS: [
  {
    users: []
  }
]
```

The `SECRET` property is your super secret key for encrypting and decrypting your collection files. Make this super private and hard to guess! By default, this API encrypts and decrypts the collection files, you can disable this by making it an empty string.

`COLLECTIONS` is where the magic happens — this property is what generates the different collections that make up your database which in turn make up the CRUD API. If you want to add more collections, just add an object into the `COLLECTIONS` array like so:

```js
COLLECTIONS: [
  {
    users: [],
  },
  {
    your_collection: [],
  },
];
```

_You can delete the users collection, it's there as an example._

1. Now you can start the API.

```bash
npm run start
```

or if you prefer to develop with Nodemon;

```bash
npm run watch
```

## CRUD API

\_api is great in that it will automatically generate all the CRUD routes with the data operation functions for you. Below are the auto-generated routes.

### Endpoints

#### Find

Before we go onto the routes, the `find` endpoints are really awesome! They can accept data operators such as filter, sort, and slice. What is even better is that you can chain them! But there is one really important thing to remember before you go chaining your URL queries — the order of operations — the data operators will generally execute from left to right but they do share the same non-guarantee as `_.forEach()` (Lodash).

Find all:

Returns all documents in a collection.

```url
hostname:port/your_collection/
```

Find by id:

You can search for a document in a collection by id. It will only return one document.

```url
hostname:port/your_collection?_id=123
```

Find by filter:

Filter only accepts JSON objects. This means when you pass it into the URL parameters, you will need to convert to a valid JSON string which will need to be converted to % notation — JSON.stringify() does this for you.

```url
hostname:port/your_collection?_filter={"key":"value"}
```

Find by sort and order:

You can sort results by key, by default the order sent back is ascending but by specifying a `_order` parameter you can have descending order. `_order` takes only two values — `asc` and `desc`.

```url
hostname:port/your_collection?_sort=key&_order=desc
```

Find by slice:

You can slice your results by specifying a start index and an end index. Either is optional and will default to `start = 0` and `end = collection.size()`. If you are slicing, an addition header `X-Total-Count` which will hold the total documents in the result returned.

```url
hostname:port/your_collection?_start=10&_end=20
```

#### Create

Create one:

You will need to pass the document values in JSON object defined in your request body. Don't worry about passing an id value as \_api will automatically generate an id for you. The response will be the document created including the unique id for it.

```url
hostname:port/your_collection/create
```

#### Update

Update by id:

You can update a document in the collection by id. The new values are to be passed in by JSON object via the request body. The response will be the document updated.

```url
hostname:port/your_collection/update?_id=123
```

Update by filter:

You can update one or more by filter. Filter only accepts JSON objects. This means when you pass it into the URL parameters, you will need to convert to a valid JSON string which will need to be converted to % notation — JSON.stringify() does this for you. The response will be the document or documents updated.

```url
hostname:port/your_collection/update?_filter={"key":"value"}
```

#### Remove

Remove by id:

You can remove a document in the collection by id. The response will be the document removed.

```url
hostname:port/your_collection/remove?_id=123
```

Remove by filter:

You can remove one or more by filter. Filter only accepts JSON objects. This means when you pass it into the URL parameters, you will need to convert to a valid JSON string which will need to be converted to % notation — JSON.stringify() does this for you. The response will be the document or documents removed.

```url
hostname:port/your_collection/remove?_filter={"key":"value"}
```

## Middleware

If you need to define middleware functions, you can do that easily through the middleware file: `src/middleware/index.js`. All you need to do is define your functions in here and they will be loaded to each collection automatically as lodash mixins (So you can chain them with other middleware or lodash functions). An example:

```js
// Create and export an object to hold middleware function declarations.
const Middleware = {
  helloWorld: () => {
    return 'Hello World!';
  },
};

export default Middleware;
```

To use this, you would then need to modify the API routes in the `src/router/index.js` file.

On-top of these chained middleware functions, you can also utilise any ExpressJS middleware as you normally would. Just make sure you define it before the collection/database generation logic.

## Helpers

The helpers file: `src/helpers/index.js` contain all your reusable functions. Just import this file anywhere you need it and you will have access to the functions defined and exported in there. Currently there are a few helper functions already defined — those are:

### isJSONString()

Checks to see if a string is a valid JSON String which can be parsed into a JSON object. Returns either `true` or `false`.

```js
isJSONString((string: String));
```

### getTotalCollections()

Returns the total number of collections in an object. See `src/config/db` and the `DB_CONFIG.COLLECTION` object for more details as to what the object should look like in terms of structure.

```js
getTotalCollections((collection: Object));
```

### responseBuilder()

This function takes in a HTTP status code, request header, data object, and a message object to then return a JSON object for the response body. If the data object provided is empty, this will return a `404 not found` response body regardless of what the code input was.

```js
responseBuilder((code: Number), (request: Object), (data: Object), (message: Object));
```

### routerQueries()

The magic function that parses through the `request.query` object to fetch and filter, sort, and slice data. This function will return an object with 2 objects within. The structure is:

```js
{
  header: {
    ...
  },
  data: {
    ...
  }
}
```

The router will have the logic to handle this for you but if you are modifying the router — keep this in mind.

```js
routerQueries((request: Object), (collection: Object), (collectionKey: String));
```

## License

The MIT License (MIT)

Copyright (c) 2018 TokenChingy

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
