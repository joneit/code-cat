Catalog the execution context object

### Synopsis

```js
function(options)
```

### Parameters
0. `this` (the execution context) is "the module," the object to catalog
1. `options` is an optional options object (see below)

### Description

Creates a _catalog_ of all members of the execution context (`this`) and all _visible_ members of all the objects along it's prototype chain.

By _visible members_ we mean only those members not shadowed by a member of the same name in a descending object in the prototype chain.

The resultant "catalog" is a plain javascript object. For each member of the catalog:
* _key_ — The name of a member of the execution context.
* _value_ — A reference to the object that owns the visible member, which will either be the execution context object itself or some object along its prototype chain.

To keep things simple:
* The catalog excludes members that reference the catalog function itself (_e.g.,_ as a result of having been mixed in)
* The catalog excludes members of `Object.prototype`
* The resultant catalog object has no prototype

### Options

#### `options.own`
If truthy, the resultant catalog is restricted to the execution context object only (excluding the prototype chain).

#### `options.greylist`
Each [`greylist`](https://github.com/joneit/greylist) option below may be:
* a string; or
* a regular expression; or
* an object (_i.e.,_ its enumerable defined keys); or
* a (nested) array of a mix of any of the above; or
* an empty array; or
* `undefined` (which is a no-op).

#### `options.greylist.white`
A whitelist of permissible object member keys to catalog.
Only listed object members are cataloged.
If `undefined`, all members are cataloged.
If an empty array, all members are blocked.

#### `options.greylist.black`
A blacklist of impermissible object member keys.
Listed object members are blocked.
If `undefined` or an empty array, all members that passed the whitelist are included.

### Example

#### A. Default behavior
When no options are specified:

```js
function MyAPI() { this.b = 0; this.c = 3; }
MyAPI.prototype = { a:1, b:2, catalog:catalog };
var myAPI = new MyAPI;
myAPI.catalog(); // { a:MyAPI.prototype, b:myAPI, c:myAPI }
Object.getPrototypeOf(myAPI).catalog(); // { a:MyAPI.prototype, b:MyAPI.prototype }
```

#### B. The `own` behavior
Proceeding from [Example A](#a-default-behavior):

```js
var options = { own: true };
myAPI.catalog(options); // { b:myAPI, c:myAPI }
```

#### C. The `greylist` behavior
Proceeding from the [Example A](#a-default-behavior):

```js
var options = { greylist: { white: ['a', 'c'] } };
myAPI.catalog(options); // { a:MyAPI.prototype, c:myAPI }

options = { greylist: { black: ['a', 'c'] } };
myAPI.catalog(options); // { b:myAPI }

var options = { greylist: { white: ['a', 'c'], black: 'a' } };
myAPI.catalog(options); // { c:myAPI }
```

#### D. Lacking class

The `catalog` function does not care where the object (and it's prototype) came from. You will get the same results regardless of whether the object is a "class" instance (_i.e.,_ was [instanced from a constructor](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/new)) or was simply [created](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/create). Compare the following to [Example A](#a-default-behavior):

```js
var myAPI = Object.create({ a:1, b:2, catalog:require('object-catalog') }};
Object.assign(myAPI, { b:0, c:3 });
myAPI.catalog(); // { a:ancestor, b:myAPI, c:myAPI }
Object.getPrototypeOf(myAPI).catalog(); // { a:ancestor, b:ancestor }
```

#### E. Without mixing in

Notice that if the catalog function is mixed into the object, it is nonetheless excluded from the output. This exclusion is by reference and is regardless of the name of the key. You do not of course need to mix the function in if you're willing to use [`call`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Function/call). Compare the following to [Example D](#d-without-mixing-in):

```js
var myAPI = Object.create({ a:1, b:2 }};
Object.assign(myAPI, { b:0, c:3 });
var catalog = require('object-catalog');
catalog.call(myAPI); // { a:ancestor, b:myAPI, c:myAPI }
Object.getPrototypeOf(myAPI).catalog(); // { a:ancestor, b:ancestor }
```