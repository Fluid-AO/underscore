
                       __
                      /\ \                                                         __
     __  __    ___    \_\ \     __   _ __   ____    ___    ___   _ __    __       /\_\    ____
    /\ \/\ \ /' _ `\  /'_  \  /'__`\/\  __\/ ,__\  / ___\ / __`\/\  __\/'__`\     \/\ \  /',__\
    \ \ \_\ \/\ \/\ \/\ \ \ \/\  __/\ \ \//\__, `\/\ \__//\ \ \ \ \ \//\  __/  __  \ \ \/\__, `\
     \ \____/\ \_\ \_\ \___,_\ \____\\ \_\\/\____/\ \____\ \____/\ \_\\ \____\/\_\ _\ \ \/\____/
      \/___/  \/_/\/_/\/__,_ /\/____/ \/_/ \/___/  \/____/\/___/  \/_/ \/____/\/_//\ \_\ \/___/
                                                                                  \ \____/
                                                                                   \/___/

[![Build Status](https://travis-ci.org/SqrTT/underscore.svg?branch=misterDW)](https://travis-ci.org/SqrTT/underscore)

#Improved Underscore version for Demandware

List of main changes:
* iterative methods works properly with demandware iterators, `_.each(basket.productLineItems, function (productLineItem) {...})` etc.
* iterative methods support second argument as string and can interpolate it as lambda expression (described bellow).
* added method `_.prop` - allows safety get deep property `_.prop(pdict, 'session.customer.profile.custom.isOurGuy')` return `isOurGuy` only if whole path exist or `undefined` in another case (will not throw error).
* added method `merge` - allows copy properties from one to another deeply by provided properties.
* methods which use `_.property` also works through `_.prop`, that means you can write `_.pluck(basket.productLineItems, 'product.custom.surprise')` and get array of `surprise's`.
* removed unsupported by DW async methods `delay`, `denounce`.
* changed delimiters for templates to `{{ | }}` (as `<% | %>` throw error in DW template engine).

## Method `prop(object, key, defaults)`
Some times is boring to write ton of condition just to get one deep property from object.
```javascript
if (pdict && pdict.session && pdict.session.customer &&
	pdict.session.customer.profile && pdict.session.customer.profile.custom.isOurGuy) {
	// .. do some magic
}
```
To solve this problem here is this method. Its pretty useful:
```javascript
if (_.prop(pdict, 'session.customer.profile.custom.isOurGuy') {
	// .. do some magic
}
```
`prop` also have third parameter if you set it than that value will be returned in case if some object is missing in path.

## Method `merge(destinationObject, sourceObject, mapObject)`
Copy properties from **sourceObject** to **destinationObject** by following the
mapping defined by **mapObject**

 - **sourceObject** is the object FROM which properties will be copied.
 - **destinationObject** is the object TO which properties will be copied.
 - **mapObject** is the object which defines how properties are copied from
**sourceObject** to **destinationObject**

example
------------

```javascript

var obj = {
  "sku" : "12345",
  "upc" : "99999912345X",
  "title" : "Test Item",
  "description" : "Description of test item",
  "length" : 5,
  "width" : 2,
  "height" : 8,
  "inventory" : {
    "onHandQty" : 12
  }
};

var map = {
  // path in dst : path in src
  "Envelope.Request.Item.SKU" : "sku",
  "Envelope.Request.Item.UPC" : "upc",
  "Envelope.Request.Item.ShortTitle" : "title",
  "Envelope.Request.Item.ShortDescription" : "description",
  "Envelope.Request.Item.Dimensions.Length" : "length",
  "Envelope.Request.Item.Dimensions.Width" : "width",
  "Envelope.Request.Item.Dimensions.Height" : "height",
  "Envelope.Request.Item.Inventory" : "inventory.onHandQty"
};

var result = _.merge({}, obj, map);

/*
{
  Envelope: {
    Request: {
      Item: {
        SKU: "12345",
        UPC: "99999912345X",
        ShortTitle: "Test Item",
        ShortDescription: "Description of test item",
        Dimensions: {
          Length: 5,
          Width: 2,
          Height: 8
        },
        Inventory: 12
      }
    }
  }
};
*/
```

## Lambda expression

Lambda is borrowed from [form.js](https://fromjs.codeplex.com/) project and is useful in case if short function should be feed to methods like `each`, `map`, `filter` etc.

It will be so tiring work to write every nested function every time. It can be evaded by using lambda expression.
Its format is almost same as C#'s.

Here's an example.

```javascript
function (arg1, arg2, arg3) {
    return arg1 * arg2 + arg3;
}
```

The function given above can be re-written as below using lambda expression.

```
(arg1, arg2, arg3) -> arg1 * arg2 + arg3
```

Parentheses can be omitted when it has only one argument.

```
arg1 -> arg1 * 3
```

Now let's apply it into real code.

```javascript
var numbers = [ 5, 4, 1, 3, 9, 8, 6, 7, 2, 0 ];

_(numbers)
    .filter(function (value) {
        return value < 5;
    })
    .each(function (value) {
        console.log(value);
    });
```

The example above can be re-written as below.

```javascript
var numbers = [ 5, 4, 1, 3, 9, 8, 6, 7, 2, 0 ];

_(numbers)
    .filter('value -> value < 5')
    .each('value -> console.log(value)');
```

### Omitting argument list

Lambda expression can be shorten more by omitting argument list.
But how can it be used without any argument specified?
There is provides several abbreviations which can be used in this case.

| Abbreviation | Meaning                          |
| ------------ | -------------------------------- |
| #n           | The _n_-th argument (zero based) |
| $            | The first argument (same as $0)  |
| $$           | The second argument (same as $1) |

For example,

```
(arg0, arg1, arg2, arg3) -> arg0 * arg1 + arg2 * arg3
```

the expression above can be shorten as below.

```
-> $0 * $1 + $2 * $3
```

or

```
-> $ * $1 + $2 * $3
```

Let's apply it into code.

```javascript
var numbers = [ 5, 4, 1, 3, 9, 8, 6, 7, 2, 0 ];

_(numbers)
    .filter(function (value) {
        return value < 5;
    })
    .each(function (value) {
        console.log(value);
    });
```

The sample above can be shorten as below.

```javascript
var numbers = [ 5, 4, 1, 3, 9, 8, 6, 7, 2, 0 ];
_(numbers).filter('-> $ < 5').each('->console.log($)');
```

As you will see, most predicator functions have similar arguments list (except comparers).
In most cases, the first argument means 'value', the second means 'key', and the last means 'external argument'.

-------------------------------------------------------------

Underscore.js is a utility-belt library for JavaScript that provides
support for the usual functional suspects (each, map, reduce, filter...)
without extending any core JavaScript objects.

For Docs, License, Tests, and pre-packed downloads, see:
http://underscorejs.org

Underscore is an open-sourced component of DocumentCloud:
https://github.com/documentcloud

Many thanks to our contributors:
https://github.com/jashkenas/underscore/contributors
