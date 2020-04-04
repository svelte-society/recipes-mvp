# recipes-mvp

MVP repo to just start collecting recipes. The core inspiration is https://vuejs.org/v2/cookbook/.

All examples have some variation "with Svelte" appended to them in an attempt to optimize for searchability.

## Table of Contents

<!-- START doctoc -->
<!-- END doctoc -->

## Authentication with Svelte

*to be written*

## Using Fetch to Consume APIs with Svelte

### Fetching on Component Mount in Svelte

We can declare a `data` variable and use the `onMount` lifecycle to fetch on mount and display data in our component:

```html
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

*Link to [Svelte REPL](https://svelte.dev/repl/99c18a89f05d4682baa83cb673135f05?version=3.20.1)*

### Working with Data in Svelte

### Related Reading and More Tips

- https://svelte.dev/docs#2_Assignments_are_reactive
- https://svelte.dev/docs#onMount
- https://svelte.dev/docs#Attributes_and_props


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
