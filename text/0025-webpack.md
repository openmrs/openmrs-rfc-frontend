# Project title
- Start Date: 2020/04/25
- RFC PR: https://github.com/openmrs/openmrs-rfc-frontend/pull/25

## Decision, including impact on distributions
All microfrontends in the OpenMRS space should use the same base webpack
configuration and tooling. To simplify sharing this information a common
utility package (e.g., `@openmrs/esm-devtools`, reusing the existing package
for everything related to development would work, too) should be created and
used.

Ideally, the new package exports commands for debugging and building. As a
result, the *package.json* of the existing microfrontend repositories changes
from

```json
{
  "scripts": {
    "start": "webpack-dev-server",
    "build": "webpack --mode production"
  },
  // ...
}
```

to

```json
{
  "scripts": {
    "start": "oesm debug",
    "build": "oesm build"
  },
  // ...
}
```

The rest remains the same.

The commands would just use a predefined version of `webpack` and associated
packages (e.g., `webpack-dev-server`) to trigger a build using a preconfigured
version of a webpack file that reads the remaining information from the local
*package.json*.

The basic *webpack.config.js* would is similar to the following:

```js
const path = require("path");
const CleanWebpackPlugin = require("clean-webpack-plugin").CleanWebpackPlugin;
const ForkTsCheckerWebpackPlugin = require("fork-ts-checker-webpack-plugin");

const root = process.cwd();
const name = require(path.resolve(root, 'package.json')).name;
const fileName = name.replace(/@/g, '').replace(/\//g, '-');
const jsonName = fileName.replace(/\-/g, '_');
const externals = [
  // list all shared externals here
  "single-spa",
  "react",
  "react-dom",
  /^@openmrs\/esm.*/,
  "i18next",
  "react-i18next",
];

module.exports = {
  entry: path.resolve(root, "src/index.ts"),
  output: {
    filename: `${fileName}.js`,
    libraryTarget: "system",
    path: path.resolve(root, "dist"),
    jsonpFunction: `webpackJsonp_${jsonName}`,
  },
  module: {
    rules: [
      {
        parser: {
          system: false,
        },
      },
      {
        test: /\.m?(js|ts|tsx)$/,
        exclude: /(node_modules|bower_components)/,
        use: {
          loader: "babel-loader",
        },
      },
      {
        test: /\.css$/,
        use: [
          { loader: "style-loader" },
          {
            loader: "css-loader",
            options: {
              modules: true,
            },
          },
        ],
      },
    ],
  },
  devtool: "sourcemap",
  devServer: {
    headers: {
      "Access-Control-Allow-Origin": "*",
    },
    disableHostCheck: true,
  },
  externals,
  plugins: [new ForkTsCheckerWebpackPlugin(), new CleanWebpackPlugin()],
  resolve: {
    extensions: [".tsx", ".ts", ".jsx", ".js"],
  },
};
```

While not necessary it should still be possible to override (parts) of this
file locally. However, in most cases no *webpack.config.js* should be required.

Furthermore, the package could come with a command to just scaffold a new
microfrontend. No copying and adjustment from an existing source required.

## Definition
A **bundler** is a tool for combining resources used to develop a frontend web
application. Its main feature is a pipeline for finding relevant resources and
combining them into distributables called bundles. A **bundle** is an output
file that is understood by a web browser, e.g., a JavaScript files.

**Webpack** is the bundler with arguably the largest community. It is also the
kind of bundler that is used by the OpenMRS microfrontend modules.

## Reason for decision
Maintaining a webpack file is tedious. Setting it up correctly is not that easy
and most of the time the file is just copied and slightly modified. The
situation gets worse if the modified file and the original file only differ in
details and if one of the details has only a non-direct impact. For instance,
if certain behavior only breaks at runtime when a detail is missed the bundler
works only in an instable state.

To avoid many pitfalls when writing webpack files we should not re-invent the
wheel for each microfrontend. Instead, we should use common tooling to avoid
writing custom webpack files. Consequently, the build gets a bit easier to
control and understand. Furthermore, scaffolding a new microfrontend should
become quite an easy and repeatable task.

## Alternatives
Besides packing the main webpack configuration in a dedicated package we can
also just enforce a common webpack configuration. The major problem with such
an approach is the implicit fragmentation.

Otherwise, going for another bundler that comes with everything out of the box
(i.e., zero configuration) would be another option. One possibility here is
Parcel. The major issue with such an approach is that the communities and
available options may not be acceptable for our needs.

## Common practices (not enforced)
- Webpack plugin for checking circular dependencies
- Linting and other options for the webpack file
- Enforcing small(er) bundle sizes
