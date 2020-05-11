# Build Setup Recipes

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Using Svelte with other technologies (e.g. PostCSS, SCSS, TypeScript, and Babel)](#using-svelte-with-other-technologies-eg-postcss-scss-typescript-and-babel)
- [Transpiling ES6 to ES5 for Legacy Browser (IE11) Support with Babel](#transpiling-es6-to-es5-for-legacy-browser-ie11-support-with-babel)
- [Using Future JS Syntax in Svelte with Babel](#using-future-js-syntax-in-svelte-with-babel)
- [Using TypeScript with Svelte](#using-typescript-with-svelte)
- [Using PostCSS/SCSS with Svelte](#using-postcssscss-with-svelte)
- [Writing Your Own Preprocessors](#writing-your-own-preprocessors)
- [How to make a pre-processor that makes it possible to use PUG/Jade](#how-to-make-a-pre-processor-that-makes-it-possible-to-use-pugjade)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Using Svelte with other technologies (e.g. PostCSS, SCSS, TypeScript, and Babel)

Svelte doesn't build in any opinions on the kind of JavaScript, CSS, or even HTML you write. Therefore, if you want to use any nonstandard syntax, you need to put that into your bundler plugin chain or Svelte's preprocessors. Here we will describe the top usecases for doing this.

First we will discuss Babel. There are two different usecases for Babel with Svelte - with accordingly two separate ways to implement it:

- Transpiling ES6 to ES5: use Babel with bundler plugin
- Using future JS syntax: use Babel with Svelte preprocessor

## Transpiling ES6 to ES5 for Legacy Browser (IE11) Support with Babel

Svelte outputs modern JavaScript, therefore for legacy browser support, you need to transpile this output back to ES5 (or older). The main strategy people adopt is having 2 builds:

- Modern JS build - transpile Svelte as is
- Legacy browser build - add Babel plugin to bundler, after Svelte plugin.

You can use an environment variable like `process.env.IS_LEGACY_BUILD` (the name is arbitrary - Sapper calls it [`SAPPER_LEGACY_BUILD`](https://github.com/sveltejs/sapper-template/blob/57c430390a5980a9e461206763a619c64ed910e0/rollup.config.js#L12)) to toggle this behavior between builds.

If you are using [Parcel](https://parceljs.org/), this should be taken care of by specifying the correct `.babelrc`.

If you are using [Sapper](https://github.com/sveltejs/sapper-template), this should be correctly set up for you.

If you are using Rollup or Webpack, you need to add the respective Babel plugins.

<details>

<summary>Rollup</summary>

Assuming you want to toggle on and off a modern and a legacy build using a `IS_LEGACY_BUILD` environment variable:

```js
// // rollup.config.js
// other imports
import svelte from "rollup-plugin-svelte";
import babel from "rollup-plugin-babel";

const legacy = !!process.env.IS_LEGACY_BUILD;

export default {
  // etc...
  plugins: [
    svelte({
      // etc..
    }),
    // this should come after the Svelte plugin
    legacy &&
      babel({
        extensions: [".js", ".mjs", ".html", ".svelte"],
        runtimeHelpers: true,
        exclude: ["node_modules/@babel/**"],
        presets: [
          [
            "@babel/preset-env",
            {
              targets: "> 0.25%, not dead",
            },
          ],
        ],
        plugins: [
          "@babel/plugin-syntax-dynamic-import",
          [
            "@babel/plugin-transform-runtime",
            {
              useESModules: true,
            },
          ],
        ],
      }),
    // ...
  ],
};
```

Of course, make sure to have the relevant dependencies like `rollup-plugin-babel @babel/core` and whatever presets and plugins you use installed.

</details>

<details>
<summary>Webpack
</summary>

_To be written - please contribute!_

</details>

## Using Future JS Syntax in Svelte with Babel

The other, less common but still handy usecase for Babel with Svelte is transpiling _future_ syntax inside `<script>` tags. For example, the [Stage 3 Optional Chaining proposal](https://babeljs.io/docs/en/babel-plugin-proposal-optional-chaining) is popular, but not yet in browsers, and more relevantly, is not yet understood by [Acorn]https://github.com/acornjs/acorn), which is what Svelte uses to parse `<script>` tags.

<details>
<summary>
Example usage:
</summary>

```html
<script>
  let foo = {
    bar: {
      baz: true,
    },
  };
  let sub = foo?.ban?.baz;
</script>

<main>
  <h1>Hello {sub}!</h1>
</main>
```

When we try to run this, Acorn throws a `ParseError: Unexpected token` error.

</details>

Here, the fix is to use Svelte's [Preprocess](https://svelte.dev/docs#svelte_preprocess) feature:

```js
// rollup.config.js
// ...

export default {
	// ...
  plugins: [
    svelte({
      // ...
      preprocess: {
        script: ({ content }) => {
          return require("@babel/core").transform(content, {
            plugins: ["@babel/plugin-proposal-class-properties"],
          });
        },
      },
    }),
 // ...
```

Of course, make sure that `@babel/core` and whatever plugins you use are installed.

Alternatively, you might wish to use [`svelte-preprocess`](https://github.com/kaisermann/svelte-preprocess/) instead, for its' other features:

```js
import preprocess from "svelte-preprocess";
// ...
preprocess: preprocess({
  babel: {
    presets: [
      [
        "@babel/preset-env",
        {
          loose: true,
          // No need for babel to resolve modules
          modules: false,
          targets: {
            // ! Very important. Target es6+
            esmodules: true,
          },
        },
      ],
    ],
  },
});
// ...
```

**Important note: Don't try to transpile to ES5 here**. This would produce unnecessary bloat as you would be transpiling on a per-component basis - [sharing Babel helpers is the reason you use bundler plugins instead](https://github.com/rollup/rollup-plugin-babel#why). Keep it light, only transpile the new stuff you use. This also makes it easy to remove in future.

You may also wish to use new JS syntax inside the JS expressions in your template markup. There is currently no easy way to do that, but [watch this issue](https://github.com/sveltejs/svelte/issues/4701).

## Using TypeScript with Svelte

**It is a common misconception that Svelte doesn't work with TypeScript yet**. Svelte is, of course, written in TypeScript, so the project strongly believes in the value of typing. The part that doesn't have full support yet is IDE-supported typechecking of templates, and [there is ongoing work on a langauge server with some pointers from the TypeScript team](https://github.com/sveltejs/language-tools/issues/11).

However, using TypeScript inside `<script>` tags is supported, and that is likely an important part of how you will be using TypeScript anyway. All of them use Svelte's preprocessor feature:

- You can use `[@babel/preset-typescript](https://babeljs.io/docs/en/babel-preset-typescript)` together with the Babel process described above to strip out types.
- You can pass the source code [through the `typescript` compiler](https://github.com/microsoft/TypeScript/wiki/Using-the-Compiler-API#a-simple-transform-function):

  ```ts
  // rollup.config.js
  import * as ts from "typescript";
  // ...
  export default {
    // ...
    plugins: [
      svelte({
        // ...
        preprocess: {
          script: ({ content }) => {
            return ts.transpileModule(content, {
              compilerOptions: { // up to you }
              })
          },
        },
      }),
  // ...
  ```

- Or instead of writing this all yourself, you can use [`svelte-preprocess`](https://github.com/kaisermann/svelte-preprocess/), which is what we recommend.

Example using `svelte-preprocess`:

```js
// rollup.config.js
import svelte from 'rollup-plugin-svelte';
import autoPreprocess from 'svelte-preprocess'
import { scss, coffeescript, pug } from 'svelte-preprocess'

export default {
  ...,
  plugins: [
    svelte({
      /**
       * Auto preprocess supported languages with
       * '<template>'/'external src files' support
       **/
      preprocess: autoPreprocess({ /* options available https://github.com/kaisermann/svelte-preprocess/#user-content-options */ })
    })
  ]
}
```

and then you can write your Svelte components with TypeScript:

```html
<script lang="typescript">
  // Compatible with Svelte v3...
  export const hello: string = 'world';
</script>
<template>
  <div>Hello {hello}</div>
</template>
```

## Using PostCSS/SCSS with Svelte

**Option 1 - Write your own**

This process is actually [documented in the Svelte docs](https://svelte.dev/docs#svelte_preprocess), but we'll restate it here with the bundler case:

```js
// rollup.config.js
const sass = require('node-sass');
// ...
export default {
  // ...
  plugins: [
    svelte({
      // ...
      preprocess: {
        style: ({ content, attributes, filename }) => {
          // only process <style lang="sass">
          if (attributes.lang !== 'sass') return;

          const { css, stats } = await new Promise((resolve, reject) => sass.render({
              file: filename,
            data: content,
            includePaths: [
              dirname(filename),
            ],
          }, (err, result) => {
            if (err) reject(err);
            else resolve(result);
          }));

          // TODO: not sure about this. maybe supposed to just return css.toString() instead
          return {
            code: css.toString(),
            dependencies: stats.includedFiles
          };
        },
      },
    }),
// ...
```

**Option 2 - use `svelte-preprocess`**

Setting up `svelte-preprocess` can help simplify this:

```js
// rollup.config.js
import svelte from 'rollup-plugin-svelte';
import autoPreprocess from 'svelte-preprocess'
import { scss, coffeescript, pug } from 'svelte-preprocess'

export default {
  ...,
  plugins: [
    svelte({
      /**
       * Auto preprocess supported languages with
       * '<template>'/'external src files' support
       **/
      preprocess: autoPreprocess({ /* options available https://github.com/kaisermann/svelte-preprocess/#user-content-options */ })
    })
  ]
}
```

and you can write SCSS/SASS in your Svelte template:

```html
<style lang="scss">
  $color: red;
  div {
    color: $color;
  }
</style>
```

The same is true for PostCSS as well, which also applies to any PostCSS plugins, like TailwindCSS.

## Writing Your Own Preprocessors

> This article references `svelte.preprocess` throughout but you may be more familiar with the `preprocess` option of `svelte-loader` or `rollup-plugin-svelte`. This `preprocess` option calls `svelte.preprocess` internally. The bundler plugin gives you easy access to it, so you don't need to transform your components before compilation manually.

The Svelte compiler expects all components it receives to be valid Svelte syntax. To use compile-to-js or compile-to-css languages, you need to make sure that any non-standard syntax is transformed before Svelte tries to parse it. To enable this Svelte provides a `preprocess` method allowing you to transform different parts of the component before it reaches the compiler.

With `svelte.preprocess` you have a great deal of flexibility in how you write your components while ensuring that the Svelte compiler receives a plain component.

### svelte.preprocess

Svelte's `preprocess` method expects an object or an array of objects with one or more of `markup`, `script`, and `style` properties, each being a function receiving the source code as an argument. The preprocessors run in this order.

```js
const preprocess = {
  markup,
  script,
  style,
};
```

In general, preprocessors receive the component source code and must return the transformed source code, either as a string or as an object containing a `code` and `map` property. The `code` property must contain the transformed source code, while the `map` property can optionally contain a sourcemap. The sourcemap is currently unused by Svelte.

## How to make a pre-processor that makes it possible to use PUG/Jade

_to be written_

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
