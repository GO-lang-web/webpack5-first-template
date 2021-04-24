## 目标

- webpack 将 js 打包成 nodejs 浏览器环境都可以用 怎么实现？

```js
|- webpack.config.js
  |- package.json
  |- /src
    |- index.js
    |- ref.json
```

init project

```js
npm init -y
npm install --save-dev webpack webpack-cli lodash
```

We install lodash as devDependencies instead of dependencies because we don't want to bundle it into our library, or our library could be easily bloated.

## Webpack Configuration

webpack.config.js

```js
const path = require("path");

module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "webpack-numbers.js",
    clean: true,
    library: {
      name: "webpackNumbers",
      type: "umd",
    },
  },
  externals: {
    lodash: {
      commonjs: "lodash",
      commonjs2: "lodash",
      amd: "lodash",
      root: "_",
    },
  },
};
```

It should be familiar have you used webpack to bundle your application. Basically, we're telling webpack to bundle src/index.js into dist/webpack-numbers.js.

## Expose the Library

So far everything should be the same as bundling an application, and here comes the different part – we need to expose exports from the entry point through **output.library** option.

We exposed the entry point as webpackNumbers so users can use it through script tag:

```js
<script src="https://example.org/webpack-numbers.js"></script>
<script>
  window.webpackNumbers.wordToNum('Five');
</script>
```

However it only works when it's referenced through script tag, it can't be used in other environments like CommonJS, AMD, Node.js, etc.

As a library author, we want it to be compatible in different environments, i.e., users should be able to consume the bundled library in multiple ways listed below:

- CommonJS module require:

```js
const webpackNumbers = require("webpack-numbers");
// ...
webpackNumbers.wordToNum("Two");
```

- AMD module require:

```js
require(["webpackNumbers"], function (webpackNumbers) {
  // ...
  webpackNumbers.wordToNum("Two");
});
```

- script tag:

```js
<!DOCTYPE html>
<html>
  ...
  <script src="https://example.org/webpack-numbers.js"></script>
  <script>
    // ...
    // Global variable
    webpackNumbers.wordToNum('Five');
    // Property in the window object
    window.webpackNumbers.wordToNum('Five');
    // ...
  </script>
</html>
```

```js
    library: {
      name: 'webpackNumbers',
      type: 'umd',
    },
```

Now webpack will bundle a library that can work with CommonJS, AMD, and script tag.

add npm scripts in package.json

`````js
"object" == typeof exports && "object" == typeof module
  ? (module.exports = t())
  : "function" == typeof define && define.amd
  ? define([], t)
  : "object" == typeof exports
  ? (exports.webpackNumbers = t())
  : (n.webpackNumbers = t());
````;
`````

run npx webpack to check the output in dist file

## Externalize Lodash

Now, if you run npx webpack, you will find that a largish bundle is created. If you inspect the file, you'll see that lodash has been bundled along with your code. In this case, we'd prefer to treat lodash as a peer dependency. Meaning that the consumer should already have lodash installed. Hence you would want to give up control of this external library to the consumer of your library.

This can be done using the externals configuration:

```js
 externals: {
     lodash: {
       commonjs: 'lodash',
       commonjs2: 'lodash',
       amd: 'lodash',
       root: '_',
     },
   },
```

This means that your library expects a dependency named lodash to be available in the consumer's environment.

## External Limitations

For libraries that use several files from a dependency:

```js
import A from "library/one";
import B from "library/two";

// ...
```

You won't be able to exclude them from the bundle by specifying library in the externals. You'll either need to exclude them one by one or by using a regular expression.

```js
module.exports = {
  //...
  externals: [
    "library/one",
    "library/two",
    // Everything that starts with "library/"
    /^library\/.+$/,
  ],
};
```

## Final Steps

Optimize your output for production by following the steps mentioned in the production guide. Let's also add the path to your generated bundle as the package's main field in with the package.json

```js
"main": "dist/webpack-numbers.js",
```

Now you can publish it as an npm package and find it at unpkg.com to distribute it to your users.

- [publish a npm package](https://docs.npmjs.com/packages-and-modules/contributing-packages-to-the-registry)
-

## fix errors

- errors : 403 Forbidden - PUT https://registry.npmjs.org/alexgy1-first-npm-package - Forbidden

- If you are a new user, please confirm your account first! done

- [self is not defined](https://stackoverflow.com/questions/64639839/typescript-webpack-library-generates-referenceerror-self-is-not-defined)

## useful articles

- [publish-to-npm](https://zellwk.com/blog/publish-to-npm/)
- [author-libraries](https://webpack.js.org/guides/author-libraries/)
