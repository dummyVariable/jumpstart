# ES6 module system

The javascript module system is essentially in helping us build javascript programs that are small and maintainable. Variables defined within a module \(i.e. a `.js` file\) are scoped within the module, rather than the global scope.

## ES6 Module vs CommonJS Module

ES6 uses a different syntax for exporting and importing modules, as compared to the CommonJS system \(Node uses two core modules for managing module dependencies: `module` and `require`\)

|  | CommonJS | ES6 Module |
| :--- | :--- | :--- |
| exporting modules \(in `./someFile.js`\) | `module.exports = myFunction` | `export default myFunction` |
| importing modules \(in `./anotherFile.js`\) | `var myFunction = require('./someFile')` | `import myFunction from "./someFile";` |

## ES6 Module Syntax

**Named exports** vs. **default exports**

* default export
  * the name of the variable being imported/exported can be different
  * Each module can only have **one** default export

```javascript
// file1.js
const someVar = 'abc'

export default someVar


// file2.js
import iCanNameThisAnything from './file1'
console.log(iCanNameThisAnything) // prints 'abc'
```

* named export
  * the name of the variable being imported/exported must be the same
  * Each module can have any number of named exports
  * when importing the named export, we **must** have curly braces

```javascript
// file1.js
export const someVar = 'abc'

// file2.js
import { someVar } from './file1' // note the addition of curly braces
console.log(someVar)
```

* you can have both named and default exports in a single file. Just make sure you're following their respective conventions when importing and exporting them. Example:

```javascript
// file1.js
export default const someVar1 = 'abc'
export const someVar2 = 'def'
export const someVar3 = 'ghi'

// file2.js
import someVar1, {someVar2, someVar3} from './file1'
// ...
```

ES6 offers flexible ways of [exporting](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) and [importing](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) modules. Read more about them in the links/docs

## ES6 Module Support in Browsers Environment

Traditionally browsers don't support JavaScript modules at all. Although you write your JavaScript codes in modules in your development environment, you typically bundle all the javascript code into one file like `bundle.js` when you serve them in production environment. \(The bundling task is usually done using some tools called `bundlers` like Webpack\).

After browsers start supporting ES6 modules, it's possible to add a new attribute `type=module` to your `<script>` tag, to denote the script to be loaded is an ES6 module. More examples can be found in [this article](https://www.contentful.com/blog/2017/04/04/es6-modules-support-lands-in-browsers-is-it-time-to-rethink-bundling/)

Some browsers don't support ES6 module yet. You can find the latest status [here](https://caniuse.com/#feat=es6-module)

## ES6 Module Support in Node.JS Environment

Node.JS is built on top of CommonJS since day one and CommonJS is still the default module system used by Node.JS. Most of the modules written for Node.JS environment still use the CommonJS `module.exports` and `require` syntax instead of ES6 modules.

The support on ES6 modules is currently [an experimental feature](https://nodejs.org/api/esm.html). However, in order to use ES6 modules in your codes, you have to change your file extension to `.mjs` so that Node is aware the module is written using ES6 modules. Some descriptions of the current status can be found [here](http://2ality.com/2019/04/nodejs-esm-impl.html)

To run JavaScript applications on NodeJS platform, you have three choices:

1. Use the CommonJS syntax \(`const a = require('a')`\), or
2. Use ES6 module syntax in your JavaScript code and use [babel plugin](https://babeljs.io/docs/en/babel-plugin-transform-es2015-modules-commonjs) to transpile the codes into CommonJS module syntax and run it on NodeJS platform.
3. Use `babel-node` command to run your codes written in ES6 module system

For the last choice using 'babel-node', you need to do the following:

* `npm install babel-cli babel-preset-es2015`
* replace the `start` script in `package.json` to use: `babel-node --presets es2015 app.js` \(instead of `node app.js`\)

See example: [https://github.com/thoughtworks-jumpstart/basic-es6-template/](https://github.com/thoughtworks-jumpstart/basic-es6-template/)

However, the documentation for [babel-node](https://babeljs.io/docs/en/babel-node) says it's "not meant for production use".

For now, to keep it simple, we should still stay with the CommonJS module system when we develop applications running on NodeJS.

## Resources

* [JS Modules](http://jsmodules.io/)
* [ES6 Modules: The Syntax](http://2ality.com/2014/09/es6-modules-final.html)
* [ES modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)
* [ES6 module system](https://ponyfoo.com/articles/es6-modules-in-depth#the-es6-module-system)
* [ES6 `import` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import)
* [Bridging CommonJS and ES6 module system](https://medium.com/web-on-the-edge/es-modules-in-node-today-32cff914e4b)
* [Using ES6 modules on the web](https://developers.google.com/web/fundamentals/primers/modules)

