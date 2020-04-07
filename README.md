# recipes-mvp

MVP repo to just start collecting recipes for [Svelte Community](https://github.com/sveltejs/community). The core inspiration is https://vuejs.org/v2/cookbook/.

All examples have some variation "with Svelte" appended to them in an attempt to optimize for searchability.

# Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents**

- [Using Fetch to Consume APIs with Svelte](#using-fetch-to-consume-apis-with-svelte)
  - [Fetching on Component Mount in Svelte](#fetching-on-component-mount-in-svelte)
  - [Fetching on Button Click in Svelte](#fetching-on-button-click-in-svelte)
  - [Dealing with CORS Errors in Svelte](#dealing-with-cors-errors-in-svelte)
  - [Further Links](#further-links)
- [Using SASS and TypeScript with Svelte Preprocessors](#using-sass-and-typescript-with-svelte-preprocessors)
- [Routing with Svelte](#routing-with-svelte)
  - [Further Links](#further-links-1)
- [Authentication with Svelte](#authentication-with-svelte)
- [Form Validation with Svelte](#form-validation-with-svelte)
- [Editable SVG Icon Systems with Svelte](#editable-svg-icon-systems-with-svelte)
- [Create a CMS-Powered Blog with Svelte](#create-a-cms-powered-blog-with-svelte)
- [Unit Testing Svelte Components](#unit-testing-svelte-components)
  - [Creating a component](#creating-a-component)
  - [Changing component props](#changing-component-props)
  - [Testing component events](#testing-component-events)
  - [Testing slots](#testing-slots)
  - [Testing the context API](#testing-the-context-api)
- [Creating Custom Scroll Actions with Svelte](#creating-custom-scroll-actions-with-svelte)
- [Debugging Svelte Apps in VS Code](#debugging-svelte-apps-in-vs-code)
- [Avoiding Memory Leaks with Svelte](#avoiding-memory-leaks-with-svelte)
- [Client-Side Storage with Svelte](#client-side-storage-with-svelte)
- [Packaging Svelte Components for npm](#packaging-svelte-components-for-npm)
- [Dockerize a Svelte App](#dockerize-a-svelte-app)
- [Practical use of Svelte slots with Google Maps](#practical-use-of-svelte-slots-with-google-maps)
- [How to make a pre-processor that makes it possible to use PUG/Jade](#how-to-make-a-pre-processor-that-makes-it-possible-to-use-pugjade)
- [Reactivity](#reactivity)
  - [Reactive assignments](#reactive-assignments)
    - [Top-level variables](#top-level-variables)
    - [`each` blocks](#each-blocks)
    - [Variables not values](#variables-not-values)
    - [Shadowed variables](#shadowed-variables)
  - [Reactive Declarations](#reactive-declarations)
    - [Defining dependencies](#defining-dependencies)
    - [Variable deconstruction](#variable-deconstruction)
    - [Hiding values from reactive declarations](#hiding-values-from-reactive-declarations)
- [Stores](#stores)
  - [The store contract](#the-store-contract)
    - [`store.subscribe`](#storesubscribe)
    - [`store.set`](#storeset)
  - [Custom stores](#custom-stores)
    - [Linked stores](#linked-stores)
  - [a custom implementation of the builtin store](#a-custom-implementation-of-the-builtin-store)
- [Server Side Rendering](#server-side-rendering)
- [Preprocessors](#preprocessors)
  - [svelte.preprocess](#sveltepreprocess)
  - [Markup preprocessors](#markup-preprocessors)
  - [Script preprocessors](#script-preprocessors)
  - [Style preprocessors](#style-preprocessors)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Build Setup Recipes

## Using SASS and TypeScript with Svelte Preprocessors

_to be written_

## Preprocessors

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

### Markup preprocessors

_to be written_

### Script preprocessors

_to be written_

### Style preprocessors

_to be written_

## How to make a pre-processor that makes it possible to use PUG/Jade

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte Language Fundamentals

## Reactivity

### Reactive assignments

The reactivity system introduced in Svelte 3 has made it easier than ever to trigger updates to the DOM. Despite this, there are a few simple rules that you must always follow. This guide explains how Svelte's reactivity system works, what you can and can't do, as well a few pitfalls to avoid.

#### Top-level variables

The simplest way to make your Svelte components reactive is by using an assignment operator. Any time Svelte sees an assignment to a _top-level variable_ an update is scheduled. A 'top-level variable' is any variable that is defined inside the script element but is not a child of _anything_, meaning, it is not inside a function or a block. Incidentally, these are also the only variables that you can reference in the DOM. Let's look at some examples.

The following works as expected and update the dom:

```sv
<script>
  let num = 0;

  function updateNum() {
    num = 25;
  }
</script>

<button on:click={updateNum}>Update</button>
<p>{num}</p>
```

Svelte can see that there is an assignment to a top-level variable and knows to rerender after the `num` variable is modified.

#### `each` blocks

The only exception to the top-level variable rule is when you are inside an `each` block. Any assignments to variables inside an `each` block trigger an update. Only assignments to array items that are objects or arrays result in the array itself updating. If the array items are primitives, the change is not traced back to the original array. This is because Svelte only reassigns the actual array items and primitives are passed by value in javascript, not by reference.

The following example causes the array and, subsequently, the DOM to be updated:

```sv
<script>
  let list = [{ n: 1 }, { n: 2 }, { n: 3 }];
</script>

{#each list as item}
  <button on:click={() => item.n *= 2 }>{ item.n }</button>
{/each}
```

This, however, will not:

```sv
<script>
  let list = [1, 2, 3];
</script>

{#each list as item}
  <button on:click={() => item *= 2 }>{ item.n }</button>
{/each}
```

#### Variables not values

Svelte only cares about which _variables_ are being reassigned, not the values to which those variables refer. If the variable you reassign is not defined at the top level, Svelte does not trigger an update, even if the value you _are_ updating updates the original variable's value as well.

This is the sort of problem you may run into when dealing with objects. Since objects are passed by reference and not value, you can refer to the same value in many different variables. Let's look at an example:

```sv
<script>
  let obj = {
    num: 0
  };

  function updateNum() {
    const o = obj;
    o.num = 25;
  }
</script>

<button on:click={updateNum}>Update</button>
<p>{obj.num}</p>
```

In this example, when we reassign `o.num` we are updating the value assigned to `obj` but since we are not updating the actual `obj` variable SVelte does not trigger an update. Svelte does not trace these kinds of values back to a variable defined at the top level and has no way of knowing if it has updated or not. Whenever you want to update the local component state, any reassignments must be performed on the _actual_ variable, not just the value itself.

#### Shadowed variables

Another situation that can sometimes cause unexpected results is when you reassign a function's parameter (as above), and that parameter has the same _name_ as a top-level variable.

```sv
<script>
  let obj = {
    num: 0
  };

  function(obj) {
    obj.num = 25;
  }
</script>

<button on:click={() => updateNum(obj)}>Update</button>
<p>{num}</p>
```

This example behaves the same as the previous example, except it is perhaps even more confusing. In this case, the `obj` variable is being _shadowed_ while inside the function, so any assignments to `obj` inside this function are assignments to the function parameter rather than the top-level `obj` variable. It refers to the same value, and it has the same name, but it is a _different_ variable inside the function scope.

Reassigning function parameters in this way is the same as reassigning a variable that points back to the top-level variable's value and does not cause an update. To avoid these problems, and potential confusion, it is a good idea not to reuse variable names in different scopes (such as inside functions), and always make sure that you are reassigning a top-level variable.

### Reactive Declarations

In addition to the assignment-based reactivity system, Svelte also has special syntax to define code that should rerun when its dependencies change using labeled statements - `$:`.

```sv
<script>
  let n = 0;

  $: n_squared = n * n;
</script>

<button on:click={() => n += 1}>{ n_squared }</button>
```

Whenever Svelte sees a reactive declaration, it makes sure to execute any reactive statements that depend on one another in the correct order and only when their direct dependencies have changed. A 'direct dependency' is a variable that is referenced inside the reactive declaration itself. References to variables inside functions that a reactive declaration _calls_ are not considered dependencies.

```sv
<script>
  let n = 0;

  const squareIt = () => n * n;

  $: n_squared = squareIt();
</script>

<button on:click={() => n += 1}>{ n_squared }</button>
```

In the above example, `n_squared` will _not_ be recalculated when `n` changes because Svelte is not looking inside the `squareIt` function to define the reactive declaration's dependencies.

#### Defining dependencies

Sometimes you want to rerun a reactive declaration when a value changes but the variable itself is not required (in the case of some side-effects). The solution to this involves listing the dependency inside the declaration in some way.

The simplest solution is to pass the variable as an argument to a function, even if that variable is not required.

```js
let my_value = 0;

function someFunc() {
  console.log(my_value);
}

$: someFunc(my_value);
```

In this example, `someFunc` doesn't require the `my_value` argument, but this lets Svelte know that `my_value` is a dependency of this reactive declaration and should rerun whenever it changes. In general, it makes sense to use any parameters that are passed in, even though all of these variables are in scope, it can make the code easier to follow.

```js
let my_value = 0;

function someFunc(value) {
  console.log(value);
}

$: someFunc(my_value);
```

Here we have refactored `someFunc` to take an argument, making the code easier to follow.

If you need to list multiple dependencies and passing them as arguments is not an option, then the following pattern can be used:

```js
let one;
let two;
let three;

const someFunc = () => sideEffect();

$: one, two, three, someFunc();
```

This simple expression informs Svelte that it should rerun whenever `one`, `two`, or `three` changes. This expression runs the line of code whenever these dependencies change.

Similarly, if you only want to rerun a function when a variable changes _and_ is truthy then you can use the following pattern:

```js
let one;

const someFunc = () => sideEffect();

$: one && someFunc();
```

If the value of `one` is not truthy, then this short circuit expression stops at `one` and `someFunc` is not executed.

#### Variable deconstruction

Assigning a new value in a reactive declaration essentially turns these declarations into computed variables. However, deconstructing an object or array inside a reactive declaration may seem verbose or even impossible at first glance.

since `$: const { value } = object` is not valid javascript, we need to take a different approach. You can deconstruct values using reactive declarations by forcing javascript to interpret the statement as an expression.

To achieve this, all we need to do is wrap the declaration with parentheses:

```js
let some_obj = { one: 1, two: 2, three: 3 };

$: ({ one, two, three } = some_obj);
```

Javascript interprets this as an expression, deconstructing the value as expected with no unnecessary code required.

#### Hiding values from reactive declarations

On ocassion, you may wish to run a reactive declaration when only _some_ of its dependencies change, in essence, you need to _hide_ specific dependencies from Svelte. This is the opposite case of declaring additional dependencies and can be achieved by not referencing the dependencies in the reactive declaration but _hiding_ those references in a function.

```js
let one;
let two;

const some_func = (n) => console.log(n * two);

$: some_func(one);
```

The reactive declaration in this example reruns _only_ when `one` changes, we have hidden the reference to `two` inside a function because Svelte does not look inside of referenced functions to track dependencies.

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte Component Recipes

## Using Fetch to Consume APIs with Svelte

Working with external data in Svelte is important. Here's a guide.

_Maintainers of this Recipe: [swyx](https://twitter.com/swyx)_

### Fetching on Component Mount in Svelte

**Method 1: Using Lifecycles**

We can declare a `data` variable and use the `onMount` lifecycle to fetch on mount and display data in our component:

```html
<!-- https://svelte.dev/repl/99c18a89f05d4682baa83cb673135f05?version=3.20.1 -->
<script>
  import { onMount } from "svelte";
  let data;
  onMount(async () => {
    data = await fetch(
      "https://api.coindesk.com/v1/bpi/currentprice.json"
    ).then((x) => x.json());
  });
</script>

<pre>
  {JSON.stringify(data, null, 2)}
</pre>
```

You can further improve this implementation by showing a placeholder while `data` is undefined and also showing an error notification if an error occurs.

**Method 2: Using Await Blocks**

Since it is very common to update your app based on the status of your data fetching, Svelte offers convenient [await blocks](https://svelte.dev/docs#await) to help.

This example is exactly equal to Method 1 above:

```html
<!-- https://svelte.dev/repl/977486a651a34eb5bd9167f989ae3e71?version=3.20.1 -->
<script>
  let promise = fetch(
    "https://api.coindesk.com/v1/bpi/currentprice.json"
  ).then((x) => x.json());
</script>

{#await promise}
<!-- optionally show something while promise is pending -->
{:then data}
<!-- promise was fulfilled -->
<pre>
		{JSON.stringify(data, null, 2)}
	</pre
>
{:catch error}
<!-- optionally show something while promise was rejected -->
{/await}
```

Here you can see that it is very intuitive where to place your loading placeholder and error display.

Related Reading:

- https://svelte.dev/docs#2_Assignments_are_reactive
- https://svelte.dev/docs#onMount
- https://svelte.dev/docs#Attributes_and_props
- https://svelte.dev/docs#await

### Fetching on Button Click in Svelte

One flaw with the above approach is that it does not offer a way for the user to refetch data, and additionally we may not want to render on mount (for data saving or UX reasons).

**Method 1: Simple Click Handler**

If we don't want to immediately load data on component mount, we can wait for user interaction instead:

```html
<!-- https://svelte.dev/repl/2a8db7627c4744008203ecf12806eb1f?version=3.20.1 -->
<script>
  let data;
  const handleClick = async () => {
    data = await fetch(
      "https://api.coindesk.com/v1/bpi/currentprice.json"
    ).then((x) => x.json());
  };
</script>

<button on:click="{handleClick}">
  Click to Load Data
</button>
<pre>
  {JSON.stringify(data, null, 2)}
</pre>
```

The user now has an intuitive way to refresh their data.

However, there are some problems with this approach. You may still need to declare an extra variable to display error state. More subtly, when the user clicks for a refresh, the stale data still displays on screen, if you are not careful.

**Method 2: Await Blocks**

It would be better to make all these commonplace UI idioms declarative. Await blocks to the rescue again:

```html
<!-- https://svelte.dev/repl/98ec1a9a45af4d75ac5bbcb1b5bcb160?version=3.20.1 -->
<script>
  let promise;
  const handleClick = () => {
    promise = fetch(
      "https://api.coindesk.com/v1/bpi/currentprice.json"
    ).then((x) => x.json());
  };
</script>

<button on:click="{handleClick}">
  Click to Load Data
</button>

{#await promise}
<!-- optionally show something while promise is pending -->
{:then data}
<!-- promise was fulfilled -->
<pre>
    {JSON.stringify(data, null, 2)}
  </pre
>
{:catch error}
<!-- optionally show something while promise was rejected -->
{/await}
```

The trick here is we can simply reassign the `promise` to trigger a refetch, which then also clears the UI of stale data while fetching.

**Method 3: Promise Swapping**

Of course, it is up to you what UX you want - you may wish to keep displaying stale data and merely display a loading indicator instead while fetching the new data. Here's a possible solution using a second promise to execute the data fetching while the main promise stays onscreen:

```html
<!-- https://svelte.dev/repl/21e932515ab24a6fb7ab6d411cce2799?version=3.20.1 -->
<script>
  let promise1, promise2;
  const handleClick = () => {
    promise2 = new Promise((res) =>
      setTimeout(() => res(Math.random()), 1000)
    ).then((x) => {
      promise1 = promise2;
      return x;
    });
  };
</script>

<button on:click="{handleClick}">
  Click to Load Data {#await promise2}🌀{/await}
</button>

{#await promise1}
<!-- optionally show something while promise is pending -->
{:then value}
<!-- promise was fulfilled -->
<pre>
    {value}
  </pre
>
{:catch error}
<!-- optionally show something while promise was rejected -->
{/await}
```

### Dealing with CORS Errors in Svelte

Svelte is purely a frontend framework, so it will be subject to the same CORS restrictions that any frontend framework faces. You will run into CORS issues in two ways:

1. In local development (making requests from `http://localhost` to `https://myapi.com`)
2. In production (making requests from `https://mydomain.com` to `https://theirapi.com`)

You can solve both with a range of solutions from having a local API dev server or proxying requests through a serverless function or API gateway. None are responsibilities of Svelte but here are some helpful resources that may help:

- https://alligator.io/nodejs/solve-cors-once-and-for-all-netlify-dev/
- https://zeit.co/docs/v2/serverless-functions/introduction
- https://docs.begin.com/en/http-functions/api-reference
- https://aws.amazon.com/blogs/mobile/amplify-framework-local-mocking/

If you happen to be running [a Sapper app](https://sapper.svelte.dev/), then you may take advantage of preloading data server-side in Sapper: https://sapper.svelte.dev/docs#Preloading.

You now have all you need to work with Data APIs in Svelte.

### Further Links

- Svelte Suspense discussion: https://github.com/sveltejs/svelte/issues/1736
- Your link here?

## Form Validation with Svelte

_to be written_

## Client-Side Storage with Svelte

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte Store Recipes

## Stores

> _This guide assumes you understand the basics of Svelte Stores. If you aren't familiar with them then working through the [relevant tutorial](tutorial/writable-stores) and reading the [store documentation](docs#svelte_store) are highly recommended._

Svelte stores offer a simple mechanism to handle shared state in your Svelte application but looking beyond the built-in store implementations will unlock a whole world of power that you could never have dreamed of. In this episode of _The Tinest Kitchen_ we'll take a close look at [The Store Contract](#The_Store_Contract), learn how to implement [Custom Stores](#Custom_Stores), by making use of the built-in store API, and explore how we can implement [a completely custom store]() without using the built-in stores at all.

### The store contract

The built-in Svelte stores (`readable`, `writable`, and `derived`) are just store _implementations_ and while they are perfectly capable of handling many tasks, sometimes you need something more specific. Although often overlooked, the store _contract_ is what gives these stores their power and flexibility. Without this contract, svelte stores would be awkward to use and require significant amounts of boilerplate.

Svelte does not compile your javascript files and, as such, only observes the store contract inside Svelte components.

#### `store.subscribe`

At its simplest, the store contract is this: any time Svelte sees a variable prepended with `$` in a Svelte component (such as `$store`) it calls the `subscribe` method of that variable. The `subscribe` method must take a single argument, which is a function, and it must _return_ a function that allows any subscribers to unsubscribe when necessary. Whenever the callback function is called, it must be passed the current store value as an argument. The callback passed to subscribe should be called when subscribing and anytime the store value changes.

The following examples aren't the _exact_ code that Svelte produces, rather, simplified examples to illustrate the behaviour.

This:

```js
import { my_store } from "./store.js";

console.log($my_store);
```

Becomes something like this:

```js
import { my_store } from "./store.js";

let $my_store;
const unsubscribe = my_store.subscribe((value) => ($my_store = value));
onDestroy(unsubscribe);

console.log($my_store);
```

The callback function passed to `my_store.subscribe` is called immediately and whenever the store value changes. Here, Svelte has automatically produced some code to assign the `my_store` value to `$my_store` whenever it is called. If `$my_store` is referenced in the component, it also causes those parts of the component to update when the store value changes. When the component is destroyed, Svelte calls the unsubscribe function returned from `my_store.subscribe`.

#### `store.set`

Optionally, a store can have a `set` method. Whenever there is an assignment to a variable prepended with `$` in a Svelte component it calls the `set` method of that variable with newly mutated or reassigned `$variable` as an argument. Typically, this `set` argument should update the store value and call all subscribers, but this is not required. For example, Svelte's `tweened` and `spring` stores do not immediately update their values but rather schedule updates on every frame for as long as the animation lasts. If you decide to take this approach with `set`, we advise not [binding](tutorial/store-bindings) to these stores as the behaviour could be unpredictable.

This:

```js
$my_store = "Hello";
```

Will become something like:

```js
$my_store = "Hello";
my_store.set($my_store);
```

The same is true when assigning to a nested property of a store.

This:

```js
$my_store.greeting = "Hello";
```

Becomes:

```js
$my_store.greeting = "Hello";
my_store.set($my_store);
```

Although Svelte's built-in stores also have an `update` method, this is not part of the contract and is not required to benefit from the automatic subscriptions, unsubscriptions, and updates that the store contract provides. Stores can have as many additional methods as you like, allowing you to build powerful abstractions that take advantage of the automatic reactivity and cleanup that the store contract provides.

To summarise, the store contract states that svelte stores must be an object containing the following methods:

- `subscribe` - Automatically called whenever svelte sees a `$` prepended variable (like `$store`) ensuring that the `$` prepended value always has the current store value. Subscribe must accept a function which is called both immediately, and whenever the store value changes, it must return an unsubscribe function. The callback function must be passed the current store value as an argument whenever it is called.
- `set` - Automatically called whenever Svelte sees an assignment to a `$` prepended variable (like `$store = 'value'`). This should generally update the store value and call all subscribers.

### Custom stores

Now we know what Svelte needs to make use of the shorthand store syntax, we can get to work implementing a custom store by augmenting a svelte store and re-exporting it. Since Svelte doesn't care about additional methods being present on store objects, we are free to add whatever we like as long as `subscribe`, and optionally `set`, are present.

#### Linked stores

In this first example, we are creating a function that returns two linked stores that update when their partner changes, this example uses this linked store to convert temperatures from Celsius to Fahrenheit and vice-versa. The interface looks like this:

```js
store : { subscribe, set }
function(a_to_b_function, b_to_a_function): [store, store]
```

To implement this store, we need to create two writable stores, write custom `set` methods for each, and return an array of store objects containing this `set` method.

We define a function first as this implementation is a store _creator_ allowing us plenty of flexibility. The function needs to take two parameters, each being a callback function which is called when the stores are updated. The first function takes the first store value and returns a value that sets the value of the second store. The second argument does the opposite. One of these functions is called when the relevant `set` method is called.

```js
import { writable } from "svelte/store";

function synced(a_to_b, b_to_a) {
  const a = writable();
  const b = writable();
}
```

The `set` methods call their own `set` with the provided value and call the partner store's `set` when the provided value is passed through the callback function.

```js
// called when store_a.set is called or its binding reruns
function a_set($a) {
  a.set($a);
  b.set(a_to_b($a));
}

// called when store_b.set is called or its binding reruns
function b_set($b) {
  b.set(b_to_a($b));
  a.set($b);
}
```

All we need to do now is return an array of objects each containing the correct `subscribe` and `set` method:

```js
return [
  { subscribe: a.subscribe, set: a_set },
  { subscribe: b.subscribe, set: b_set },
];
```

Inside a component, we can use this synced store creator by deconstructing the returned array. This ensures Svelte can subscribe to each store individually, as stores definitions need to be at the top level for this to happen. This store can be imported and reused in any component.

```js
import { synced } from "./synced.js";

const [a, a_plus_five] = synced(
  (a) => a + 5,
  (b) => a - 5
);

$c = 0; // set an initial value
```

Since we have written custom `set` methods, we are also free to bind to each individual store. When one store updates, the other also updates after the provided function is applied to the value.

See it in action below. The following example uses the `synced` store to convert between Celsius and Fahrenheit in both directions.

_the stuff i actually wrote ends here, this is a fun example that could be included though_

### a custom implementation of the builtin store

A simple store is about 20 lines of code, in many cases the built-in stores provide good primitives you can build on but sometimes it makes sense to write your own.

The most basic implementation would look something like this ([REPL](https://svelte.dev/repl/1c055b975b6d42f5b8623bad5d92e8fc?version=3.14.0)) (this is simpler than the built-in stores):

```js
function writable(init) {
  let _val = init;
  const subs = [];

  const subscribe = (cb) => {
    subs.push(cb);
    cb(_val);

    return () => {
      const index = subs.findIndex((fn) => fn === cb);
      subs.splice(index, 1);
    };
  };

  const set = (v) => {
    _val = v;
    subs.forEach((fn) => fn(_val));
  };

  const update = (fn) => set(fn(_val));

  return { subscribe, set, update };
}
```

From this point you could add whatever functionality you wanted.

Edit: Probably worth mentioning that this is a full writable implementation, only the subscribe method and its return value (an unsubscribe function) are required to be a valid store.

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte Action Recipes

## Creating Custom Scroll Actions with Svelte

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte Transition Recipes

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte App-Level Design Patterns

## Routing with Svelte

_to be written_

- A map for an app: what is routing
- Different approaches
  - XML-stylee: [svelte-routing](https://github.com/EmilTholin/svelte-routing)
  - Express-stylee: [navaid](https://github.com/lukeed/navaid)
  - FS based routing: [routify](https://routify.dev/)

### Further Links

- https://routify.dev/ (with https://github.com/sveltech/routify-starter)
- https://github.com/EmilTholin/svelte-routing

## Authentication with Svelte

_to be written_

## Server Side Rendering

_some content_

- What is it
- What does the API look like
- building an SSR component
- hydrating an SSR cmponent with a client build
- building a simple express-based SSR server thing
- putting it all together

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Svelte Performance Tips

## Avoiding Memory Leaks with Svelte

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Testing and Debugging Svelte

## Unit Testing Svelte Components

_Some assumptions here, thos was pulled from something i wrote and the context was different. Edits will be required._

When testing svelte components in a node environment and using them outside of a Svelte application, we will be mostly interacting programmatically with Svelte’s client-side API as well as the helpers found in @testing-library/svelte.

### Creating a component

Creating a component is as simple as passing the component constructor, and any initial props, to the provided render function.

```js
import { render } from "@testing-library/svelte";
import Button from "../src/Button.svelte";

test("should render", () => {
  const results = render(Button, { props: { label: "a button" } });

  expect(() => results.getByLabelText("a button")).not.toThrow();
});
```

`results` is an object containing a series of methods that can be used to query the rendered component in a variety of ways. You can see a list of all query methods in the testing-library documentation [link].

### Changing component props

A components props can be changed or set by calling the .\$set method of the component itself. results also has a component key which contains the component instance giving us access to svelte’s full client-side API.

Svelte schedules all component updates for completion asynchronously in the next microtask. Awaiting the action that triggers that microtask will ensure that the component has been updated before proceeding through the code. This also applies to any Svelte component update, regardless of whether it is programmatically changed (via component.\$set) or via a ‘user’ action (such as clicking a button). For this to work we need to make sure that we make the test’s callback function async, so we can use await in the body.

```js
import { render } from "@testing-library/svelte";
import Button from "../src/Button.svelte";

test("should render", async () => {
  const { getByLabelText, component } = render(Button, {
    props: { label: "a button" },
  });

  await component.$set({ label: "another button" });
  expect(() => results.getByLabelText("another button")).not.toThrow();
});
```

### Testing component events

Component events have a different API to props and it is important we take the time to test them correctly. We will use a combination of jest mock functions and the svelte event API to make sure our component event is calling the provided handler. We can provide an event handler to a component event via a component's .$on method. The .$on method returns and off method, if we need to remove the listener.

```js
import { render, fireEvent } from "@testing-library/svelte";
import Button from "../src/Button.svelte";

test("events should work", () => {
  const { getByLabelText, component } = render(Button, {
    props: { label: "a button" },
  });

  const mock = Jest.fn();
  const button = getByLabelText("a button");

  component.$on("submit", mock);
  fireEvent.click(button);

  expect(mock).toHaveBeenCalled();
});
```

### Testing slots

Slots are more difficult to test as there is no programmatic interface for working with them either inside or outside of a Svelte component. The simplest way is to create a test specific component that utilises a dynamic component to and passes in a default slot to that component which can be asserted against. In this example the component passed as a prop will be used as the containing or parent component of the slotted child content.

```svelte
<script>
  export let Component;
</script>

<svelte:component this={Component}>
  <h1 data-testid="slot">Test Data</h1>
</svelte:component>
```

Then the component you wish to test can be passed to the constructor as a prop in order to mount the component correctly:

```js
import { render } from "@testing-library/svelte";

import SlotTest from "./SlotTest.svelte";
import ComponentToBeTested from "./ComponentToBeTested.svelte";

test("it should render slotted content", () => {
  const { getByTestId } = render(SlotTest, {
    props: { Component: ComponentToBeTested },
  });

  expect(getByTestId("slot")).not.toThrow();
});
```

### Testing the context API

As with slots, there is no programmatic interface for the Context API (setContext, getContext). If you are testing a component, in isolation, that would normally have a parent setting a context to be read by a child, then the simplest solution is to use a test specific parent. This is similar to the approach we used when testing slots. A test component might look something like this.

```svelte
<script>
  import { setContext } from "svelte";

  export let Component;
  export let context_key;
  export let context_value;

  setContext(key, value);
</script>

<svelte:component this={Component} />
```

The component we wish to test looks something like this:

```svelte
<script>
  import { KEY } from './Parent.svelte';

  const ctx = getContext(KEY);
</script>

<button>{ctx.title}</button>
```

We can test this like so:

```js
import { render } from "@testing-library/svelte";

import ContextTest from "./ContextTest.svelte";
import ComponentToBeTested from "./ComponentToBeTested.svelte";
// Context is keyed, we need to get the actual key to ensure the correct context is selected by the child.
import { KEY } from "./OriginalParentComponent.svelte";

test("it should render slotted content", () => {
  const { getByRole } = render(ContextTest, {
    props: {
      Component: ComponentToBeTested,
      context_key: KEY,
      context_value: { title: "Hello" },
    },
  });
  const button = getByRole("button");

  expect(button.innerHTML).toBe("Hello");
});
```

_to be written_

## Debugging Svelte Apps in VS Code

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Publishing Svelte Components/Deploying Svelte Apps

## Packaging Svelte Components for npm

_to be written_

## Dockerize a Svelte App

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

# Special Usecase Walkthroughs

Here we simply translate recipes from other framework recipe collections to Svelte, if they don't fit neatly anywhere. So you can compare and onboard easily.

## Editable SVG Icon Systems with Svelte

_to be written_

## Practical use of Svelte slots with Google Maps

_to be written_

## Create a CMS-Powered Blog with Svelte

_to be written_

[Jump back up to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
