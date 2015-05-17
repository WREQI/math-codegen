# math-codegen 

[![NPM][npm-image]][npm-url]

[![Build Status][travis-image]][travis-url] [![Coverage Status][coveralls-image]][coveralls-url]  [![Dependency Status][david-image]][david-url]

> Generates JavaScript code from mathematical expressions

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Description](#description)
  - [Lifecycle](#lifecycle)
    - [Parse](#parse)
    - [Compile](#compile)
    - [Eval](#eval)
  - [Examples](#examples)
  - [Differences with math.js' expression parser](#differences-with-mathjs-expression-parser)
- [Install](#install)
- [Usage](#usage)
- [API](#api)
  - [`var instance = new CodeGenerator([options])`](#var-instance--new-codegeneratoroptions)
  - [`instance.parse(code)`](#instanceparsecode)
  - [`instance.compile(namespace)`](#instancecompilenamespace)
- [Inspiration projects](#inspiration-projects)
- [License](#license)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Description

A flexible interpreter for mathematical expressions which allows the programmer to change the usual semantic of an
operator bringing the operator overloading polymorphism to JavaScript (emulated with function calls),
in addition an expression can be evaluated under any adapted namespace providing expression portability between numeric libraries 

### Lifecycle

- `parse` a mathematical expression is parsed with [Esprima](http://esprima.org/)
- `compile` the parsed string is compiled against a namespace producing executable JavaScript code
- `eval` the executable JavaScript code is evaluated against a context

#### Parse

For example let's consider the following expression with the variables `a`, `b` and `c` which are user defined:

```javascript
'1 + 2 * x'
```

which can be emulated with function calls instead of operators (the parser identifies the operators as 
[Binary Expressions](https://github.com/estree/estree/blob/master/spec.md#binaryexpression))

```javascript
'add(1, multiply(2, x))'
```

now we can introduce the namespace `ns` where `add` and `multiply` come from

```javascript
'ns.add(1, ns.multiply(2, x))'
```

the variables (which for the parser are [Identifiers](https://github.com/estree/estree/blob/master/spec.md#identifier))
come from a context called `scope` but they might also be constant values defined in the namespace:

```javascript
'ns.add(1, ns.multiply(2, (scope["x"] || ns["x"]) ))'
```

the constant values might have different meanings for different namespaces therefore a `factory` is needed
on the namespace to transform these values into values the namespace can operate with

```javascript
'ns.add(ns.factory(1), ns.multiply(ns.factory(2), (scope["x"] || ns["x"]) ))'
```

#### Compile

Now that we have a parsed expression we have to compile it against a namespace to produce
executable JavaScript code
 
```javascript
parse('1 + 2 * x').compile(namespace)

// returns something like this
(function (ns) {
  return {
    eval: function (scope) {
      // scope processing
      // ...
      // the string parsed above goes here
    }
  }
})(namespace)
```

#### Eval

The object returned above can be evaluated within a context

```javascript
parse('1 + 2 * x').compile(namespace).eval(scope)
```

### Examples

TODO:

### Differences with math.js' expression parser

Math.js expression parser API is quite similar having the same lifecycle however there are some
important facts I've found:

- `math.js` has a custom expression parser, `math-codegen` uses Esprima which support the ES5 grammar
[(ESTree AST nodes)](https://github.com/estree/estree/blob/master/spec.md)
- `math.js` v1.x arrays can represent matrices with `ns.matrix` or as a raw array, `math-codegen` doesn't
make any assumptions of arrays and treats them just like any other constant allowing the namespace to 
decide what to do with an array in its factory

## Install

```sh
$ npm install --save math-codegen
```

## Usage

```js
var CodeGenerator = require('math-codegen');
new CodeGenerator([options]).parse(code).compile(namespace).eval(scope)
```

## API

### `var instance = new CodeGenerator([options])`
**properties**
* `statements` {Array} An array of statements parsed from an expression
* `interpreter` {Interpreter} Instance of the Interpreter class
* `defs` {Object} An object with additional definitions available during the compilation
that exist during the instance lifespan
 
**params**
* `options` {Object} Options available for the interpreter
  * `[options.factory="ns.factory"]` {string} factory method under the namespace 
  * `[options.raw=false]` {boolean} True to interpret BinaryExpression, UnaryExpression and ArrayExpression
   in a raw way without wrapping the operators with the factory method
  * `[options.rawArrayExpressionElements=true]` {boolean} true to interpret the array elements in a raw way
  * `[options.rawCallExpressionElements=false]` {boolean} true to interpret call expression
   elements in a raw way

### `instance.parse(code)`

**chainable**
**params**
* `code` {string} string to be parsed
 
Parses a program using [Esprima](http://esprima.org/), each Expression Statement is saved in
`instance.statements`

Node types implemented:

- Nodes:
  - [ExpressionStatement](https://github.com/estree/estree/blob/master/spec.md#expressionstatement)
- Expressions:
  - [ArrayExpression](https://github.com/estree/estree/blob/master/spec.md#arrayexpression)
  - [UnaryExpression](https://github.com/estree/estree/blob/master/spec.md#unaryexpression)
  - [BinaryExpression](https://github.com/estree/estree/blob/master/spec.md#binaryexpression)
  - [AssignmentExpression](https://github.com/estree/estree/blob/master/spec.md#assignmentexpression)
  - [ConditionalExpression](https://github.com/estree/estree/blob/master/spec.md#conditionalexpression)
  - [CallExpression](https://github.com/estree/estree/blob/master/spec.md#callexpression)
  
### `instance.compile(namespace)`
  
**chainable**
**params**
* `namespace` {Object}

Compiles the code making `namespace`'s properties available during evaluation
 
**returns** {Object}
* `return.eval` {Function} Function to be evaluated under a context
 **params**
  * `scope` {Object}

## Inspiration projects

- [math.js expression parser](http://mathjs.org/docs/expressions/index.html)

## License

2015 MIT © [Mauricio Poppe]()

[npm-image]: https://nodei.co/npm/math-codegen.png?downloads=true
[npm-url]: https://npmjs.org/package/math-codegen
[travis-image]: https://travis-ci.org/maurizzzio/math-codegen.svg?branch=master
[travis-url]: https://travis-ci.org/maurizzzio/math-codegen
[coveralls-image]: https://coveralls.io/repos/maurizzzio/math-codegen/badge.svg?branch=master
[coveralls-url]: https://coveralls.io/r/maurizzzio/math-codegen?branch=master
[david-image]: https://david-dm.org/maurizzzio/math-codegen.svg
[david-url]: https://david-dm.org/maurizzzio/math-codegen
