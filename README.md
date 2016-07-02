# base-data [![NPM version](https://img.shields.io/npm/v/base-data.svg?style=flat)](https://www.npmjs.com/package/base-data) [![NPM downloads](https://img.shields.io/npm/dm/base-data.svg?style=flat)](https://npmjs.org/package/base-data) [![Build Status](https://img.shields.io/travis/node-base/base-data.svg?style=flat)](https://travis-ci.org/node-base/base-data)

adds a `data` method to base-methods.

## TOC

- [Install](#install)
- [Usage](#usage)
- [API](#api)
- [Glob patterns](#glob-patterns)
- [Namespacing](#namespacing)
- [History](#history)
- [Related projects](#related-projects)
- [Contributing](#contributing)
- [Building docs](#building-docs)
- [Running tests](#running-tests)
- [Author](#author)
- [License](#license)

_(TOC generated by [verb](https://github.com/verbose/verb) using [markdown-toc](https://github.com/jonschlinkert/markdown-toc))_

## Install

Install with [npm](https://www.npmjs.com/):

```sh
$ npm install --save base-data
```

## Usage

Adds a `data` method to [base](https://github.com/node-base/base) that can be used for setting, getting and loading data onto a specified object in your application.

```js
var Base = require('base');
var data = require('base-data');

// instantiate `Base`
var base = new Base();
// add `data` as a plugin
base.use(data());
```

**Examples**

Add data:

```js
app.data('a', 'b');
app.data({c: 'd'});
app.data('e', ['f']);
console.log(app.cache.data);
//=> {a: 'b', c: 'd', e: ['f']}
```

**cache.data**

By default, all data is loaded onto `app.cache.data`. This can be customized by passing the property to use when the plugin is initialized.

For example, the following set `app.foo` as object for storing data:

```js
app.use(data('foo'));
app.data('a', 'b');
console.log(app.foo);
//=> {a: 'b'}
```

## API

### [.dataLoader](index.js#L66)

Register a data loader for loading data onto `app.cache.data`.

**Params**

* `ext` **{String}**: The file extension for to match to the loader
* `fn` **{Function}**: The loader function.

**Example**

```js
var yaml = require('js-yaml');

app.dataLoader('yml', function(str, fp) {
  return yaml.safeLoad(str);
});

app.data('foo.yml');
//=> loads and parses `foo.yml` as yaml
```

### [.data](index.js#L101)

Load data onto `app.cache.data`

**Params**

* `key` **{String|Object}**: Key of the value to set, or object to extend.
* `val` **{any}**
* `returns` **{Object}**: Returns the instance of `Template` for chaining

**Example**

```js
console.log(app.cache.data);
//=> {};

app.data('a', 'b');
app.data({c: 'd'});
console.log(app.cache.data);
//=> {a: 'b', c: 'd'}

// set an array
app.data('e', ['f']);

// overwrite the array
app.data('e', ['g']);

// update the array
app.data('e', ['h'], true);
console.log(app.cache.data.e);
//=> ['g', 'h']
```

### [.data.extend](index.js#L252)

Shallow extend an object onto `app.cache.data`.

**Params**

* `key` **{String|Object}**: Property name or object to extend onto `app.cache.data`. Dot-notation may be used for extending nested properties.
* `value` **{Object}**: The object to extend onto `app.cache.data`
* `returns` **{Object}**: returns the instance for chaining

**Example**

```js
app.data({a: {b: {c: 'd'}}});
app.data.extend('a.b', {x: 'y'});
console.log(app.get('a.b'));
//=> {c: 'd', x: 'y'}
```

### [.data.merge](index.js#L285)

Deeply merge an object onto `app.cache.data`.

**Params**

* `key` **{String|Object}**: Property name or object to merge onto `app.cache.data`. Dot-notation may be used for merging nested properties.
* `value` **{Object}**: The object to merge onto `app.cache.data`
* `returns` **{Object}**: returns the instance for chaining

**Example**

```js
app.data({a: {b: {c: {d: {e: 'f'}}}}});
app.data.merge('a.b', {c: {d: {g: 'h'}}});
console.log(app.get('a.b'));
//=> {c: {d: {e: 'f', g: 'h'}}}
```

### [.data.union](index.js#L317)

Union the given value onto a new or existing array value on `app.cache.data`.

**Params**

* `key` **{String}**: Property name. Dot-notation may be used for nested properties.
* `array` **{Object}**: The array to add or union on `app.cache.data`
* `returns` **{Object}**: returns the instance for chaining

**Example**

```js
app.data({a: {b: ['c', 'd']}});
app.data.union('a.b', ['e', 'f']}});
console.log(app.get('a.b'));
//=> ['c', 'd', 'e', 'f']
```

### [.data.set](index.js#L337)

Set the given value onto `app.cache.data`.

**Params**

* `key` **{String|Object}**: Property name or object to merge onto `app.cache.data`. Dot-notation may be used for nested properties.
* `val` **{any}**: The value to set on `app.cache.data`
* `returns` **{Object}**: returns the instance for chaining

**Example**

```js
app.data.set('a.b', ['c', 'd']}});
console.log(app.get('a'));
//=> {b: ['c', 'd']}
```

### [.data.get](index.js#L360)

Get the value of `key` from `app.cache.data`. Dot-notation may be used for getting nested properties.

**Params**

* `key` **{String}**: The name of the property to get.
* `returns` **{any}**: Returns the value of `key`

**Example**

```js
app.data({a: {b: {c: 'd'}}});
console.log(app.get('a.b'));
//=> {c: 'd'}
```

## Glob patterns

Glob patterns may be passed as a string or array. All of these work:

```js
app.data('foo.json');
app.data('*.json');
app.data(['*.json']);
// pass options to node-glob
app.data(['*.json'], {dot: true});
```

## Namespacing

Namespacing allows you to load data onto a specific key, optionally using part of the file path as the key.

**Example**

Given that `foo.json` contains `{a: 'b'}`:

```js
app.data('foo.json');
console.log(app.cache.data);
//=> {a: 'b'}

app.data('foo.json', {namespace: true});
console.log(app.cache.data);
//=> {foo: {a: 'b'}}

app.data('foo.json', {
  namespace: function(fp) {
    return path.basename(fp);
  }
});
console.log(app.cache.data);
//=> {'foo.json': {a: 'b'}}
```

## History

**v0.3.6**

* adds a basic loader that only calls the `JSON.parse` method, if no other loaders are defined
* calls `.isRegistered` from [base](https://github.com/node-base/base) to ensure the plugin is only loaded once on an instance

## Related projects

You might also be interested in these projects:

* [base](https://www.npmjs.com/package/base): base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting… [more](https://github.com/node-base/base) | [homepage](https://github.com/node-base/base "base is the foundation for creating modular, unit testable and highly pluggable node.js applications, starting with a handful of common methods, like `set`, `get`, `del` and `use`.")
* [base-cli](https://www.npmjs.com/package/base-cli): Plugin for base-methods that maps built-in methods to CLI args (also supports methods from a… [more](https://github.com/node-base/base-cli) | [homepage](https://github.com/node-base/base-cli "Plugin for base-methods that maps built-in methods to CLI args (also supports methods from a few plugins, like 'base-store', 'base-options' and 'base-data'.")
* [base-config](https://www.npmjs.com/package/base-config): base-methods plugin that adds a `config` method for mapping declarative configuration values to other 'base… [more](https://github.com/node-base/base-config) | [homepage](https://github.com/node-base/base-config "base-methods plugin that adds a `config` method for mapping declarative configuration values to other 'base' methods or custom functions.")
* [base-option](https://www.npmjs.com/package/base-option): Adds a few options methods to base, like `option`, `enable` and `disable`. See the readme… [more](https://github.com/node-base/base-option) | [homepage](https://github.com/node-base/base-option "Adds a few options methods to base, like `option`, `enable` and `disable`. See the readme for the full API.")
* [base-pipeline](https://www.npmjs.com/package/base-pipeline): base-methods plugin that adds pipeline and plugin methods for dynamically composing streaming plugin pipelines. | [homepage](https://github.com/node-base/base-pipeline "base-methods plugin that adds pipeline and plugin methods for dynamically composing streaming plugin pipelines.")
* [base-plugins](https://www.npmjs.com/package/base-plugins): Upgrade's plugin support in base applications to allow plugins to be called any time after… [more](https://github.com/node-base/base-plugins) | [homepage](https://github.com/node-base/base-plugins "Upgrade's plugin support in base applications to allow plugins to be called any time after init.")
* [base-store](https://www.npmjs.com/package/base-store): Plugin for getting and persisting config values with your base-methods application. Adds a 'store' object… [more](https://github.com/node-base/base-store) | [homepage](https://github.com/node-base/base-store "Plugin for getting and persisting config values with your base-methods application. Adds a 'store' object that exposes all of the methods from the data-store library. Also now supports sub-stores!")

## Contributing

This document was generated by [verb-readme-generator](https://github.com/verbose/verb-readme-generator) (a [verb](https://github.com/verbose/verb) generator), please don't edit directly. Any changes to the readme must be made in [.verb.md](.verb.md). See [Building Docs](#building-docs).

Pull requests and stars are always welcome. For bugs and feature requests, [please create an issue](../../issues/new).

Or visit the [verb-readme-generator](https://github.com/verbose/verb-readme-generator) project to submit bug reports or pull requests for the readme layout template.

## Building docs

_(This document was generated by [verb-readme-generator](https://github.com/verbose/verb-readme-generator) (a [verb](https://github.com/verbose/verb) generator), please don't edit the readme directly. Any changes to the readme must be made in [.verb.md](.verb.md).)_

Generate readme and API documentation with [verb](https://github.com/verbose/verb):

```sh
$ npm install -g verb verb-readme-generator && verb
```

## Running tests

Install dev dependencies:

```sh
$ npm install -d && npm test
```

## Author

**Jon Schlinkert**

* [github/jonschlinkert](https://github.com/jonschlinkert)
* [twitter/jonschlinkert](http://twitter.com/jonschlinkert)

## License

Copyright © 2016, [Jon Schlinkert](https://github.com/jonschlinkert).
Released under the [MIT license](https://github.com/node-base/base-data/blob/master/LICENSE).

***

_This file was generated by [verb](https://github.com/verbose/verb), v0.9.0, on July 02, 2016._