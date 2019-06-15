# Pagination mixin for LoopBack

Paginator adds an easy to use pagination to any of your models. See the [example](#usage) below.

## Installation

```
$ npm i loopback-paginator-mixin --save
```

## Config

### Server Config

With [loopback-boot@v2.8.0](https://github.com/strongloop/loopback-boot/) [mixinSources](https://github.com/strongloop/loopback-boot/pull/131) have been implemented in a way which allows for loading this mixin without changes to the server.js file previously required. Just add `"../node_modules/loopback-paginator-mixin"` to the `mixins` property of your `server/model-config.json`.

```javascript
{
  "_meta": {
    "mixins": [
      "loopback/common/mixins",
      "../node_modules/loopback-paginator-mixin/lib",
      "../common/mixins"
    ]
  }
}
```

### Model Config

To use with your models just add `Paginator: true` to `mixins` in your model config and the default options will be used:

```javascript
{
  "name": "Model",
  "properties": {
    "name": {
      "type": "string",
    }
  },
  "mixins": {
    "Paginator": true
  }

  ...

  // you can also overide the default values:
  "mixins": {
    "Paginator": {
      "limit": 5,         // items per page, default: 10
      "maxLimit": 60,     // max items per page, default: 100
      "noMaxLimit": true, // only use this, if you know what you are doing!
      "firstPage": true,  // paginate to page=1, by default
      "methods" : ['find', 'search', 'prototype.__get__books'] // remote methods applaying paginator hooks
    }
  }
}
```

#### Model Config Options

* `limit` is the default limit to be used for this model
* `maxLimit` is the default maximum number of items per page for this model. As you can override the default limit using the [LoopBack limit filter](https://loopback.io/doc/en/lb3/Limit-filter.html) it might come in handy to set a maxLimit to prevent your API from being abused. The default is `100`.
* `noMaxLimit` set to `true` will deactivate the `maxLimit`. Be careful!
* `firstPage` set to `true` will paginate by default to page=1 (by default), set to `false` will return unpaginated results.
* `methods` are remote methods which are planning to apply paginator hooks. The default applied method is ['find'].

### Global Config

It is also possible to configure the mixin globally in your `config.json`. Just add `paginator` and use the same options as with the model above:

```javascript
{
  "paginator": {
    "limit": 20,        // items per page, default: 10
    "maxLimit": 300,    // max items per page, default: 100
    "noMaxLimit": true, // only use this, if you know what you are doing!
    "firstPage": true,  // paginate to page=1, by default
    "methods" : ['find', 'search', 'prototype.__get__books'] // remote methods applaying paginator hooks
  }
}
```

#### Global Config Options

* `limit` is the default limit to be used globally
* `maxLimit` is the default maximum number of items per page globally. As you can override the default limit using the [LoopBack limit filter](https://loopback.io/doc/en/lb3/Limit-filter.html) it might come in handy to set a maxLimit to prevent your API from being abused. The global default is `100`.
* `noMaxLimit` set to `true` will deactivate the `maxLimit`. Be careful!
* `firstPage` set to `true` will paginate by default to page=1 (by default), set to `false` will return unpaginated results.
* `methods` are remote methods which are planning to apply paginator hooks. The default applied method is ['find'].

## Usage

When Paginator is added to a model and the `page` query parameter is present (e.g. `?page=1`), Model.find() will return an object with `data` and `meta`. `data` is an array with the queried items, limited to the number you defined in the mixin options (see [Model Config](#model-config)). `meta` contains information about the requested page (see example below). You can specify the page as a query parameter (e.g. `?page=3`). ~~If no page is specified it defaults to 1~~ (deprecated in versions >= 2.0.0).

`/GET https://example.com/api/items?page=3`

```javascript
{
  "data": [
    {
      "title": "Item 1",
      "description": "Cool first item.",
      "id": "c5075168-abe0-41c6-8052-e07745eade48"
    },
    {
      "title": "Item 2",
      "description": "Cool second item.",
      "id": "0cad55df-c59d-4195-bb4f-7a252bd4bbe8",
    }

    ...

  ],
  "meta": {
    "totalItemCount": 95, // total number of items
    "totalPageCount": 10, // total number of all pages
    "itemsPerPage": 10,   // numberof items per page
    "currentPage": 3,     // the current page
    "nextPage": 4,        // the next page, only present if there is another page
    "previousPage": 2     // the previous page, only present if currentPage != 1
  }
}
```

The `methods` option's default value is `['find']`, some methods defined by relationship in Model.json file will also apply these mixin hooks.

For exmaple, if an user has many books, by defining the relationship in JSON file, Loopback will add an instance remote method `prototype.__get__books` for you. You can just add this method in the `methods` array, the Paginator will work without any other configuration.

```json
"methods" : ["find", "prototype.__get__books"]
```

If the method you are defining is a customized remote method like `search`, then you will have to provide the `totalItemCount` value to make the pagination works, here is an working exmaple:

```javascript
// Node.js util
const util = require('util');

// A full-text search remote method
MyModel.search = async (keyword, filter, options) => {
  const connector = MyModel.dataSource.connector;

  const sql = 'select * from MyModel where MATCH ' +
    '(name, description, comment) ' +
    'AGAINST (? IN NATURAL LANGUAGE MODE) order by name desc limit ?,?';

  const countSql = 'select count(*) as rowCount from MyModel where MATCH ' +
    '(name, description, comment) ' +
    'AGAINST (? IN NATURAL LANGUAGE MODE)';

  const executeAsync = util.promisify(connector.execute).bind(connector);
  const countResult = await executeAsync(countSql, [keyword]);

  if (countResult[0] &&
    countResult[0]['rowCount'] &&
    countResult[0]['rowCount'] > 0) {
    const rows = await executeAsync(sql, [keyword, filter['skip'], filter['limit']]);
    // Set totalItemCount value for Paginator
    filter.totalItemCount = countResult[0]['rowCount'];
    return rows;
  } else {
    filter.totalItemCount = 0;
    return [];
  }
};

MyModel.remoteMethod('search', {
  description: 'Find MyModel by keyword.',
  accepts: [
    {arg: 'keyword', type: 'string', required: true, http: {source: 'query'},
      description: 'Eg. "Steve"'},
    {arg: 'filter', type: 'object', description: 'Dummy filter for Paginator'}
  ],
  returns: {
    arg: 'data', type: 'object', root: true,
    description: 'An array of results.'
  },
  http: {verb: 'get'}
});
```

## ToDo

- [x] allow to override limit with an URL parameter

## License

[MIT](LICENSE)
