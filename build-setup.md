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
- [How to make a pre-processor that makes it possible to use Pug/Jade](#how-to-make-a-pre-processor-that-makes-it-possible-to-use-pugjade)

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

```svelte
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

```svelte
<script lang="typescript">
  // Compatible with Svelte v3...
  export const hello: string = 'world';
</script>
<template>
  <div>Hello {hello}</div>
</template>
```

More information from community blogposts:

- https://codechips.me/how-to-use-typescript-with-svelte/

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

```svelte
<style lang="scss">
  $color: red;
  div {
    color: $color;
  }
</style>
```

The same is true for PostCSS as well, which also applies to any PostCSS plugins, like TailwindCSS (see [Tailwind demo here](https://github.com/tailwindcss/setup-examples/tree/master/examples/sapper)).

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

## How to make a pre-processor that makes it possible to use Pug/Jade

Pug is an alternative templating language that compiles to html, using whitespace instead of angle brackets:

```js
const pug = require("pug");
const string = "p Hello World!" // pug template language
const html = pug.render(file);
console.log(html); 		// <p>Hello World!</p>
```

Our goal is to write Pug in our Svelte files instead of HTML where we can do all the scoped styling and JS goodness. Let's create an `app.svelte` file with this: 

```svelte
<style>
  p {
    color: red;
  }
</style>

p Hello World!
```

We need to run it through Svelte to generate JavaScript, not HTML:

```js
const svelte = require("svelte/compiler");
const fs = require("fs");
const file = fs.readFileSync("app.svelte", "utf8");
const result = svelte.compile(file);
console.log(result.js.code); // svelte.compile returns both js and sourcemap
```

<details>
<summary>
But this logs out the wrong thing!
</summary>


```js
/* generated by Svelte v3.23.0 */
import {
        SvelteComponent,
        detach,
        init,
        insert,
        noop,
        safe_not_equal,
        text
} from "svelte/internal";

function create_fragment(ctx) {
        let t;

        return {
                c() {
                        t = text("p Hello World!"); // WRONG
                },
                m(target, anchor) {
                        insert(target, t, anchor);
                },
                p: noop,
                i: noop,
                o: noop,
                d(detaching) {
                        if (detaching) detach(t);
                }
        };
}

class Component extends SvelteComponent {
        constructor(options) {
                super();
                init(this, options, null, create_fragment, safe_not_equal, {});
        }
}

export default Component;
```
</details>

To do this properly, we need to *preprocess* the template file. Let's use `svelte.preprocess` instead of `svelte.compile`:

```js
const result = svelte.preprocess(file, {
  markup: ({ content }) => console.log(content)
});
```

Notice also that the `markup` includes the `style` tag as well. Some preprocessors might need that, but we don't, so we can strip it out. Finally, we can run the stripped code through `pug`:

```js
const pug = require("pug");
const svelte = require("svelte/compiler");
const fs = require("fs");
const file = fs.readFileSync("app.svelte", "utf8");

// https://github.com/sveltejs/svelte/blob/8fc85f0ef6b53ed85e54c129d79270fe577626dc/src/compiler/preprocess/index.ts#L97
const styleRegex = /<!--[^]*?-->|<style(\s[^]*?)?>([^]*?)<\/style>/gi;
const scriptRegex = /<!--[^]*?-->|<script(\s[^]*?)?>([^]*?)<\/script>/gi;
(async function () {
  const result = await svelte.preprocess(file, {
    // options
    markup: ({ content }) => {
      let code = content.replace(styleRegex, "").replace(scriptRegex, "");
      code = pug.render(code);
      return { code };
    },
  });
  console.log(result); // <p>Hello World!</p>
})();
```

This looks right. Our final task is to run this through `svelte.compile`:

```js
const res = svelte.compile(result.code);
console.log(res.js.code);
```



<details>
<summary>
And this generates the correct result.
</summary>

```js
/* generated by Svelte v3.23.0 */
import {
        SvelteComponent,
        detach,
        element,
        init,
        insert,
        noop,
        safe_not_equal
} from "svelte/internal";

function create_fragment(ctx) {
        let p;

        return {
                c() {
                        p = element("p");
                        p.textContent = "Hello World!"; // correct!
                },
                m(target, anchor) {
                        insert(target, p, anchor);
                },
                p: noop,
                i: noop,
                o: noop,
                d(detaching) {
                        if (detaching) detach(p);
                }
        };
}

class Component extends SvelteComponent {
        constructor(options) {
                super();
                init(this, options, null, create_fragment, safe_not_equal, {});
        }
}

export default Component;
```

</details>

The [`svelte.preprocess` API](https://svelte.dev/docs#svelte_preprocess) also offers a `dependencies` array you can use to optimize for recompiling when files are changed.

Most of the time, you will be using a bundler plugin rather than using `svelte.compile` and `svelte.preprocess` directly. These plugins have a slightly different API, but you can still slot in your handwritten preprocessor code inline:

```js
// example rollup.config.js
import * as fs from 'fs';
import svelte from 'rollup-plugin-svelte';
const pug = require("pug");
const styleRegex = /<!--[^]*?-->|<style(\s[^]*?)?>([^]*?)<\/style>/gi;
const scriptRegex = /<!--[^]*?-->|<script(\s[^]*?)?>([^]*?)<\/script>/gi;

export default {
  input: 'src/main.js',
  output: {
    file: 'public/bundle.js',
    format: 'iife'
  },
  plugins: [
    svelte({
      // etc
      preprocess: {
        markup: ({ content }) => {
	      let code = content.replace(styleRegex, "").replace(scriptRegex, "");
	      code = pug.render(code);
	      return { code };
        }
      },
    })
  ]
}
```

If you want to use Pug and HTML markup interchangeably, you may wish to adopt the non-standard but community norm of adding a `<template>` tag for non-HTML markup:

```svelte
<style>
  p {
    color: red;
  }
</style>

<template lang="pug">p Hello World!</template>
```

This guarantees no ambiguity when you have a Svelte codebase that is a mix of Pug and HTML templates. In order to process this you will have to add some checking and preprocessing:

```js
const pug = require("pug");
const svelte = require("svelte/compiler");
const fs = require("fs");
const file = fs.readFileSync("app.svelte", "utf8");

// https://github.com/sveltejs/svelte/blob/8fc85f0ef6b53ed85e54c129d79270fe577626dc/src/compiler/preprocess/index.ts#L97
const styleRegex = /<!--[^]*?-->|<style(\s[^]*?)?>([^]*?)<\/style>/gi;
const scriptRegex = /<!--[^]*?-->|<script(\s[^]*?)?>([^]*?)<\/script>/gi;
const markupPattern = new RegExp(
  `<template([\\s\\S]*?)(?:>([\\s\\S]*)<\\/template>|/>)`
);
(async function () {
  const result = await svelte.preprocess(file, {
    // options
    markup: ({ content }) => {
      // lifted from https://github.com/sveltejs/svelte-preprocess/blob/38b32b110b7e81c995da14dc813f002346b9e0af/src/autoProcess.ts#L179
      const templateMatch = content.match(markupPattern);
      if (!templateMatch) return { code: content };
      const [fullMatch, attributesStr, templateCode] = templateMatch;
      /** Transform an attribute string into a key-value object */
      const attributes = attributesStr
        .split(/\s+/)
        .filter(Boolean)
        .reduce((acc, attr) => {
          const [name, value] = attr.split("=");
          // istanbul ignore next
          acc[name] = value ? value.replace(/['"]/g, "") : true;
          return acc;
        }, {});
      /** Transform the found template code */
      let code = content.replace(styleRegex, "").replace(scriptRegex, "");
      if (attributes.lang === "pug") {
        code = templateCode;
        code = pug.render(code);
      }
      code =
        content.slice(0, templateMatch.index) +
        code +
        content.slice(templateMatch.index + fullMatch.length);
      return { code };
    },
  });
  const res = svelte.compile(result.code);
  console.log(res.js.code);
})();
```

This makes the preprocessor only work if `<template lang="pug">` is used.


[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
