# sequelize-restful-extended

[![Build Status](https://travis-ci.org/pjanaya/sequelize-restful-extended.svg?branch=master)](https://travis-ci.org/pjanaya/sequelize-restful-extended)

A connect module based on a fork of sequelize-restful. Creates a restful API with associations from your Sequelize models and allows you to include parameters like `sort`, `offset`, `limit` and `order`. It also allows more complex operations like data range filters or include complex WHERE operations with many conditions in the fields. This operations include LIKE, GTE (Greater than or equal ">="), GT (Greater than ">"), LTE (Lower than or equal "<="), LT (Lower than "<") and NE (Not equal "!=").

It uses the Sequelize function `findAndCountAll` instead of `findAll`. Thanks to this, the basic GET request returns the total count of rows in the response. This number doesn't take account of the query parameters `offset` and `limit`. This feature makes easier to do pagination using the generated API.    

## Usage

```js
var express   = require('express')
  , Sequelize = require('sequelize')
  , http      = require('http')
  , restful   = require('sequelize-restful-extended')
  , sequelize = new Sequelize('database', 'username', 'password')
  , app       = express()

// define all your models before the configure block

app.configure(function() {
  app.use(restful(sequelize, { /* options */ }))
})

http.createServer(app).listen(app.get('port'), function(){
  console.log("Express server listening on port " + app.get('port'))
})
```

## Options

```js
{
  // Parameter:   endpoint
  // Description: Define the path to the restful API.
  // Default:     '/api'

  endpoint: '/restful',

  // Parameter:   allowed
  // Description: Define which models will be exposed through the restful API
  // Default:     'new Array()' if it is an Empty array, all the models will be exposed by default

  allowed: new Array('Model0', 'Model1', 'Model2'),

  // Parameter:   extendedMode
  // Description: If it's true, all GET request uses the findAndCountAll function of Sequelize, 
  //              returning the count on the response. If it's false uses the default findAll.
  // Default:     true

  extendedMode: true
}
```

## The API

### GET /api

Returns a list of all declared models

```console
$ curl http://localhost:3000/api
```

```js
{
  "status": "success"
  "data": [
    {
      "name": "Tag",
      "tableName": "Tags"
    },
    {
      "name": "Image",
      "tableName": "Images"
    },
    {
      "name": "Project",
      "tableName": "Projects"
    }
  ]
}
```

### HEAD /api/Tags

Returns a description of a declared model

```console
$ curl -i -X HEAD http://localhost:3000/api/Tags
```

The result of the request is part of the response headers! The header's name is `Sequelize-Admin`.

```js
{
  "status": "success",
  "data": {
    "name": "Tag",
    "tableName": "Tags",
    "attributes": {
      "title": "VARCHAR(255)",
      "id": {
        "type": "INTEGER",
        "allowNull": false,
        "primaryKey": true,
        "autoIncrement": true
      },
      "createdAt": {
        "type": "DATETIME",
        "allowNull": false
      },
      "updatedAt": {
        "type": "DATETIME",
        "allowNull": false
      },
      "ProjectId": {
        "type": "INTEGER"
      }
    }
  }
}
```

### GET /api/Tags

Returns all Tags

```console
$ curl http://localhost:3000/api/Tags
```

```js
{
  "status": "success",
  "count": 3,
  "data": [{
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  },{
    "title": "foo2",
    "id": 2,
    "createdAt": "2013-02-10T09:48:14.000Z",
    "updatedAt": "2013-02-10T09:48:14.000Z",
    "ProjectId": 2
  },{
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }]
}
```

### GET /api/Tags?sort=-title

Returns all Tags sorted by title descending

```console
$ curl http://localhost:3000/api/Tags?sort=-title
```

```js
{
  "status": "success",
  "count": 3,
  "data": [{
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  },{
    "title": "foo2",
    "id": 2,
    "createdAt": "2013-02-10T09:48:14.000Z",
    "updatedAt": "2013-02-10T09:48:14.000Z",
    "ProjectId": 2
  },{
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  }]
}
```

### GET /api/Tags?title=foo

Returns all Tags where title is foo

```console
$ curl http://localhost:3000/api/Tags?title=foo
```

```js
{
  "status": "success",
  "count": 1,
  "data": {
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  }
}
```

### GET /api/Tags?title_like=foo

Returns all Tags where title is LIKE foo (in this example, all of them)

```console
$ curl http://localhost:3000/api/Tags?title_like=foo
```

```js
{
  "status": "success",
  "count": 3,
  "data": [{
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  },{
    "title": "foo2",
    "id": 2,
    "createdAt": "2013-02-10T09:48:14.000Z",
    "updatedAt": "2013-02-10T09:48:14.000Z",
    "ProjectId": 2
  },{
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }]
}
```

### GET /api/Tags?id_gte=2

Returns all Tags where ID is >= 2

```console
$ curl http://localhost:3000/api/Tags?id_gte=2
```

```js
{
  "status": "success",
  "count": 2,
  "data": [{
    "title": "foo2",
    "id": 2,
    "createdAt": "2013-02-10T09:48:14.000Z",
    "updatedAt": "2013-02-10T09:48:14.000Z",
    "ProjectId": 2
  },{
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }]
}
```

### GET /api/Tags?id_gt=2

Returns all Tags where ID is > 2

```console
$ curl http://localhost:3000/api/Tags?id_gt=2
```

```js
{
  "status": "success",
  "count": 1,
  "data": {
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }
}
```

### GET /api/Tags?id_lte=2

Returns all Tags where id is <= 2

```console
$ curl http://localhost:3000/api/Tags?id_lte=2
```

```js
{
  "status": "success",
  "count": 2,
  "data": [{
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  },{
    "title": "foo2",
    "id": 2,
    "createdAt": "2013-02-10T09:48:14.000Z",
    "updatedAt": "2013-02-10T09:48:14.000Z",
    "ProjectId": 2
  }]
}
```

### GET /api/Tags?id_lt=2

Returns all Tags where id is < 2

```console
$ curl http://localhost:3000/api/Tags?id_lt=2
```

```js
{
  "status": "success",
  "count": 1,
  "data": {
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  }
}
```

### GET /api/Tags?id_ne=2

Returns all Tags where id is != 2

```console
$ curl http://localhost:3000/api/Tags?id_ne=2
```

```js
{
  "status": "success",
  "count": 2,
  "data": [{
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  },{
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }]
}
```

### GET /api/Tags?limit=2

Returns the first two tags (Notice the count attribute in the response indicating the total tags)

```console
$ curl http://localhost:3000/api/Tags?limit=2
```

```js
{
  "status": "success",
  "count": 3,
  "data": [{
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  },{
    "title": "foo2",
    "id": 2,
    "createdAt": "2013-02-10T09:48:14.000Z",
    "updatedAt": "2013-02-10T09:48:14.000Z",
    "ProjectId": 2
  }]
}
```


### GET /api/Tags?limit=2&offset=2

Returns the first two tags after a specific amount of elements (Notice the count attribute in the response indicating the total tags)

```console
$ curl http://localhost:3000/api/Tags?limit=2&offset=2
```

```js
{
  "status": "success",
  "count": 3,
  "data": {
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }
}
```
### GET /api/Tags?createdAt_start=2013-02-11&createdAt_end=2013-02-12

Returns all tags created between two dates

```console
$ curl http://localhost:3000/api/Tags?createdAt_start=2013-02-11&createdAt_end=2013-02-12
```

```js
{
  "status": "success",
  "count": 1,
  "data": {
    "title": "foo3",
    "id": 3,
    "createdAt": "2013-02-11T09:48:14.000Z",
    "updatedAt": "2013-02-11T09:48:14.000Z",
    "ProjectId": 2
  }
}
```

### GET /api/Tags/1

Returns the data of the Tag with the id 1.

```console
$ curl http://localhost:3000/api/Tags/1
```

```js
{
  "status": "success",
  "data": {
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  }
}
```

### POST /api/Tags

Creating a new instance of a model

```console
curl -d "title=hallo%20world" http://localhost:3000/api/Tags
```

```js
{
  "status": "success",
  "data": {
    "title": "hallo world",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z"
  }
}
```

### PUT /api/Tags/1

Updating an already existing instance of a model

```console
curl -d "title=fnord" -X PUT http://localhost:3000/api/Tags/1
```

It returns the updated record

```js
{
  "status": "success",
  "data": {
    "title": "fnord",
    "id": 1,
    "createdAt": "2013-02-14T19:52:04.000Z",
    "updatedAt": "2013-02-14T19:53:30.066Z",
    "ProjectId": 1
  }
}
```

### DELETE /api/Tags/1

Deleting an existing instance of a model

```console
curl -i -X DELETE http://localhost:3000/admin/api/Tags/3
```

```js
{
  "status": "success",
  "data": {}
}
```

## The API for Associations

### GET /api/Projects/1/Tags

Returns all the instance of 'associated_dao_factory' associated to the instance 1 of 'dao_factory'

```console
curl -i -X GET http://localhost:3000/admin/api/Projects/1/Tags

```

```js
{
  "status": "success",
  "data": {
    "title": "foo",
    "id": 1,
    "createdAt": "2013-02-09T09:48:14.000Z",
    "updatedAt": "2013-02-09T09:48:14.000Z",
    "ProjectId": 1
  }
}
```

### DELETE /api/Photo/1/Photographer

Deleting an existing association for 1:1 or N:1 association.

```console
curl -i -X DELETE http://localhost:3000/admin/api/Photo/1/Photographer
```

```js
{
  "status": "success",
  "data": {}
}
```

### DELETE /api/Projects/1/Tags/1

Deleting an existing association between instances

```console
curl -i -X DELETE http://localhost:3000/admin/api/Projects/1/Tags/3
```

```js
{
  "status": "success",
  "data": {}
}
```
