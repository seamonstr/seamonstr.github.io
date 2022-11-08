---
title: Managing javascript environments, code and builds
---

## JS Environment

It's not a common practise to use a node virtual environment per project like Python, because unlike Python, node already keeps installed packages separate from the globally installed environment.

### How node manages modules

`npm`, when run without `-g`, creates a directory called `./node_modules` to install new modules into; this is used as part of the search directory in addition to the globally installed packages.

When you run a node application and require or import a module, the following locations are searched:
1. The script's parent directory's `node_modules`, then the parent's parent's `node_modules`, and so on to the root of the file system.
1. Anything listed in NODE_PATH, which is `:` or `;` (windows) delimited
1. The global directories in `~/.node_modules` and `~/.node_libraries`
1. $node_prefix/lib/node

Obviously it's much more complicated than this; see the [full resolution algo[(https://nodejs.org/api/esm.html#resolver-algorithm-specification).

### npx

`npx` is part of `npm`, and is used to run commands from packages that would ordinarily be on the `bin` path somewhere; eg. `webpack`.

`npx` will try to find the command in all installed packages; if it doesn't find it, it'll search the `npmjs` repository for it. If it finds it there, it'll offer to download it and run it (with any dependencies).

It will *not* add those downloaded packages to either the `package.json` or to the current `.node_modules` directory; `npx` maintains its own cache of packages (in `~/.npm/_npx` on Unixy OSes).

### Managing node project deps

To cleanly manage project dependencies with `npm`:
1. Manage the global version of node you're using with [`nvm`](https://github.com/nvm-sh/nvm#installing-and-updating).
2. Create a dependencies file in your project root called `package.json`:

```json
{ 
    "name": "yourapp", 
    "version": "0.0.1", 
    "dependencies":  {
        "jade": "0.4.1"
    }
}
```
3. Install the contents with `npm install`
4. Add new packages to the project with either `npm add` or `npm install` (aliases of each other) - this updates the `package.json` file and the `package-lock.json` generated file.
5. If you delete packages from the `package.json` file, running `node install` (no args) will update your `node_modules` directory to remove the ones no longer needed.


## Webpack

Webpack is a bundler. It takes a JS file as import, compiles it, and generates a single JS file containing all of the input files identities that can be downloaded as a single asset.

* Add it to the project with `npm add webpack; npm add webpack-cli`
* Run it with `npx webpack-cli app_entrypoint.js`
* It generates the output in `dist/main.js`
* Works for both commonJS, ES6 and AMD modules
* Generates a script file you can call directly, so unlike with AMD loading, you don't need to specify the module loader ("requirejs") as the script entrypoint:

```html
    <script src="dist/main.js" ></script>
```

* Webpack has a config file that you can use to tweak it's inputs, outputs and settings. The file is a module, either CommonJS or ES style. A basic example follows; because it's an ES module, it needs to be called `webpack.config.mjs`. If it was the CommonJS style, it needs to be `webpack.common.js`.

You can run `webpack` with no args in a directory that has a config file in it.

```js
import * as Url from "node:url";

console.log(import.meta);

// This object is the interface to the config module - `entry` and `output`.
export default {
    entry:{
        main: "./test.js" // doesn't have to be called `main`; names the output js
        // Could have more entries here with different inputs & output names
    },
    output: {
        path: Url.fileURLToPath(new URL('dist', import.meta.url)),
        filename: "[name].js" // Templates the name from the `entry` section
    }
}
```

### Bundling css with Webpack

Webpack bundles CSS into the app by configuring a pipeline of modules/plugins in the webpack config:
1. `css-loader` reads the specified css files, and implements support for a `@import url("file2.css");` directive, which isn't standard css. `css-loader` does all of the includes to support this.
2. `style-loader` takes the css output from `css-loader` and generates a Javascript module that can inject the css as styles into the `<head>` element of an HTML doc.
    * The resulting script can be directly referenced by the HTML in a `script` tag: `<script src="dist/style.js"></script>`
    * ...or just referenced in an `import` in another JS script that is referenced from the page:

```js
// Webpack will translate this to JS and inject it into any HTML file that 
// uses this script
import "./file.css" 
```

Example webpack config that achieves this:

```js
import * as Url from "node:url";

console.log(import.meta);

export default {
    entry:{
        style: "./file.css"
    },
    output: {
        path: Url.fileURLToPath(new URL('dist', import.meta.url)),
        filename: "[name].js"
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                // the pipeline is executed in inverse order to its declaration
                use: ['style-loader', 'css-loader']
            }
        ]
    }
}
```

### Webpack dev server

The issue with Webpack is that you don't get live updates as you edit code during development; there's now a build process in between your code and the browser-served artefacts.

Webpack dev server solves this issue - it watches the sources, rebuilds the distribution when something's edited and pings the browser to reload. You get a page on reload that gives you any warnings from the compilation. This is achieved by appending an iframe to the page's `body` content with a tagged `div` in it. A webpack-dev-server client is injected into the compiled JS bundle which adds content to that div, and sets up a loop to check with the server for updates.

Dev server config needs to be in the webpack config. An example is as follows; specifically note the `output/publicPath` and `devserver/static` settings.

```js
import path from "node:path";
import * as Url from "node:url";

const dirname = Url.fileURLToPath(new URL('.', import.meta.url))
export default {
    mode: 'development',
    devtool: "nosources-source-map",
    entry:{
        style: "./file.css",
        start: "./main.js"
    },
    output: {
        path: path.join(dirname, 'dist'),
        // devserver will mount above `path` onto `publicPath`
        // ie.  ./dist/start.js will serve as http://localhost/js/start.js
        publicPath: '/js/', 
        filename: "[name].js"
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: ['style-loader', 'css-loader']
            }
        ]
    },
    devServer: {
        // devserver will mount this directory onto the root URL
        // ie. ./public/index.html will be at http://localhost/index.html
        static: './public'
    }
}
```