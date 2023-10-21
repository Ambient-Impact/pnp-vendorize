Copy front-end assets from a
[Plug'n'Play](https://yarnpkg.com/features/pnp)-compliant package manager such
as [Yarn](https://yarnpkg.com/) or [pnpm](https://pnpm.io/) into a publicly
accessible vendor directory.

# Why?

If you're coming from the PHP world where [the Composer package
manager](https://getcomposer.org/) is the de facto standard in managing
dependencies, and you're working with a content management system (CMS) like
[Drupal](https://www.drupal.org/) that has its own framework for defining
libraries, handling dependency trees, and bundling them as needed for a given
route, then you might not want [Webpack](https://webpack.js.org/) to bundle
your vendor libraries yet still make use of Webpack's other features and
ecosystem of plug-ins.

# What?

Let's say you have `dependency1` and `dependency2` in your `package.json`:

```javascript
{
  "name": "my-package",
  // ...
  "dependencies": {
    // ...
    "dependency1": "^1.0.0",
    "dependency2": "^1.0.0",
    // ...
  }
}

```

Then you do a [`yarn install`](https://yarnpkg.com/cli/install) or
[`pnpm install`](https://pnpm.io/cli/install) and your package manager does its
magic to pull in the packages. Now they're in `.yarn/cache` or `node_modules`
but that's (hopefully) outside of your public web root. How do you make those
specific dependencies web accessible without writing custom code or bundling
via something like Webpack?

# How?

```bash
yarn add 'webpack-yarn-vendorize@github:Ambient-Impact/webpack-yarn-vendorize' --dev
```

or

```bash
pnpm add 'webpack-yarn-vendorize@github:Ambient-Impact/webpack-yarn-vendorize' --dev
```

Next, you need to define the subset of dependencies you want vendorized by
adding a `"vendorize"` entry to your `package.json`:

```javascript
{
  "name": "my-package",
  // ...
  "dependencies": {
    // ...
    "dependency1": "^1.0.0",
    "dependency2": "^1.0.0",
    // ...
  },
  "vendorize": [
    "dependency1",
    "dependency2"
  ]
}
```

Note that you must explicitly declare a package as a dependency. If you don't,
Vendorize will throw an error - even if the package has been installed by Yarn
from another workspace in the same project.

Now you can run `yarn run vendorize` or `pnpm run vendorize` and once it's
completed, you'll find a `vendor` directory inside your package like so:

```
my-package
↳ vendor
  ↳ dependency1
    ↳ ...
  ↳ dependency2
    ↳ ...
```

You can also automate this so you don't need to run the command yourself by
adding it as a `postinstall` script:

```javascript
{
  "name": "my-package",
  // ...
  "scripts": {
    // ...
    "postinstall": "yarn run vendorize",
    // ...
  },
  // ...
}
```

If using pnpm, replace `yarn run vendorize` with `pnpm run vendorize` above.

Now you only need to install your package and it'll automagically run Vendorize.

# Advanced

Vendorize can be configured by converting the `vendorize` key in your
`package.json` into an object with the array of packages to vendorize under a
`packages` key like so:

```javascript
{
  "name": "my-package",
  // ...
  "dependencies": {
    // ...
    "dependency1": "^1.0.0",
    "dependency2": "^1.0.0",
    // ...
  },
  "vendorize": {
    "packages": [
      "dependency1",
      "dependency2"
    ]
  }
}
```

Then you can specify one or more of the following:

```javascript
{
  // ...
  "vendorize": {
    "packages": [
      "dependency1",
      "dependency2"
    ],
    // If true (the default), will delete the contents of the vendor directory
    // before copying files into it.
    "cleanBefore": true,
    // Don't like the name "vendor"? Weird, but you can change it, sure.
    "dirName": "vendor",
    // If true (the default), a .gitignore file will be placed in the vendor
    // directory to ignore its contents.
    "gitIgnore": true
  }
}
```

# Miscellaneous

Not to be confused with [the `vendorize`
package](https://www.npmjs.com/package/vendorize), which only copies npm
packages from a `node_modules` directory.
