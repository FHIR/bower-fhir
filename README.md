fhir.js
=======

[![Build Status](https://travis-ci.org/FHIR/fhir.js.svg)](https://travis-ci.org/FHIR/fhir.js)

[![Gitter chat](https://badges.gitter.im/FHIR/fhir.js.png)](https://gitter.im/FHIR/fhir.js)

JavaScript client for FHIR

## Goals:

 - Support FHIR CRUD operations
 - Friendly and expressive query syntax
 - Support for adapters that provide idiomatic interfaces in angular, jQuery, extjs, etc
 - Support for access control (HTTP basic, OAuth2)
 - ...

## Development

`Node.js` is required for build.

We recommend installling Node.js using [nvm](https://github.com/creationix/nvm/blob/master/README.markdown)

Build & test:

```
git clone https://github.com/FHIR/fhir.js
cd fhir.js
npm install

# buld fhir.js
npm run-script build

# run all tests once
npm run-script spec

# watch tests while development
npm run-script spec-watch

# run integration tests
npm run-script integrate
```

## API


### Create instance of the FHIR client

To communicate with concrete FHIR server, you can
create instance of the FHIR client, passing a
configuration object & adapter object.  Adapters are
implemented for concrete frameworks/libs (see below).

```
var cfg = {
  // FHIR server base url
  baseUrl: 'http://myfhirserver.com',
  auth: {
     bearer: 'token',
     // OR for basic auth
     user: 'user',
     pass: 'secret'
  }
}

myClient = fhir(cfg, adapter)
```

### Adapter implementation

Currently each adapter must implement an
`http(requestObj)` function:

Structure of requestObj:

* `method` - http method (GET|POST|PUT|DELETE)
* `url` - url for request
* `headers` - object with headers (i.e. {'Category': 'term; scheme="sch"; label="lbl"'}
* `success` - success callback, which should be called with (data, status, headersFn, config)
  * data - parsed body of responce
  * status - responce HTTP status
  * headerFn - function to get header, i.e. headerFn('Content')
  * config - initial requestObj passed to http
* `error` - error callback, which should be called with (data, status, headerFn, config)


Here are implementations for:

* [AngularJS adapter](https://github.com/FHIR/fhir.js/blob/master/src/adapters/angularjs.coffee)
* [jQuery adapter](https://github.com/FHIR/fhir.js/blob/master/src/adapters/jquery.coffee)
* [Node adapter](https://github.com/FHIR/fhir.js/blob/master/src/adapters/node.coffee)
* [YUI adapter](https://github.com/FHIR/fhir.js/blob/master/src/adapters/yui.coffee)

### Conformance & Profiles

### Resource's CRUD

To create a FHIR resource, call
`myClient.create(entry, callback, errback)`, passing
an object that contains the following propperties:

* `content` (required) - resource in FHIR json
* `tags` (opttional) - list of categories (see below)

In case of success,the  callback function will be
invoked with an object that contains the following
attributes:

* `id` - url of created resource
* `content` - resource json
* `category` - list of tags

```javascript

var entry = {
  category: [{term: 'TAG term', schema: 'TAG schema', label: 'TAG label'}, ...]
  content: {
    resourceType: 'Patient',
    //...
  }
}

myClient.create(entry,
 function(entry){
    console.log(entry.id)
 },
 function(error){
   console.error(error)
 }
)

```

### Tags Operations

### Search

`fhir.search('Patient', queryObject, callback, errback)` function is used
for [FHIR resource's search](http://www.hl7.org/implement/standards/fhir/search.html).

If success callback will be called with resulting [bundle](http://www.hl7.org/implement/standards/fhir/json.html#bundle).

For queryObject syntax `fhir.js` adopts
mongodb-like query syntax ([see](http://docs.mongodb.org/manual/tutorial/query-documents/)):

```javascript
{name: 'maud'}
//=> name=maud

{name: {$exact: 'maud'}}
//=> name:exact=maud

{name: {$or: ['maud','dave']}}
//=> name=maud,dave

{name: {$and: ['maud',{$exact: 'dave'}]}}
//=> name=maud&name:exact=Dave

{birthDate: {$gt: '1970', $lte: '1980'}}
//=> birthDate=>1970&birthDate=<=1980

{subject: {$type: 'Patient', name: 'maud', birthDate: {$gt: '1970'}}}
//=> subject:Patient.name=maud&subject:Patient.birthDate=>1970

{'subject.name': {$exact: 'maud'}}
//=> subject.name:exact=maud

```

For more information see [tests](https://github.com/FHIR/fhir.js/blob/master/test/querySpec.coffee)

## AngularJS adapter: `ng-fhir`

AngularJS adapter after `npm run-script build` can be found at `dist/ngFhir.js`


Usage:

```coffeescript
angular.module('app', ['ng-fhir'])
  # configure base url
  .config ($fhirProvider)->
     $fhirProvider.baseUrl = 'http://try-fhirplace.hospital-systems.com'
  .controller 'mainCtrl', ($scope, $fhir)->
     $fhir.search('Patient', {name: {$exact: 'Maud'}})
       .error (error)->
         $scope.error = error
       .success (bundle)->
         $scope.patients = bundle.entry
```

## jQuery adapter: `jqFhir`

jQuery build can be found at `dist/jqFhir.js`

[Example app](http://embed.plnkr.co/e4BKr0M07q4FVVQeP6f4/)


Usage:

```html
<script src="./jquery-???.min.js"> </script>
<script src="./jqFhir.js"> </script>
```


```javascript
// create fhir instance
var fhir = jqFhir({
    baseUrl: 'https://ci-api.fhir.me',
    auth: {user: 'client', pass: 'secret'}
})

fhir.search('Patient', {name: 'maud'})
.then(function(bundle){
  console.log('Search patients', bundle)
})
```

## Node.js adapter: `npm install fhir.ns`

Via NPM you can `npm install fhir.js`. (If you want to work on the source code,
you can compile coffee to js via `npm install`, and use `./lib/adapters/node`
as an entrypoint.)

```
var mkFhir = require('fhir.js');

var client = mkFhir({
  baseUrl: 'http://try-fhirplace.hospital-systems.com'
});

client.search( 'Patient', { 'birthdate': '1974' }, function(err, bundle) {
  var count = (bundle.entry && bundle.entry.length) || 0;
  console.log("# Patients born in 1974: ", count);
});

```

## YUI adapter: `yuiFhir`

YUI build can be found at `dist/yuiFhir.js`

NOTE: The current implementation creates a YUI sandbox per request which is expensive.

Usage:

```html
<script src="./yui-???.min.js"> </script>
<script src="./yuiFhir.js"> </script>
```

```javascript
// create fhir instance
var fhir = jqFhir({
    baseUrl: 'https://ci-api.fhir.me',
    auth: {user: 'client', pass: 'secret'}
})

fhir.search(type: 'Patient', query: {name: 'maud'}, success: function(bundle) {}, error: function() {})
```
## TODO

* npm package
* bower package

## Contribute

Join us by [github issues](https://github.com/FHIR/fhir.js/issues) or pull-requests
