# Svelte Language Fundamentals

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents**

- [Reactivity](#reactivity)
- [Looping](#looping)
- ["Scoped Global" CSS](#scoped-global-css)
- [Passing Values from JS to CSS Variables](#passing-values-from-js-to-css-variables)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Reactivity

### Reactive assignments

The reactivity system introduced in Svelte 3 has made it easier than ever to trigger updates to the DOM. Despite this, there are a few simple rules that you must always follow. This guide explains how Svelte's reactivity system works, what you can and can't do, as well a few pitfalls to avoid.

#### Top-level variables

The simplest way to make your Svelte components reactive is by using an assignment operator. Any time Svelte sees an assignment to a _top-level variable_ an update is scheduled. A 'top-level variable' is any variable that is defined inside the script element but is not a child of _anything_, meaning, it is not inside a function or a block. Incidentally, these are also the only variables that you can reference in the DOM. Let's look at some examples.

The following works as expected and update the dom:

```svelte
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

> From **Svelte 3.23.1** onwards the following issue no longer applies. You can now assign to array item primitives and the changes will be reflected in the original array. See this [REPL](https://svelte.dev/repl/bc170b644f554eb29374138167dea4f0?version=3.23.1) running on **Svelte 3.23.1**, where the issue has been fixed. And this [REPL](https://svelte.dev/repl/bc170b644f554eb29374138167dea4f0?version=3.23.0) running on **Svelte 3.23.0** where the issue still applies. What follows is for archival purposes for anyone on **Svelte 3.23.0** and below. For further context see Svelte issue [4744](https://github.com/sveltejs/svelte/issues/4744).

The only exception to the top-level variable rule is when you are inside an `each` block. Any assignments to variables inside an `each` block trigger an update. Only assignments to array items that are objects or arrays result in the array itself updating. If the array items are primitives, the change is not traced back to the original array. This is because Svelte only reassigns the actual array items and primitives are passed by value in javascript, not by reference.

The following example causes the array and, subsequently, the DOM to be updated:

```svelte
<script>
  let list = [{ n: 1 }, { n: 2 }, { n: 3 }];
</script>

{#each list as item}
  <button on:click={() => item.n *= 2 }>{ item.n }</button>
{/each}
```

This, however, will not:

```svelte
<script>
  let list = [1, 2, 3];
</script>

{#each list as item}
  <button on:click={() => item *= 2 }>{ item }</button>
{/each}
```

The easiest workaround is to just reference the array item by index from inside the each block:

```svelte
<script>
  let list = [1, 2, 3];
</script>

{#each list as item, index}
  <button on:click={() => list[index] *= 2 }>{ item }</button>
{/each}
```

#### Variables not values

Svelte only cares about which _variables_ are being reassigned, not the values to which those variables refer. If the variable you reassign is not defined at the top level, Svelte does not trigger an update, even if the value you _are_ updating updates the original variable's value as well.

This is the sort of problem you may run into when dealing with objects. Since objects are passed by reference and not value, you can refer to the same value in many different variables. Let's look at an example:

```svelte
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

```svelte
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

```svelte
<script>
  let n = 0;

  $: n_squared = n * n;
</script>

<button on:click={() => n += 1}>{ n_squared }</button>
```

Whenever Svelte sees a reactive declaration, it makes sure to execute any reactive statements that depend on one another in the correct order and only when their direct dependencies have changed. A 'direct dependency' is a variable that is referenced inside the reactive declaration itself. References to variables inside functions that a reactive declaration _calls_ are not considered dependencies.

```svelte
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

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

## Looping

`{#each}` block allows you to loop only **array** or **array-like object** (i.e. it has a `.length` property).

Here are some examples if you want to loop through data structures besides array.

### Looping a map

You can use spread operator `[...value]` for [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) to get an array of key value pairs.

```svelte
<script>
  const map = new Map([['.svelte', 'Svelte'], ['.js', 'JavaScript']]);
</script>

{#each [...map] as [key, value]}
	<div>
		{key}: {value}
  </div>
{/each}
```

Both [`Map.keys()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/keys) and [`Map.values()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map/values) method return an iterable. To use `{#each}` with iterable, you can use spread operator `[...value]` on the iterable.

```svelte
<script>
  const map = new Map([['.svelte', 'Svelte'], ['.js', 'JavaScript']]);
</script>

{#each [...map.keys()] as key}
	<div>
		{key}
  </div>
{/each}

{#each [...map.values()] as value}
	<div>
		{value}
  </div>
{/each}
```

### Looping a set

You can use spread operator `[...value]` for [Set](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set) to get an array of items.

```svelte
<script>
  const set = new Set(['.svelte', '.js']);
</script>

{#each [...set] as item}
	<div>
		{item}
	</div>
{/each}
```

### Looping a string

[String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) is considered an **array-like object** as it has `.length` property.

```svelte
<script>
  const string = 'Svelte';
</script>

{#each string as character}
	<div>
		{character}
	</div>
{/each}
```

### Looping a generator function

[Generator function `function*`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/function*) returns a [generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator) object, which conforms to both the [iterable protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterable_protocol) and the [iterator protocol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols#The_iterator_protocol).

To use `{#each}` with generator function, you can use spread operator `[...value]` on the generator.

```svelte
<script>
	function* generator() {
		yield '.svelte';
		yield '.js';
	}
</script>

{#each [...generator()] as item}
	<div>
		{item}
	</div>
{/each}
```

### Binding to spreaded item

Once you spread a Map, Set, Generator, or any Iterable, you are creating a new array, and therefore binding (`bind:`) with the item may not work anymore.

```svelte
<script>
  const map = new Map([['.svelte', 'Svelte'], ['.js', 'JavaScript']]);
</script>

{#each [...map] as [key, value]}
  <!-- You can't change the value of the input, nor the value in the map -->
  <input bind:value={value} />
{/each}
```

To workaround this, you can use `on:input` listener

```svelte
<script>
  const map = new Map([['.svelte', 'Svelte'], ['.js', 'JavaScript']]);
</script>

{#each [...map] as [key, value]}
  <input
    {value}
    on:input={(event) => {
      map.set(key, event.currentTarget.value);
      map = map;
    }}
  />
{/each}
```

## "Scoped Global" CSS

Sometimes your template code doesn't match your CSS. If you generate html via `{@html}` or have some unstyled elements inside child components, you might want to style it, however Svelte won't let you write CSS that doesn't exist in the templates. You might feel forced to use `:global()` to make the CSS work, but that would leak it out to the rest of your app. So instead you could try this trick:

```svelte
<div>
  <Paragraph />
  <Paragraph />
  <Paragraph />
</div>
<style>
  div :global(p + p) {
    margin-top: 1rem;
  }
</style>
```

Now that `p` styling will be output by Svelte, AND it won't leak out to the rest of your app.

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)

## Passing Values from JS to CSS Variables

You can easily read in CSS media query values into JS with [`window.matchMedia`](https://developer.mozilla.org/en/docs/Web/API/Window/matchMedia). However, sometimes you want to pass information from JS to CSS variables, or have CSS Variables read into JS.

To set CSS Variables on an element, you can use the `style` attribute.

```svelte
<!-- App.svelte -->
<script>
  import Child from './Child.svelte';
  let backgroundColor = 'blue';
  export let width = 30;
  export let height = 30;
</script>
<label>Color <input type="color" bind:value={backgroundColor} /></label>
<label>Width <input type="range" bind:value={width} /></label>
<label>Height<input type="range" bind:value={height} /></label>

<div style="
	--backgroundColor: {backgroundColor};
	--width: {width}px;
	--height: {height}px;
">
  <Child />
</div>

<!-- Child.svelte -->
<style>
  div {
    background-color: var(--backgroundColor);
		height: var(--height);
		width: var(--width);
  }
</style>
<div />
```

Alternatively, if you can have a custom action to do this. This is already available with [svelte-css-vars](https://github.com/kaisermann/svelte-css-vars).

```svelte
<!-- App.svelte -->
<script>
  import Circle from './Circle.svelte';
</script>

<Circle size="80x80" bg="url(https://placekitten.com/80/80) center" />

<Circle
  size={120}
  bg="radial-gradient(circle, #051937, #004d7a, #008793, #00bf72, #a8eb12) " />

<Circle
  size={180}
  bg="linear-gradient(45deg, #EE617D 25%, #3D6F8E 25%, #3D6F8E 50%, #EE617D 50%,
  #EE617D 75%, #3D6F8E 75%, #3D6F8E 100%) center / 100% 20px" />

<Circle size="600x200" bg="url(https://placekitten.com/250/200) center" />

<!-- Circle.svelte -->
<script>
  import cssVars from 'svelte-css-vars';

  export let bg = 'black';
  export let size = '50x50';

  $: [width, height = width] = size.toString().split(/[x|\/]/);
  $: styleVars = {
    width: `${width}px`,
    height: `${height}px`,
    bg,
  };
</script>

<style>
  div {
		display: inline-block;
    width: var(--width);
    height: var(--height);
    background: var(--bg);
    border-radius: 50%;
  }
</style>

<div use:cssVars={styleVars} />
```

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
