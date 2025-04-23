# tl;dr
Explain knowledge about:
- `setup` function
- Async component
- Suspense
# Suspense
`<Suspense>` is a built-in component for orchestrating async dependencies in a component tree.
- Ref: https://vuejs.org/guide/built-ins/suspense#suspense
There are two ways to use `Suspense`
- Top-level await
- Async Component
## Top-level await
First, we will discover what is `setup` function in Vue component?
### `setup` function
The `setup()` hook serves as the entry point for  [Composition API](https://vuejs.org/api/composition-api-setup.html) usage in components. For example:
```vue
<script>
import { ref } from 'vue'

export default {
  setup() {
    const count = ref(0)

    // expose to template and other options API hooks
    return {
      count
    }
  },

  mounted() {
    console.log(this.count) // 0
  }
}
</script>

<template>
  <button @click="count++">{{ count }}</button>
</template>
```
In the example above, you can define `count` which is a reactive and return it. In fact, you can define `computed`, `methods` inside this hook. `setup` runs before the instance creation i.e. before the [`onBeforeCreated`](https://vuejs.org/api/options-lifecycle.html#beforecreate) and [`onCreated`](https://vuejs.org/api/options-lifecycle.html#created) hook.

> Note: You can use \<script setup\> when working with Composition API and Single-File Components (SFCs)
### What is top-level await?
Briefly, you use `await` inside setup function:
```vue
<script>
import { ref } from 'vue';
	async setup () {
		const books = ref(null);
		const result = await fetch('api/v2/books');
		books.value = await result.json();
		return { books };
	}
</script>
```
or with await inside `script setup` (top-level await):
```vue
<script setup>
const res = await fetch(...)
const posts = await res.json()
</script>

<template>
  {{ posts }}
</template>
```


## Async Component
### What is async component?
**Async component** is a component whose definition is loaded on demand (lazily) rather than being included in the initial JavaScript bundle of your application
### What are the problems?
- Large Initial Bundle Size
- Slow Initial Load Time
- Wasted Resources
### Example
```javascript
import { defineAsyncComponent } from 'vue';

// Basic async component
const AsyncComp = defineAsyncComponent(() => import('./components/MyComponent.vue'));

// Usage in another component's <script setup> or options API
// <template>
//   <AsyncComp />
// </template>
```
or with [Vue router](https://router.vuejs.org/):
```javascript
// router/index.js
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
  {
    path: '/profile',
    // Route-level code-splitting
    component: () => import('../views/UserProfile.vue') // Directly uses dynamic import
  },
  // ... other routes
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});

export default router;
```
## Combine with Suspense
Both of them make requests to fetch data, `Suspense` can be used to handle the loading, successful, error state with no jarring user experience
- Handle nested components which each of them makes its own API calls can be challenged. The loading skeleton may look glitching.

For example, Suspense is used for show `loading` state:
```vue
<template>
	<Suspense>
		<template #default> // what is template default
			<AsyncComponent />
		</template>

		<template #fallback>
			<icon loading>
		</template>
	</Suspense>
</template>
```
> Note: If you don't wrap top-level await with `Suspense`, browser can not work correctly. In fact, it will throw a warning.

### Error handling
Use lifecycle hook `onErrorCaptured` to capture the error and handle async errors inside parent component:
```vue
<script setup>
import { onErrorCaptured } from 'vue'

const error = ref(null)
onErrorCaptured(() => {
	error.value = 'Sorry, something went wrong.'
})
</script>

<template>
	<Suspense>
		<template #default>
			<MyAsyncComponent />
		</template>

		<template #fallback>
			<div v-if="error">{{error}}</div>
			<LoadingIndicator v-else />
		</template>
	</Suspense>
</template>
```
You can place the error rendering inside the `\#default` template. But IMO, when an error happens, it means the `Suspense` does not resolve so it will revert back to the `\#fallback` content.
# Bonus: Compare two ways of fetching network data
There are two ways you can fetch data:
- fetch with `onMounted` life cycle hook
- fetch with top-level await

| Feature      | Fetching with `onMounted`                                                                                                                                                                                                                                                                            | Fetching with top-level await                                                                                                                                                                                                                                                                            |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pros**     | - Component structure renders immediately (good for perceived performance sometimes)<br>- No `<Suspense>` wrapper needed. <br>- More granular control over _in-component_ loading states.                                                                                                            | - Data available for _initial_ render.<br>- Simplifies component template (loading/error handled by `<Suspense>`)<br>- Better coordination of multiple nested async dependencies.<br>- Often better/cleaner integration with Server-Side Rendering (SSR).<br>- Centralized fallback UI via `<Suspense>`. |
| **Cons**     | - Requires manual loading/error state management within the component (`v-if`s).<br>- Can cause layout shifts if loading state differs significantly from final state. <br>- Data is fetched _after_ initial render, potentially slightly later.<br>- Can be verbose if repeated in many components. | - Requires `<Suspense>` wrapper in the parent component.<br>- Initial display is delayed until data arrives (user sees fallback). <br>- Can feel slightly less immediate if fallback isn't engaging.<br>- Error handling might shift towards `<Suspense>` (`onErrorCaptured`) or error boundaries.       |
| **SSR**      | Can be complex; often requires specific framework patterns to fetch data server-side and hydrate correctly without re-fetching on client.                                                                                                                                                            | Generally integrates more naturally. Data is fetched on the server during render, `<Suspense>` handles hydration state.                                                                                                                                                                                  |
| **Use-case** | - simple<br>                                                                                                                                                                                                                                                                                         | - initial display<br>- loading state<br>- SSR                                                                                                                                                                                                                                                            |
