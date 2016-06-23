Configuring Ava for Testing a React / Webpack Project
=====================================================
[Ava][ava] is a javascript test-runner.


Install Dependencies
--------------------
Install and save these dev dependencies.
```
$ npm install -D ava enzyme sinon react-addons-test-utils babel-plugin-webpack-loaders
```

Set up Ava in `package.json`
----------------------------
This is a basic AVA configuration with two npm scripts: `test` and `watch`.

`package.json`:
```json
"scripts": {
  "test": "ava",
  "watch": "npm t -- -w"
},
"ava": {
  "files": [
    "hzdg/static/hzdg/**/__tests__/**/*.js"
  ],
  "concurrency": 5,
  "failFast": true,
  "tap": false
}
```

Transpile `stage-0` modules with babel
--------------------------------------
AVA has babel `stage-2` built in but if you want to use `stage-0` stuff in your tests or your modules, you need to make a couple adjustments to the test env.

By default AVA has the `"espower", "transform-runtime"` plugins so we add them but we also add the `transform-decorators-legacy` so we can use babel 5's decorators (decorators are still in development in babel 6).

We will add our AVA-babel configuration to out `.babelrc` inside the `test` env.

`.babelrc`:
```json
{
  "env": {
    "test": {
      "plugins": [
        "espower",
        "transform-runtime",
        "transform-decorators-legacy",
      ],
      "presets": ["es2015", "stage-0", "react"]
    }
  }
}
```

We edit our AVA configuration in `.package.json` to use our `.babelrc` for its babel configuration and configure our `npm test` command to set the node environment to `test`.

`package.json`:
```json
"scripts": {
  "test": "NODE_ENV=test ava",
  "watch": "npm t -- -w",
},
"ava": {
  "files": [
    "hzdg/static/hzdg/**/__tests__/**/*.js"
  ],
  "concurrency": 5,
  "failFast": true,
  "tap": false,
  "require": [
    "babel-register"
  ],
  "babel": "inherit"
}
},
```


Add webpack support
-------------------
Webpack allows for a few things to go into your code that normally wouldn't be there so we need to help AVA deal with it.

The [`babel-plugin-webpack-loaders`][babel-plugin-webpack-loaders] package will help AVA work with our webpack loaders and [`resolve.modules` directories][webpack-resolve-modules]. We just need to edit our `.bablrc` test env and npm test script to get it configured.

Add the `babel-plugin-webpack-loaders` plugin to `.babelrc` and specify `${CONFIG}` as the config path. We need an absolute path to the webpack config file and we can pass it in as an env argument.
`.babelrc`:
```json
{
  "env": {
    "test": {
      "plugins": [
        "espower",
        "transform-runtime",
        "transform-decorators-legacy",
        ["babel-plugin-webpack-loaders", {
          "config": "${CONFIG}"
        }]
      ],
      "presets": ["es2015", "stage-0", "react"]
    }
  }
}
```

Specify the absolute path of your webpack config in the `CONFIG` env variable. You also have to disable AVA's babel cache to prevent some unwanted caching.

```json
"scripts": {
  "test": "NODE_ENV=test CONFIG=$(pwd)/webpack.config.js BABEL_DISABLE_CACHE=1 ava",
  "watch": "npm t -- -w"
},
```

If you're using __webpack 2__'s new `resolve.modules` in your webpack config you will need to use absolute paths or prefix the paths with a `./` to get them to work with `babel-plugin-webpack-loaders`

`webpack.config.js`:
```javascript
modules: [
  path.resolve(__dirname, 'src'),
  path.resolve(__dirname, 'node_modules'),
],
```

<!-- resources -->

[ava]: https://github.com/avajs/ava
[enzyme]: http://airbnb.io/enzyme/
[sinon]: http://sinonjs.org/
[babel-plugin-webpack-loaders]: https://github.com/istarkov/babel-plugin-webpack-loaders
[webpack-resolve-modules]: https://github.com/istarkov/babel-plugin-webpack-loaders/pull/18
