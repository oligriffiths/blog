title: 12-Minify
date: 2018-04-27
---

Minifying, or Uglifying as it's also sometimes called, is the process of turning normal JavaScript or CSS
into a compressed version, with much shorter variable and function names (for JS), and unnecessary whitespace removed
to save on bytes shipped to the browser. Fewer bytes means less time transferring files and less time for the browser
to parse the file.

```sh
yarn add --dev rollup-plugin-uglify@^3.0.0 broccoli-clean-css@^2.0.1
```

This will install the Rollup Uglify plugin. It is also possible to use the `broccoli-uglify-sourcemap` Broccoli plugin,
as Rollup seems to work just fine with it. However, it makes more sense to use the supported Rollup plugin
rather than the Broccoli one. We're also installing the `broccoli-clean-css` plugin that will compress our CSS.

Now update your `Brocfile.js` to:

```js
// Brocfile.js
const funnel = require('broccoli-funnel');
const merge = require('broccoli-merge-trees');
const compileSass = require('broccoli-sass-source-maps')(require('sass'));
const Rollup = require('broccoli-rollup');
const babel = require('rollup-plugin-babel');
const nodeResolve = require('rollup-plugin-node-resolve');
const commonjs = require('rollup-plugin-commonjs');
const LiveReload = require('broccoli-livereload');
const log = require('broccoli-stew').log;
const esLint = require('broccoli-lint-eslint');
const sassLint = require('broccoli-sass-lint');
const uglify = require('rollup-plugin-uglify');
const CleanCss = require('broccoli-clean-css');

const env = require('broccoli-env').getEnv() || 'development';
const isProduction = env === 'production';

// Status
console.log('Environment: ' + env);

const appRoot = "app";

// Copy HTML file from app root to destination
const html = funnel(appRoot, {
  files: ["index.html"],
  annotation: "Index file",
});

// Lint js files
let js = esLint(appRoot, {
  persist: true
});

// Compile JS through rollup
const rollupPlugins = [
  nodeResolve({
    jsnext: true,
    browser: true,
  }),
  commonjs({
    include: 'node_modules/**',
  }),
  babel({
    exclude: 'node_modules/**',
  }),
];

// Uglify the output for production
if (isProduction) {
  rollupPlugins.push(uglify());
}

js = new Rollup(js, {
  inputFiles: ['**/*.js'],
  rollup: {
    input: 'app.js',
    output: {
      file: 'assets/app.js',
      format: 'es',
      sourcemap: !isProduction,
    },
    plugins: rollupPlugins,
  }
});

// Lint css files
let css = sassLint(appRoot + '/styles', {
  disableTestGenerator: true,
});

// Copy CSS file into assets
css = compileSass(
  [css],
  'app.scss',
  'assets/app.css',
  {
    sourceMap: !isProduction,
    sourceMapContents: true,
    annotation: "Sass files"
  }
);

// Compress our CSS
if (isProduction) {
  css = new CleanCss(css);
}

// Copy public files into destination
const public = funnel('public', {
  annotation: "Public files",
});

let tree = merge([html, js, css, public], {annotation: "Final output"});

if (!isProduction) {
  // Log the output tree
  tree = log(tree, {
    output: 'tree',
  });

  // Include the live reload server
  tree = new LiveReload(tree, {
    target: 'index.html',
  });
}

module.exports = tree;
```

So what we're doing here is 2 things.

1. Moving the Rollup plugins into a variable, so we can append the `uglify()` plugin only for production
2. Compressing the CSS only for production

That's it, now try building your prod application: `yarn build-prod`

Completed Branch: [12-minify](https://github.com/oligriffiths/broccolijs-tutorial/tree/12-minify)

Next: [13-fingerprints](13-fingerprints.html)
