# Svelte Form Recipes

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [General Form Recipes](#general-form-recipes)
- [Form Validation Recipes](#form-validation-recipes)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## General Form Recipes

Some general recipes when working with forms in Svelte.

### Bind Checkbox Groups

Instead of using `checked` and `on:change` to manage a group of radio buttons, use `bind:group`. (Credit: [Li Hau](https://twitter.com/lihautan/status/1275808445753516032))

![image](https://user-images.githubusercontent.com/6764957/85953244-89bd9e00-b9a1-11ea-802c-ec7574be4dcb.png)


You can bind any object as the value - not just strings:

- bind to an object: https://svelte.dev/repl/2b143322f242467fbf2b230baccc0484?version=3.23.2
- bind to an array: https://svelte.dev/repl/02eda4dbf10648888827e38800703575?version=3.23.2

### Prevent Window Close If Input Is Not Saved

```svelte
<script>
	let value = ''
	let savedValue = ''

	function unload(event) {
		if (savedValue === value) return;
		event.preventDefault();
		event.returnValue = 'dirty value';
		return 'dirty value';
	}
</script>
<input bind:value />

<button on:click={() => savedValue = value}>Save</button>
<svelte:window on:beforeunload={unload} />
```

REPL: https://svelte.dev/repl/00fd6d0df4bb4c6596d6f7ddb6d4b96a?version=3.23.2

## Form Validation Recipes

In this section we look at various methods to validate forms in Svelte

### Form Validation with Yup

[Yup](https://github.com/jquense/yup) is a JavaScript schema builder for value passing and validation. We can use Yup to help us validate forms in Svelte.

We can use `bind:value` to bind input value, and validate the form using `schema.validate(values)`.

```svelte
<script>
  // Define schema with Yup
  import * as yup from 'yup';
  const schema = yup.object().shape({
    email: yup
      .string()
      .required('Please provide your email')
      .email("Email doesn't look right"),
    password: yup.string().required('Password is required'),
  });

  let values = {};
  let errors = {};

  async function submitHandler() {
    try {
      // `abortEarly: false` to get all the errors
      await schema.validate(values, { abortEarly: false });
      alert(JSON.stringify(values, null, 2));
      errors = {};
    } catch (err) {
      errors = extractErrors(err);
    }
  }
  function extractErrors(err) {
    return err.inner.reduce((acc, err) => {
      return { ...acc, [err.path]: err.message };
    }, {});
  }
</script>

<form on:submit|preventDefault={submitHandler}>
  <div>
    <input
      type="text"
      name="email"
      bind:value={values.email}
      placeholder="Your email"
    />
    {#if errors.email}{errors.email}{/if}
  </div>
  <div>
    <input
      type="password"
      name="password"
      bind:value={values.password}
      placeholder="Password"
    />
    {#if errors.password}{errors.password}{/if}
  </div>
  <div>
    <button type="submit">Register</button>
  </div>
</form>
```
