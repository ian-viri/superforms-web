<script lang="ts">
	import Form from './Form.svelte'
  import Next from '$lib/Next.svelte'
  import Flowchart from './Flowchart.svelte'
	import SuperDebug from 'sveltekit-superforms/client/SuperDebug.svelte'
  import { concepts } from '$lib/navigation/sections'

	export let data;
</script>

# Events

<svelte:head><title>Events</title></svelte:head>

With `use:enhance` enabled, you get full control over the form submit process through a number of events.

## Event flowchart

<Flowchart />

> In a single-page application (SPA), the events are triggered as usual, but the validation result, usually returned from the server between `onSubmit` and `onResult`, is created on the client. You should use `onUpdate` for validating it. See [the SPA page](/concepts/spa) for details.

## Usage

```ts
const { form, enhance } = superForm(data.form, {
  onSubmit: ({ action, data, form, controller, cancel }) => void
  onResult: ({ result, formEl, cancel }) => void
  onUpdate: ({ form, cancel }) => void
  onUpdated: ({ form }) => void
  onError: (({ result, message }) => void) | 'apply'
})
```

### onSubmit

```ts
onSubmit: ({ action, data, form, controller, cancel }) => void;
```

The `onSubmit` event is the first one triggered, hooking you into SvelteKit's `use:enhance` function, also giving you a chance to cancel the submission altogether. See SvelteKit docs for the [SubmitFunction](https://kit.svelte.dev/docs/types#public-types-submitfunction) signature.

### onResult

```ts
onResult: ({ result, formEl, cancel }) => void
```

If the submission isn't cancelled, the form is posted to the server, which responds with a SvelteKit [ActionResult](https://kit.svelte.dev/docs/types#public-types-actionresult), triggering the `onResult` event.


`result` contains the ActionResult. You can modify it, changes will be applied further down the event chain. `formEl` is the `HTMLFormElement` of the form. `cancel()` is a function which will cancel the rest of the event chain and any form updates.

> If you just want to check if the the form validation succeeded, use [onUpdated](/concepts/events#onupdated) instead.

`onResult` is useful when you have set `applyAction = false` and still want to redirect, since the form doesn't do that automatically in that case. Then you can do:

```ts
const { form, enhance } = superForm(data.form, {
  applyAction: false,
  onResult({ result }) {
    if (result.type === 'redirect') {
      goto(result.location);
    }
  }
});
```

Also if `applyAction` is `false`, which means that `$page.status` won't update, you'll find the status code for the request in `result`.

### onUpdate

```ts
onUpdate: ({ form, formEl, cancel }) => void
```

If you don't care about the details of the `ActionResult`, rather the validation result, `onUpdate` is triggered just before the form update is being applied, and gives you the option to modify the validation result in `form`, or `cancel()` the update altogether. You have also access to the form's `HTMLFormElement` with `formEl`.

### onUpdated

```ts
onUpdated: ({ form }) => void
```

If you just want the default behaviour and do something after a valid update, like showing a toast notification, `onUpdated` is the easiest way.

`form` contains the validation result, and is read-only here, since the stores have updated at this point.

**Example:**

```ts
const { form, enhance } = superForm(data.form, {
  onUpdated({ form }) {
    if (form.valid) {
      // Successful post! Do some more client-side stuff.
    }
  }
});
```

### onError

```ts
onError: (({ result, message }) => void) | 'apply'
```

Finally, the `onError` event allows you to act on `ActionResult` errors. `result` is the ActionResult with the `error` property casted to [App.Error](https://kit.svelte.dev/docs/types#app-error), and the `message` parameter is the same as the `$message` store, so you can conveniently set it to the error value here.

```ts
const { form, enhance } = superForm(data.form, {
  onError({ result, message }) {
    message.set(result.error.message);
  }
});
```

You can also set `onError` to the string value "apply", in which case the default `applyAction` behaviour will be used, rendering the nearest `+error` boundary and wiping out the form, so be careful.

> If you're unsure what event to use, start with `onUpdated`. If your app is a [SPA](/concepts/spa), `onUpdate` is most likely the one you should be using to validate the form data.

<Next section={concepts} />
