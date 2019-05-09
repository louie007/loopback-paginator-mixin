# Pagination mixin for LoopBack

Paginator adds an easy to use pagination to any of your models. See the [example](#usage) below.

## Installation

```
$ npm i loopback-paginator --save
```

## Config

### Server Config

With [loopback-boot@v2.8.0](https://github.com/strongloop/loopback-boot/) [mixinSources](https://github.com/strongloop/loopback-boot/pull/131) have been implemented in a way which allows for loading this mixin without changes to the server.js file previously required. Just add `"../node_modules/loopback-paginator"` to the `mixins` property of your `server/model-config.json`.

```javascript
{
  "_meta": {
    "mixins": [
      "loopback/common/mixins",
      "../node_modules/loopback-paginator/lib",
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
      "firstPage": true   // paginate to page=1, by default
    }
  }
}
```

#### limit and maxLimit

* `limit` is the default limit to be used for this model
* `maxLimit` is the default maximum number of items per page for this model. As you can override the default limit using the [LoopBack limit filter](https://loopback.io/doc/en/lb3/Limit-filter.html) it might come in handy to set a maxLimit to prevent your API from being abused. The default is `100`.
* `noMaxLimit` set to `true` will deactivate the `maxLimit`. Be careful!
* `firstPage` set to `true` will paginate to page=1 (by default), set to `false` will return unpaginated results.

### Global Config

It is also possible to configure the mixin globally in your `config.json`. Just add `paginator` and use the same options as with the model above:

```javascript
{
  "paginator": {
    "limit": 20,        // items per page, default: 10
    "maxLimit": 300,    // max items per page, default: 100
    "noMaxLimit": true, // only use this, if you know what you are doing!
    "firstPage": true   // paginate to page=1, by default
  }
}
```

#### limit and maxLimit

* `limit` is the default limit to be used globally
* `maxLimit` is the default maximum number of items per page globally. As you can override the default limit using the [LoopBack limit filter](https://loopback.io/doc/en/lb3/Limit-filter.html) it might come in handy to set a maxLimit to prevent your API from being abused. The global default is `100`.
* `noMaxLimit` set to `true` will deactivate the `maxLimit`. Be careful!
* `firstPage` set to `true` will paginate by default to page=1 (by default), set to `false` will return unpaginated results.

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

## ToDo

- [x] allow to override limit with an URL parameter

## License

[MIT](LICENSE)

## Changelog

### v2.0.0
- Breaking change: node >=8.0.0 is required!
- Breaking change: omitting the page parameter no longer defaults to page=1, it now returns unpaginated results

### v1.0.0
- Allow to override limit with the LoopBack limit filter
- Add `maxLimit` option
- Add `noMaxLimit` option

### v0.9.6
- Add global config
