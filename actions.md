# Svelte Action Recipes

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Dynamic CSS in Svelte style tags](#dynamic-css-in-svelte-style-tags)
- [Creating Custom Scroll Actions with Svelte](#creating-custom-scroll-actions-with-svelte)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Dynamic CSS in Svelte style tags

In Svelte it is not currently possible to use variables in the `<style>` tags. This is a workaround to enable that using an action. It is simple and uses css custom properties (thus it is not something that is supported in Internet Explorer).

This is a very short and succint action that is just a couple of lines of code. Let's take a look:

```js
export function styles(node, styles) {
	setCustomProperties(node, styles)
	
	return {
		update(styles) {
			setCustomProperties(node, styles)
		}
	};
}

function setCustomProperties(node, styles) {
	Object.entries(styles).forEach(([key, value]) => {
		node.style.setProperty(`--${key}`, value)
	})
}
```

You pass in an object that contains your custom styles and it will create custom properties that are applied to the element in question. Here's how you would apply it to an element:

```svelte
<script>
	import { styles } from './styles.js'
	export let color = "pink";
</script>

<style>
	h1 {
		color: var(--color);
	}
</style>

<h1 use:styles={{ color: color }}>
	Child color is {color}
</h1>
```

This gets cascaded down to grand-children and even further down in the tree, which might or might not be what you want. It is, after all, just CSS. Keep this in mind if you want to use this method.

Applying a color from the parent is as simple as exposing it and use a regular old prop: `<Child color="blue" />`. [Here's a REPL to play around with.](https://svelte.dev/repl/154d60d78b8e47eab06519bd24e5eca0?version=3.23.2)

Inspired by [kaisermann's svelte-css-vars](https://github.com/kaisermann/svelte-css-vars)

Recipe maintainer [Svelte School](https://svelte.school) and [@kevmodrome](https://github.com/kevmodrome)

## Creating Custom Scroll Actions with Svelte

_to be written_

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
