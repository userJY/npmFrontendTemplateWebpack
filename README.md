# MWS - Template for building frontend npm packages via Webpack

## Summary

* Write our code in the `src` folder. 
* Webpack will build our js and types to the `lib` folder and the htmlplugin generate a html file in the `lib/demo` folder to run our library on the browser.  
* On publish, we simply delete the `lib/demo` folder and publish the rest of the `lib` folder.  

## To run

```bash
npm run build #calls webpack build to emit the js libraries and index.html files
npm run dev #runs the webserver
```

## To publish

```bash
npm run pub
```

package.json

```json
  "main": "./lib/index.js",
  "types": "./lib/index.d.ts",
  "files": [
    "lib/**/*"
  ],
  "scripts": {
    "build": "webpack",
    "dclean": "rm -rf docs",
    "doc": "rm -rf docs && typedoc --entryPointStrategy resolve ./src/index.ts",
    "dev": "npm run clean && webpack && webpack serve --https",
    "clean": "tsc --build --clean",
    "publish" : "npm run build && rm -rf lib/demo && npm run doc && npm publish",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

## webpack.config.js

* repo
  * package.json
  * tsconfig.json
  * webpack.config.js
  * lib    <---- `path: path.resolve(__dirname, 'lib'),` output used for publishing
    * demo
      * index.html   <--- `filename: path.resolve('lib', 'demo','index.html'),` HTMLWebpackPlugin input (3)
    * @types    <----`{ from: "src/@types", to: "@types" },` CopyPlugin output (5)
      * custom.d.ts  
    * index.d.ts <--- Generated from typescript declarations
    * index.js  <---- `filename: 'index.js',` (2)
  * src
    * @types   <----`{ from: "src/@types", to: "@types" },` CopyPlugin input (5)
      * custom.d.ts  
    * template.html <---- ` template: path.resolve('src','template.html') ` HTMLWebpackPlugin input (4)
    * index.ts  <-- `index: path.resolve(__dirname, 'src', 'index.ts'),`  (1)

```js
const path = require('path');
const HTMLWebpackPlugin = require('html-webpack-plugin')
const CopyPlugin = require("copy-webpack-plugin");

//commonJS syntax

module.exports = {
    mode: 'development',
    entry: {
        
        index: path.resolve(__dirname, 'src', 'index.ts'), // (1)
        
    },
    output: {
        path: path.resolve(__dirname, 'lib'),
        filename: 'index.js', // (2)
        clean: true,
        assetModuleFilename: '[name][ext]'
    },
    ...,
    plugins : [
        new HTMLWebpackPlugin({
            bleh: "Webpack App",
            filename: path.resolve('lib', 'demo','index.html'), // (3)
            template: path.resolve('src','template.html') // (4)
        }),
        new CopyPlugin({
            patterns: [
              { from: "src/@types", to: "@types" },  // (5)

            ],
          }),
    ],
    devServer: {
        static: {
            directory: path.resolve(__dirname,'lib', 'demo')
        },
        port: 5555,
        open: false,
        hot: true,
        compress: true,
        historyApiFallback: true,
    },
    devtool: "inline-source-map",
    //the compiled website is obfusticated JS, and errors will only show the compiled JS
    //source-map lets you know where in your source is your error
}
```





tsconfig.json

```json
{
  "compilerOptions": {
        /* Basics */
    "outDir": "./lib", /* output dir of emitted js files */
    "declaration": true, /*Generate *.d.ts files*/
    ...
    "skipLibCheck": true, /* skips checking *.d.ts of ALL node_modules/@types BUT type-checks if app depends on the module */
    "typeRoots": ["src/@types","node_modules/@types"], /*List of folder to include type definition, default is `node_modules/@types/ */
  },

  //Generally, include source directory; exclude distribution directory
  "include": ["src/**/*.ts", "src/**/*.tsx", "src/**/*.js", "src/**/*.jsx"], /* Specifies an array of filenames or patterns to include in the program. */
  "exclude": ["lib","node_modules", "doc", "jest.config.ts", "**/*.spec.ts"]  /* Specifies an array of filenames or patterns that should be skipped when resolving "include":[..] above */

}
```