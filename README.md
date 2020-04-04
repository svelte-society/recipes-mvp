# recipes-mvp

MVP repo to just start collecting recipes for [Svelte Community](https://github.com/sveltejs/community). The core inspiration is https://vuejs.org/v2/cookbook/.

All examples have some variation "with Svelte" appended to them in an attempt to optimize for searchability.

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Using Fetch to Consume APIs with Svelte](#using-fetch-to-consume-apis-with-svelte)
  - [Fetching on Component Mount in Svelte](#fetching-on-component-mount-in-svelte)
  - [Fetching on Button Click in Svelte](#fetching-on-button-click-in-svelte)
  - [Working with Data in Svelte](#working-with-data-in-svelte)
  - [Further Links](#further-links)
- [Authentication with Svelte](#authentication-with-svelte)
- [Form Validation with Svelte](#form-validation-with-svelte)
- [Editable SVG Icon Systems with Svelte](#editable-svg-icon-systems-with-svelte)
- [Create a CMS-Powered Blog with Svelte](#create-a-cms-powered-blog-with-svelte)
- [Unit Testing Svelte Components](#unit-testing-svelte-components)
- [Creating Custom Scroll Actions with Svelte](#creating-custom-scroll-actions-with-svelte)
- [Debugging Svelte Apps in VS Code](#debugging-svelte-apps-in-vs-code)
- [Avoiding Memory Leaks with Svelte](#avoiding-memory-leaks-with-svelte)
- [Client-Side Storage with Svelte](#client-side-storage-with-svelte)
- [Packaging Svelte Components for npm](#packaging-svelte-components-for-npm)
- [Dockerize a Svelte App](#dockerize-a-svelte-app)
- [Practical use of Svelte slots with Google Maps](#practical-use-of-svelte-slots-with-google-maps)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Using Fetch to Consume APIs with Svelte

Working with external data in Svelte is important. Here's a guide.

*Maintiners of this Recipe: [swyx](https://twitter.com/swyx)*

### Fetching on Component Mount in Svelte

**Method 1: Using Lifecycles**

We can declare a `data` variable and use the `onMount` lifecycle to fetch on mount and display data in our component:

```html
<!-- https://svelte.dev/repl/99c18a89f05d4682baa83cb673135f05?version=3.20.1 -->
<script>
  import { onMount } from 'svelte'
  let data
  onMount(async () => {
	data = await fetch('https://api.coindesk.com/v1/bpi/currentprice.json').then(x => x.json())
  })
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
let promise = fetch('https://api.coindesk.com/v1/bpi/currentprice.json')
	.then(x => x.json())
</script>

{#await promise}
	<!-- optionally show something while promise is pending -->
{:then data}
	<!-- promise was fulfilled -->
	<pre>
		{JSON.stringify(data, null, 2)}
	</pre>
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
  let data
  const handleClick = async () => {
    data = await fetch('https://api.coindesk.com/v1/bpi/currentprice.json')
	.then(x => x.json())
  }
</script>

<button on:click={handleClick}>
  Click to Load Data
</button>
<pre>
  {JSON.stringify(data, null, 2)}
</pre>
```

The user now has an intuitive way to refresh their data.

However, there are some problems with this approach. You may still need to declare an extra variable to display error state. More subtly, when the user clicks for a refresh, the stale data still displays on screen, if you are not careful.

**Method 2: Await Blocks**

It would be better to make all this commonplace UI idioms declarative. Await blocks to the rescue again:

```html
<!-- https://svelte.dev/repl/98ec1a9a45af4d75ac5bbcb1b5bcb160?version=3.20.1 -->
<script>
  let promise
  const handleClick = () => {
	promise = fetch('https://api.coindesk.com/v1/bpi/currentprice.json')
		.then(x => x.json())
  }
</script>

<button on:click={handleClick}>
  Click to Load Data
</button>

{#await promise}
  <!-- optionally show something while promise is pending -->
{:then data}
  <!-- promise was fulfilled -->
  <pre>
    {JSON.stringify(data, null, 2)}
  </pre>
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
  let promise1, promise2
  const handleClick = () => {
 	promise2 = new Promise(res => setTimeout(() => res(Math.random()), 1000)).then(x=> {promise1 = promise2; return x})
  }
</script>

<button on:click={handleClick}>
  Click to Load Data {#await promise2}🌀{/await}
</button>

{#await promise1}
  <!-- optionally show something while promise is pending -->
{:then value}
  <!-- promise was fulfilled -->
  <pre>
    {value}
  </pre>
{:catch error}
  <!-- optionally show something while promise was rejected -->
{/await}
```

You now have all you need to work with Data APIs in Svelte.

### Working with Data in Svelte

Once we have

### Further Links

- 

## Authentication with Svelte

*to be written*

## Form Validation with Svelte

*to be written*

## Editable SVG Icon Systems with Svelte

*to be written*

## Create a CMS-Powered Blog with Svelte

*to be written*

## Unit Testing Svelte Components

*to be written*

## Creating Custom Scroll Actions with Svelte

*to be written*

## Debugging Svelte Apps in VS Code

*to be written*

## Avoiding Memory Leaks with Svelte

*to be written*

## Client-Side Storage with Svelte

*to be written*

## Packaging Svelte Components for npm

*to be written*

## Dockerize a Svelte App

*to be written*

## Practical use of Svelte slots with Google Maps

*to be written*
