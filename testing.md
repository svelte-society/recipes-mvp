# Testing and Debugging Svelte

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Table of Contents**

- [Unit Testing Svelte Components](#unit-testing-svelte-components)
- [Debugging Svelte Apps in VS Code](#debugging-svelte-apps-in-vs-code)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

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

Alternatively [svelte-htm](https://www.npmjs.com/package/svelte-htm) allows to use a [jsx](https://reactjs.org/docs/introducing-jsx.html)-like [syntax](https://www.npmjs.com/package/htm#syntax-like-jsx-but-also-lit) for specifying the test case:

```js
import { render, fireEvent } from "@testing-library/svelte";
import html from "svelte-htm";

import Button from "../src/Button.svelte";

test("events should work", () => {
  const mock = Jest.fn();
  const { getByLabelText, component } = render(
    html`<${Button} label="a button" on:submit=${mock}><//>`
  );

  const button = getByLabelText("a button");

  fireEvent.click(button);

  expect(mock).toHaveBeenCalled();
});
```

One step further goes [svelte-jsx](https://www.npmjs.com/package/svelte-jsx) which allows to use [jsx](https://reactjs.org/docs/introducing-jsx.html) for specifying the test case. This requires a working [babel setup](https://www.npmjs.com/package/svelte-jsx#babel-configuration).

> Please note that jsx does not allow `:` in attribute names. Those should be replace by `_`. In case of `on:eventname` this can additionally be written as `onEventname`.

```js
import { render, fireEvent } from "@testing-library/svelte";

import Button from "../src/Button.svelte";

test("events should work", () => {
  const mock = Jest.fn();
  const { getByLabelText, component } = render(
    <Button label="a button" onSubmit={mock}></Button>
  );

  const button = getByLabelText("a button");

  fireEvent.click(button);

  expect(mock).toHaveBeenCalled();
});
```

### Testing slots

Slots are more difficult to test as there is no programmatic interface for working with them either inside or outside of a Svelte component. The simplest way is to create a test specific component that utilizes a dynamic component to and passes in a default slot to that component which can be asserted against. In this example the component passed as a prop will be used as the containing or parent component of the slotted child content.

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

To simplify this case and provide the familiar svelte component syntax [svelte-htm](https://www.npmjs.com/package/svelte-htm) and [svelte-jsx](https://www.npmjs.com/package/svelte-jsx) both support slotted content. Here is an example using `svelte-htm`:

```js
import { render } from "@testing-library/svelte";
import html from "svelte-htm";

import ComponentToBeTested from "./ComponentToBeTested.svelte";

test("it should render slotted content", () => {
  const { getByTestId } = render(html`
    <${ComponentToBeTested} data-testid="slot">
      <h1 data-testid="slot">Test Data</h1>
    <//>
  `);

  expect(getByTestId("slot")).not.toThrow();
});
```

The next example uses `svelte-jsx`:

```js
import { render } from "@testing-library/svelte";

import ComponentToBeTested from "./ComponentToBeTested.svelte";

test("it should render slotted content", () => {
  const { getByTestId } = render(
    <ComponentToBeTested data-testid="slot">
      <h1 data-testid="slot">Test Data</h1>
    </ComponentToBeTested>
  );

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

[svelte-fragment-component](https://www.npmjs.com/package/svelte-fragment-component) provides some useful component lifecycle properties and a context property. In combination with [svelte-jsx](https://www.npmjs.com/package/svelte-jsx) ([svelte-htm](https://www.npmjs.com/package/svelte-htm) works similar) the example could be written as:

```js
import { render } from "@testing-library/svelte";
import Fragment from 'svelte-fragment-component';

import ComponentToBeTested from "./ComponentToBeTested.svelte";
// Context is keyed, we need to get the actual key to ensure the correct context is selected by the child.
import { KEY } from "./OriginalParentComponent.svelte";

test("it should render slotted content", () => {
  const { getByRole } = render(<Fragment context={{[KEY]: { title: "Hello" }}} />});
  const button = getByRole("button");

  expect(button.innerHTML).toBe("Hello");
});
```

This additionally allows to use a component within the slot content which has access to the context value.

### Testing the `bind:` directive

As with slots and context, there is no programmatic interface for the `bind:` directive. Using [svelte-htm](https://www.npmjs.com/package/svelte-htm) or [svelte-jsx](https://www.npmjs.com/package/svelte-jsx) it is possible to test components with data to flow from child to parent. To mimic this two-binding both libraries use a writeable store as property value. This is actually a feature of [svelte-hyperscript](https://www.npmjs.com/package/svelte-hyperscript) which is the underlying implementation.

Here is an example with `svelte-htm`:

```js
import { render, act } from "@testing-library/svelte";
import userEvent from "@testing-library/user-event";

import { writable, get } from "svelte/store";

import html from "svelte-htm";

test("write into an input", async () => {
  const text = writable();
  const { getByRole } = render(html`<input bind:value=${text} />`);

  const input = getByRole("textbox");

  await userEvent.type(input, "some text");
  expect(get(text)).toBe("some text");

  await act(() => text.set("another text"));
  expect(input).toHaveValue("another text");
});
```

And here is an example with `svelte-jsx`:

> Please note that jsx does not allow `:` in attribute names. Those should be replace by `_`.

```js
import { render, act } from '@testing-library/svelte';
import userEvent from '@testing-library/user-event';

import { writable, get } from 'svelte/store';

test('write into an input', async () => {
  const text = writable();
  const { getByRole } = render(<input bind_value={text}>);

  const input = getByRole('textbox');

  await userEvent.type(input, 'some text');
  expect(get(text)).toBe('some text');

  await act(() => text.set('another text'));
  expect(input).toHaveValue('another text');
})
```

## Testing the `use:` directive

Actions are functions that are called with an element. A test component might look something like this.

```svelte
<script>
  export let action
</script>

<h1 use:action data-testid="element">Test Data</h1>
```

Then the component you wish to test can be passed to the constructor as a prop in order to mount the component correctly:

```js
import { render } from "@testing-library/svelte";

import ActionTest from "./ActionTest.svelte";
import action from "./actionToBeTested";

test("it should render slotted content", () => {
  const { getByTestId } = render(ActionTest, {
    props: { action },
  });

  const element = getByTestId("element");

  // test the action effect
});
```

This test can be written without creating the additional `ActionTest` component using [svelte-jsx](https://www.npmjs.com/package/svelte-jsx) ([svelte-htm](https://www.npmjs.com/package/svelte-htm) works similar):

```js
import { render } from "@testing-library/svelte";

import action from "./actionToBeTested";

test("it should render slotted content", () => {
  const { getByTestId } = render(
    <h1 use_action={action} data-testid="element">
      Test Data
    </h1>
  );

  const element = getByTestId("element");

  // test the action effect
});
```

_to be written_

## Debugging Svelte Apps in VS Code

_to be written_

[Back to Table of Contents](https://github.com/svelte-society/recipes-mvp#table-of-contents)
