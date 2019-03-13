title: 05-ES6 Transpilation
date: 2018-04-27
---

So, our javascript is just your run-of-the-mill, runs in the browser, ES5, boring old javascript. But we're making
a shiny new, state-of-the-art javascript application, so we should really be pushing the latest and greatest ES6
syntax. Step up [Babel](https://babeljs.io).

`Babel` is a javascript transpiler. It will transpile (convert from one format to another), ES6 syntax javascript, to
ES5 syntax javascript that is runnable in the browser. For this, we will require 
[broccoli-babel-transpiler](https://github.com/babel/broccoli-babel-transpiler).
 
```sh
yarn add --dev @babel/core@^7.1.0 broccoli-babel-transpiler@^7.0.0 @babel/preset-env@^7.1.0
```

Now open `app/app.js` and set the contents to:

```js
// app/app.js
const message = 'Eat your greens';
function foo() {
    setTimeout(() => {
        console.log(message);
        console.log(this);
    });
}
new foo();
```

And `Brocfile.js`

```js
// Brocfile.js
const funnel = require('broccoli-funnel');
const merge = require('broccoli-merge-trees');
const compileSass = require('broccoli-sass-source-maps')(require('sass'));
const babel = require('broccoli-babel-transpiler');

const appRoot = 'app';

// Copy HTML file from app root to destination
const html = funnel(appRoot, {
  files: ["index.html"],
  annotation: "Index file",
});

// Copy JS file into assets
let js = funnel(appRoot, {
  files: ["app.js"],
  destDir: "/assets",
  annotation: "JS files",
});

// Transpile JS files to ES5
js = babel(js, {
  browserPolyfill: true,
  sourceMap: 'inline',
});

// Compile SCSS files
const css = compileSass(
  [appRoot],
  'styles/app.scss',
  'assets/app.css',
  {
    sourceMap: true,
    sourceMapContents: true,
    annotation: "Sass files"
  }
);

// Copy public files into destination
const public = funnel('public', {
  annotation: "Public files",
});

module.exports = merge([html, js, css, public], {annotation: "Final output"});
```

Make a `.babelrc` file in the root directory:
```json
{
  "presets": [
    [
      "@babel/env",
      {
        "targets": {
          "browsers": ["last 2 versions"]
        }
      }
    ]
  ]
}
```

So what's happening here? First off, we've declared `js` as a mutable variable (`let`) rather than an immutable
variable (`const`) so we can re-assign it. Next, we pass the `js` tree into the babel transpiler, this will
transpile the ES6 syntax to ES5. The `.bablerc` is the standard practice for configuring Babel, here we're targeting
the last 2 major versions of each browser for transpilation, this means that language features that are unsupported in
these browsers will have them transpiled into a format that *is* compatible.

If you `build` this now, and open `dist/assets/app.js`, you should see:

```js
// dist/assets/app.js
'use strict';

var message = 'Eat your greens';
function foo() {
    var _this = this;

    setTimeout(function () {
        console.log(message);
        console.log(_this);
    });
}
new foo();
//# sourceMappingURL=...
```

So, a few things have happened here:

* `const` has been changed to `var`
* The arrow function `() => {}` inside the setTimeout has been converted to a regular function.
* The use of `this` within the arrow function refers to the correct `this` context.
* You should also notice the sourcemap at the bottom, this has to be `inline` for this plugin to work

If you run this in the browser, you'll see the message and the correct `this` is logged to the console.

Now try adding a `debugger;` statement into the function and notice that the console stops at the breakpoint.
The presented source code should be the original ES6 version, *not* the transpiled ES5 version.

Completed Branch: [05-es6-transpilation](https://github.com/oligriffiths/broccolijs-tutorial/tree/05-es6-transpilation)

Next: [06-es6-modules](06-es6-modules.html)
