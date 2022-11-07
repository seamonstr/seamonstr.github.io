# JS environment management and  packaging

## JS Environment

It's not a common practise to use a node virtual environment per project like Python, because unlike Python, node already keeps installed packages separate from the globally installed environment.

`npm`, when run without `-g`, creates a directory called `./node_modules` to install new modules into; this is used as part of the search directory in addition to the globally installed packages.

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
