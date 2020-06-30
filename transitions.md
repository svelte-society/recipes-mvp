# Svelte Transition Recipes

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [Draw svg paths](#draw-svg-paths)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

(to be completed)

https://dev.to/ekafyi/svelte-tutorials-learning-notes-transitions-2jd2

## Draw svg paths

To draw out the svg path as the svg enters the DOM, you can use the built-in `draw` transition.

```svelte
<script>
  import { draw } from 'svelte/transition';
  let visible = false;
</script>

<input type="checkbox" bind:checked={visible} />

{#if visible}
  <svg>
    <path transition:draw={{duration: 3000}} d="..." />
  </svg>
{/if}
```

[REPL](https://svelte.dev/repl/27a3259ac6a44d71a8cbf020e02f7cd0?version=3.23.2)

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
