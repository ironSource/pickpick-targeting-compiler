# pickpick-targeting-compiler

**compiles targeting expressions to javascript code for pickpick engine**

-   uses [jsep](https://github.com/soney/jsep) to parse expressions and turn them into a javascript function that evaluates a conditiongiven a certain input.
-   uses the `vm` module to have isolated context
-   in addition to normal javascript operators, there are some special operators (see below for example):
    -   `in` is exactly `Array.includes`
    -   `match` regexp test
    -   `startsWith` and `endsWith` is exactly `String.startsWith` and `String.endsWith` respectively
-   exposes a bunch of useful `is*` functions from `util` module
-   open for extension via user defined envinronment accessible using `$`

## example

`npm i pickpick-targeting-compiler`

```js
const assert = require('assert')
const compile = require('pickpick-targeting-compiler')

// intentionally using var here 

// all normal js operators apply
var { isMatch, features } = compile('_.geo === "US"')
assert(isMatch({ geo: 'US' }))

var { isMatch, features } = compile('_.number >= 0')
assert(isMatch({ number: 1 }))
assert(isMatch({ number: 2 }))
assert(isMatch({ number: 3 }))

// can create compound logical statements
var { isMatch, features } = compile('_.geo === "US" && _.number > 5')
assert(isMatch({ geo: 'US', number: 8 }))

// in operator - same as Array.includes(value)
var { isMatch, features } = compile('_.geo in ["US", "MX"]')
assert(isMatch({ geo: 'US' }))

// operators apply to context data as well
var { isMatch, features } = compile('_.geo in _.page')
assert(isMatch({ geo: 'US', page: ["US", "MX"] }))

// startsWith operator - same as String.startsWith(string)
var { isMatch, features } = compile('_.geo startsWith "x"')
assert(isMatch({ geo: 'xyz' }))

// endsWith operator - same as String.endsWith(string)
var { isMatch, features } = compile('_.geo endsWith "z"')
assert(isMatch({ geo: 'xyz' }))

// match operator for literal regular expressions, same as /regex/g.test('value')
var { isMatch, features } = compile('_.geo match "[0-9]"')
assert(isMatch({ geo: 0 }))

// deeplyEquals operator performs a deep equal comparison (uses [lodash.isEqual](https://lodash.com/docs/4.17.10#isEqual))
var { isMatch, features } = compile('_.geo deeplyEquals [1, 2, 3]')
assert(isMatch({ geo: [1, 2, 3] }))

// exposes a bunch of isSomething from util:
// isNullOrUndefined,
// isNull,
// isUndefined,
// isString,
// isNumber,
// isArray,
// isObject,
// isBoolean,
// isDate,
// isNull,
// isPrimitive
var { isMatch, features } = compile('isNumber(_.geo)')
assert(isMatch({ geo: 0 }))

// provide user defined functions and data
const userEnvironment = { x: [1, 2, 3], isOk: (arg) => arg.startsWith('x') }
var { isMatch, features } = compile('_.geo in $.x && $.isOk(_.page)', { userEnvironment })
assert(isMatch({ geo: 1, page: 'x.html' }))
```

## api

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [compile](#compile)

### compile

[index.js:56-156](https://github.com/ironSource/pickpick-targeting-compiler/blob/5059d0b7773162460d5ae0d9b10f72d883e6aad9/index.js#L56-L156 "Source code on GitHub")

Compile an expression into a resuable javascript function

**Parameters**

-   `expression` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** any expression that can be parsed by [jsep](https://github.com/soney/jsep)
-   `options` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)**  (optional, default `{}`)
    -   `options.matcherProperty` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** return the match function using a different property name in the compound return value, e.g: (optional, default `'isMatch'`)
    -   `options.userEnvironment` **[Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)** allows the user to inject additional functionality that will be exposed to the expression. e.g: (optional, default `{}`)
    -   `options.userEnvNamespace` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** change the name used to access user environment in an expression (optional, default `$`)
    -   `options.inputNamespace` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** change the name used to access input in the expression (optional, default `_`)

**Examples**

```javascript
// options.matcherProperty
 
   // normally this would be isMatch instead of 'foo'
   let { foo } = compile('_.geo === "1"', { matcherProperty: 'foo' })
```

```javascript
// options.userEnvironment
   
   let userEnvironment = {
 	    geos: ['MX', 'US', 'IL'],
  	    format: value => value.toUpperCase()
   }
   
   let { isMatch } = compile('$.format(_.geo) in $.geos', { userEnvironment })
```

Returns **[Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)** a function that accepts input and returns a boolean value

## license

[MIT](http://opensource.org/licenses/MIT) © ironSource LTD
