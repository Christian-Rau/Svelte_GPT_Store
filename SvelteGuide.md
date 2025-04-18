Great! I’ll prepare a clean and well-structured documentation file in plain text format, containing accurate and up-to-date SvelteKit 2.0 and Svelte 5 code examples, covering topics like project structure, routing, runes, state management, adapters, form actions, and more. This will serve as a foundational reference for your personal GPT coding assistant.

I’ll begin compiling and formatting the content now, and I’ll let you know as soon as your reference file is ready.

# SvelteKit 2.0 and Svelte 5 – Comprehensive Reference

## 1. Project Structure

A standard SvelteKit 2.0 project has a well-defined structure. Below is a typical layout:

```plaintext
my-project/
├ src/
│ ├ lib/
│ │ ├ server/         (server-only utilities and modules)
│ │ └ ...             (shared library components/utilities)
│ ├ params/          (custom route parameter matchers)
│ ├ routes/          (application routes, pages and layouts)
│ │ ├ +layout.svelte  (root layout for all pages)
│ │ ├ +page.svelte    (root index page)
│ │ ├ about/
│ │ │ └ +page.svelte  ("/about" page)
│ │ ├ blog/
│ │ │ ├ +page.svelte  ("/blog" index page)
│ │ │ └ [slug]/
│ │ │    └ +page.svelte (dynamic "/blog/[slug]" page)
│ │ └ ...
│ ├ app.html          (base HTML template for the app)
│ ├ error.html        (template for error fallback pages)
│ ├ hooks.client.js   (client-side hooks)
│ └ hooks.server.js   (server-side hooks)
├ static/             (static assets like images, fonts)
├ tests/              (test files)
├ package.json
├ svelte.config.js
├ tsconfig.json       (or jsconfig.json for JS projects)
└ vite.config.js      (Vite configuration)
```

**Key Directories/Files:**

- **src/**: Contains the application source code. Notably:
  - **src/routes/**: Your pages and API endpoints, organized by filesystem routing (see Routing below).
  - **src/lib/**: Reusable components and utilities. Anything here can be imported via the `$lib` alias. **src/lib/server/** is specifically for server-only code (only imported on the server side).
  - **src/params/**: Custom route param matchers (for advanced routing patterns).
  - **src/app.html**: The HTML template for the app. It includes special placeholders like `%sveltekit.head%` (for meta/tags), `%sveltekit.body%` (the rendered page content), etc. This file defines the basic HTML structure for server-rendered pages.
  - **src/error.html**: Template used when an unrecoverable error occurs and no route can handle it. Contains placeholders for error messages and status.
  - **src/hooks.server.js** and **src/hooks.client.js**: Define global hooks that run on every request (server) or navigation (client). Use these for tasks like authentication, logging, or custom request handling (see Hooks section).
- **static/**: Static files served as-is (accessible via `/` path). Good for images, favicons, etc., that don't need processing.
- **tests/**: Location for tests (if using testing libraries).
- **Configuration files**: **svelte.config.js** (SvelteKit configuration, including adapter setup), **vite.config.js** (for Vite customization), and potentially other configs like ESLint or Prettier if set up. **package.json** lists dependencies and scripts. TypeScript projects include **tsconfig.json**.

This structure separates concerns clearly: pages in `routes`, reusable code in `lib`, static assets in `static`, and config at the root. It helps maintain scalability as your app grows.

## 2. Routing (Basic and Advanced)

SvelteKit uses filesystem-based routing. The files and folders under `src/routes` determine the URL structure of the application.

### Basic Routing

- Each page is a Svelte component file named `+page.svelte` (and optionally a corresponding `+page.js`/`+page.ts` or `+page.server.js` for data loading, see Form Actions and Loading Data).
- Folders become path segments in the URL. For example, a file at `src/routes/about/+page.svelte` corresponds to the `/about` route.
- The root `src/routes/+page.svelte` (in the routes folder itself) is the homepage (`/` path). A `+layout.svelte` placed in any folder provides a layout that wraps all `+page.svelte` files in that folder and its subdirectories.
- **Dynamic Routes**: To create routes with variables, folder names or file names can be placed in square brackets. For example:
  - `src/routes/blog/[slug]/+page.svelte` will match URLs like `/blog/hello-world` or any `/blog/<slug>` and make the segment available as a route parameter (`slug`).
  - You can access route params in a page's load function (`params.slug`) or via the `$page` store or props (explained below).
- **Nested Routes and Layouts**: If a directory contains its own `+layout.svelte`, that layout wraps pages within that directory. Layouts can be nested multiple levels deep. The absence of a `+layout.svelte` means the page inherits the nearest parent layout (ultimately the root layout).
- By default, SvelteKit will automatically handle **404 Not Found** scenarios. If no file matches a URL, SvelteKit will render an error (you can customize with an `+error.svelte` in a route directory or at root for a global error page).

**Basic example: Defining a dynamic route and using its parameter**:

In `src/routes/products/[id]/+page.js` (server load for the product page):

```js
// +page.js (or +page.server.js if only running on server)
export function load({ params }) {
	const id = params.id;
	// Fetch or compute data for product with this id
	return { productId: id };
}
```

In `src/routes/products/[id]/+page.svelte`:

```svelte
<script>
	// In Svelte 5, page props (including load result) are accessed via $props
	let { productId } = $props();
</script>

<h1>Product {productId}</h1><p>Details about product ID {productId}...</p>
```

When the user navigates to `/products/42`, SvelteKit will load the data (providing `params.id === "42"` to the load function) and then render the `+page.svelte`, displaying "Product 42" as the heading.

### Advanced Routing

SvelteKit offers advanced routing features for complex use cases:

- **Rest Parameters** (`[...param]`): Capture an unknown number of segments. For example, a route file `src/routes/[...parts]/+page.svelte` will match any path with one or more segments (e.g. `/a`, `/a/b/c`) and provide `params.parts` as an array (`["a"]` or `["a","b","c"]` respectively). This is useful for catch-all routes or flexible URL structures. A rest route can even match nothing (e.g. `/a/z` matching `src/routes/a/[...parts]/z` with `parts` empty). Always validate `params` in this case if needed.
- **Optional Parameters** (`[[param]]`): Make a segment optional. For example, a directory named `[[lang]]` containing `+page.svelte` would match either `/en` (where `params.lang === "en"`) or `/` (with no lang param). Essentially, the segment may be present or not. This can be useful for optional locale codes or modes in the URL.
- **Route Groups** (`(group)`): Parentheses around a directory name group routes without affecting the URL path. Group directories are ignored in the URL structure but allow you to organize routes or apply specific layouts. For instance, you might have `(admin)` and `(site)` groups to separate admin routes from normal site routes, each with its own `+layout.svelte`. Example:
  ```plaintext
  src/routes/
  ├ (site)/
  │   ├ +layout.svelte   (layout for site section)
  │   └ about/
  │       └ +page.svelte ("/about", uses site layout)
  ├ (admin)/
  │   ├ +layout.svelte   (layout for admin section)
  │   └ dashboard/
  │       └ +page.svelte ("/dashboard", uses admin layout)
  └ +layout.svelte       (root layout for pages not in a more specific group)
  ```
  In this example, `/about` uses the site layout, `/dashboard` uses the admin layout, and any route outside those groups would use the root layout. The `(site)` and `(admin)` directory names do not appear in the URL.
- **Routing Priority and Matching**: SvelteKit determines which route file to use based on specificity. Static routes (without params) have higher priority than dynamic ones. Among dynamic routes, those with fewer or more constrained parameters outrank catch-alls. For example, `/about` (static) is checked before `/[slug]` (dynamic), and `/product/[id]/edit` (two segments, one dynamic) is checked before `/product/[...path]` (rest). You generally don't need to worry about this as long as your file structure is sensible; just know that the "most specific" match wins.
- **Encoding**: File and folder names represent URL segments. If you need special characters in a route (like an emoji or reserved characters), you can use URL encoding in the file name (e.g., `[x2e]` to represent a literal `.` in a folder name). This is rarely needed, but SvelteKit supports it for edge cases like `.well-known` routes.
- **Error Routes**: You can define an `+error.svelte` in a directory to handle errors for that segment of the app. For example, `src/routes/admin/+error.svelte` would render for errors thrown in `/admin/*` pages or in their load functions. A root `src/routes/+error.svelte` catches unhandled errors globally. (This is related to routing in that it fits into the file structure.)

## 3. State Management and Stores

State in a Svelte/SvelteKit app can live in components, in Svelte stores, or on the server. SvelteKit adds considerations because of server rendering and multiple users.

**Principles:**

- **Avoid Shared State on the Server:** Each HTTP request in SvelteKit runs in isolation. Never store request-specific data in a module scope variable on the server (as that would be shared across requests, leading to user data leaking between sessions). For example, if you create a writable store at the top level of a module (outside any function) and update it during a request, that change would affect all other requests using that module. To keep state per request, use things like `event.locals` (for data specific to a request) or ensure that any stateful logic runs in the `load` function or inside onMount on the client.
- **Use Stores or Context for Client State:** For browser state (like UI state, form inputs, etc.), Svelte's reactive stores are a convenient solution. Svelte provides `writable`, `readable`, and `derived` store utilities (from `svelte/store`). A store is an object that holds a value and allows subscription. Components can subscribe to stores by prefixing them with `$` in the template, which auto-subscribes and updates on changes.
- **No Side-Effects in Load Functions:** The `load` function (in `+page.js/+page.server.js`) should remain pure (no direct side-effects like writing to a database or modifying global state). It should only fetch and return data. This ensures that whether it's called on server or client, it produces consistent results and doesn't accidentally run unintended actions multiple times. Side-effects (like sending analytics, writing files, etc.) should be done in endpoints or form actions, not in load.
- **Preserving State Across Navigations:** When using client-side routing (navigating via links in an SPA fashion), component state is preserved by default if the same component remains in the DOM. However, if you navigate to a different page and then back, by default the component will remount fresh (losing internal state). If you need to preserve state between navigations (for example, remember scroll position or form data), consider storing that state in a store or in the URL (so that going back will restore it) or use SvelteKit **snapshots**. Snapshots can capture the state of a page when navigating away and restore it if you navigate back, which is useful for transient UI state that shouldn't reset on back navigation.
- **Storing State in the URL:** When appropriate, encode state in the URL (route params or query params). This makes the state shareable/bookmarkable and ensures that a page can restore its state from the URL alone. For example, current page number, search query, or selected filters can be reflected in the query string so that reloading the page or sharing the link preserves that state.
- **Context API for Deep State Sharing:** Svelte's context API (`setContext` and `getContext`) allows passing state or stores down the component tree without prop drilling. This is useful for providing global or semi-global state (like a theme setting or a user object) to many components. Use context rather than relying on global variables, especially to avoid cross-request issues on the server (each request can set up its own context).

**Using Svelte Stores:**

Svelte stores are a core mechanism for state outside of local component variables. For example, to create a simple store:

```svelte
<!-- src/lib/stores.js -->
<script context="module">
	import { writable } from 'svelte/store';
	export const count = writable(0);
</script>
```

You can import and use this store in any component:

```svelte
<script>
	import { count } from '$lib/stores.js';

	function increment() {
		count.update((n) => n + 1);
	}
</script>

<button on:click={increment}>
	Count: {$count}
</button>
```

Here, `count` is a writable store initialized to `0`. Inside the `<button>`, we use `{$count}` to subscribe to the store’s value (the `$` prefix auto-unsubscribes when the component is destroyed). Clicking the button calls `increment()`, which uses `count.update` to increment the value. All components using `count` will update reactively.

**Avoiding server shared store issues:** If you put such a store in a module that is imported on the server (e.g., in a load function), that store instance will be created once at server start and reused. This is usually fine for constants or read-only data, but if you ever modify it on the server, it would affect all users. As a rule, only use writable stores for client state (or use them in a context that is unique per request). SvelteKit's server `event.locals` can store data for the duration of a request (not accessible in components, but useful in server logic). For passing data from server to client, use the load function return and page props.

**Context Example:** (More details in section 9, but in brief)

```svelte
<!-- Parent.svelte -->
<script>
	import { setContext } from 'svelte';
	import { writable } from 'svelte/store';
	const user = writable({ name: 'Alice' });
	setContext('user', user);
</script>

<Child />
```

```svelte
<!-- Child.svelte -->
<script>
	import { getContext } from 'svelte';
	const user = getContext('user');
</script>

<p>Logged in as {$user.name}</p>
```

In this example, the parent provides a writable store `user` via context. The child (or any descendant) can retrieve it and use it. Updates to `user` (e.g., `user.update(u => {...})`) would be reflected in all components using it. This approach prevents having to pass `user` as a prop through many layers and avoids global singletons that break server isolation.

## 4. Svelte 5 Runes ($state, $derived, $effect, etc.)

Svelte 5 introduces **runes**, which are special keywords (not imported functions) that give the Svelte compiler explicit instructions about reactivity and state. Runes replace Svelte 3/4’s implicit reactivity (like the `$:` reactive statements and auto-subscription to stores) with a clearer, more explicit system. They work in `.svelte` component files as well as in `.svelte.js`/`.svelte.ts` module files, meaning you can use them for state that is shared across components or in standalone logic.

The major runes are: **$state**, **$derived**, **$effect**, **$props**, **$bindable**, **$inspect**, and **$host**. Each serves a different purpose:

### $state

`$state(initialValue)` declares a **reactive state variable**. Think of it as a replacement for a normal `let` that you want to be reactive, or a replacement for a Svelte store when you only need the state in the current component or module. When a $state variable changes, the component updates (re-renders) any usage of that variable.

- You can use `$state` with primitive values, objects, or arrays. If you pass an object or array, Svelte will make it a **deeply reactive proxy** – meaning changes to its properties or elements (e.g. `obj.prop = 5` or `arr.push('x')`) are tracked and will trigger updates.
- For data types that can’t be proxied (like `Map` or `Set`), Svelte provides reactive equivalents in `svelte/reactivity`, or you can use `$state.raw` (see below).
- **Assignment vs mutation:** If you use $state with an object/array, you can mutate it directly (`stateObj.prop = ...`) and it will be reactive. If you use `$state.raw`, you opt out of deep reactivity and must reassign the variable for changes to be seen.

**Examples:**

```svelte
<script>
	// A simple reactive counter
	let count = $state(0);

	// An object that is deeply reactive
	let user = $state({ name: 'Anon', age: 30 });

	// A list using raw state (not deeply reactive)
	let items = $state.raw([1, 2, 3]);

	function addItem() {
		// Since items is raw, we reassign it to trigger an update
		items = [...items, items.length + 1];
	}
</script>

<p>Count: {count}</p>
<button on:click={() => count++}>Increment</button>

<p>User: {user.name} ({user.age} years old)</p>
<button on:click={() => user.age++}>Birthday (age becomes {user.age})</button>

<p>Items: {items.join(', ')}</p>
<button on:click={addItem}>Add Item</button>
```

**Explanation:** `count` is a $state, so `count++` (which is a direct mutation) will trigger the paragraph to update with the new count. `user` is a proxy – updating `user.age` triggers reactive updates just like reassigning would. `items` is declared with `$state.raw`, so pushing to it directly wouldn’t trigger an update. Instead, we reassign `items`to a new array (spread old items plus new one) in the`addItem`function to update the UI. Use`$state.raw` if you want to manage updates manually or for types that can’t be proxied. There’s also `$state.snapshot(value)` to create a non-reactive snapshot (useful if you want to freeze a reactive value at a moment in time).

### $derived

`$derived( expression )` creates a **derived reactive value** that automatically recalculates when its dependencies change. It’s similar to a computed property: the expression inside `$derived(...)` is re-evaluated whenever any $state or other reactive value it refers to updates.

- Use `$derived` when one value should always reflect some transformation of another value(s). For example, a `$derived` can keep a filtered list in sync with an original list, or compute a summary (like a total) from an array of items.
- The dependency tracking is done at runtime—when the component runs, Svelte monitors which reactive states are accessed inside the `$derived` expression, and those become its dependencies.
- `$derived.by(fn)` is an alternative that takes a function, useful for more complex logic (especially multi-line) within the derived calculation.
- You can temporarily override a derived value by assigning to it, which breaks the automatic link until the next time its dependencies change. This can be useful for implementing optimistic UI updates (e.g., setting a derived value by hand and then letting it revert when actual data arrives).

**Examples:**

```svelte
<script>
	let x = $state(5);
	let doubled = $derived(x * 2);

	let numbers = $state([1, 2, 3]);
	let sum = $derived.by(() => {
		// This function will re-run whenever `numbers` changes
		return numbers.reduce((a, b) => a + b, 0);
	});
</script>

<p>{x} doubled is {doubled}.</p>
<p>Numbers: {numbers.join(', ')} (sum = {sum})</p>

<button on:click={() => (x = x + 1)}>Increment x</button>
<button on:click={() => numbers.push(numbers.length + 1)}>Add Number</button>
```

**Explanation:** Here `doubled` is always `x * 2`. When `x` changes (via the button or any other means), `doubled` automatically recalculates. The second example uses `$derived.by` to calculate the sum of an array. Whenever an element in `numbers` changes or the array length changes, the derived function runs and updates `sum`. If you click "Add Number", a new number is pushed into `numbers` (since `numbers` is a reactive proxy, `push` is detected as a change), and `sum` updates accordingly.

### $effect

`$effect(() => { ... })` runs the provided function as a **side effect** whenever its dependencies change and on component mount (after first render). This replaces the older `$:` reactive statement for running arbitrary code in reaction to state changes.

- Use $effect for things like logging, manual DOM manipulations, interacting with browser APIs, or calling external functions whenever certain state changes.
- The effect function will run after the DOM is updated (post-reactive update), ensuring the UI is in sync with state when the effect runs (unless you use a special variant like $effect.pre).
- You should **not update reactive state inside the same $effect** (at least not without guard conditions), because that would trigger the effect to run again and could lead to infinite loops. If you find you want to derive a new state from existing state, use $derived instead. $effect should generally not produce values used in the UI, just perform side tasks.
- Variants: `$effect.pre(fn)` runs the effect function _before_ the DOM updates (useful if you need to measure the old DOM state), and `$effect.root(fn)` runs an effect that isn't tied to component lifecycle (it won't be cleaned up on destroy unless you manage it).
- The effect function can return nothing or a cleanup function. If a cleanup function is returned, it will run prior to the next effect invocation (and on component unmount) – similar to how `onDestroy` works, but tied to this effect.

**Example:**

```svelte
<script>
	let name = $state('World');

	// Get reference to a DOM element via bind:this for demonstration
	let heading;

	$effect(() => {
		console.log(`Hello, ${name}!`);
		// Example side-effect: update document title
		document.title = `Greetings to ${name}`;
	});
</script>

<h1 bind:this={heading}>Hello, {name}!</h1>
<input bind:value={name} placeholder="Enter your name" />
```

In this example, `$effect` logs a greeting and updates the page title whenever `name` changes (including initially when the component mounts). Typing in the input updates `name` (because of `bind:value`), which triggers the effect. If we had multiple dependencies, accessing them inside the effect body would register them as triggers. The effect runs only in the browser (not during server-side rendering), so `document.title` is safe here. If the component unmounted, any future effects would stop (and if we returned a cleanup function from $effect, it would run on unmount).

Another example (from a more graphical scenario):

```svelte
<script>
	let size = $state(50);
	let canvas;
	// Draw a square of side length `size` on a canvas whenever `size` changes
	$effect(() => {
		const ctx = canvas.getContext('2d');
		ctx.clearRect(0, 0, canvas.width, canvas.height);
		ctx.fillRect(0, 0, size, size);
	});
</script>

<input type="range" min="0" max="100" bind:value={size} />
<canvas bind:this={canvas} width="100" height="100"></canvas>
```

Here, adjusting the range input changes `size`, which triggers the effect to redraw a square on the canvas. The `$effect` runs after each change to `size` (and after the input’s value is updated in the DOM if that mattered). It also runs when the component first mounts (drawing the initial square of size 50). If the component is destroyed, Svelte will automatically clean up the effect.

### $props

`$props()` is used to **declare component props** (inputs). It replaces the older `export let ...` syntax from Svelte 3/4. By calling `$props()`, you get an object of all props passed into the component, which you typically destructure to individual variables.

- **Basic usage:** `let { propName } = $props();` will create a prop named `propName` that the parent component can provide. This is done inside a `<script>` at the top of your component.
- You can provide default values and even TypeScript types: e.g. `let { count: number = 0 } = $props();` declares a prop `count` of type `number` with a default value 0 if not provided by the parent.
- `$props()` can be called at most once in a component, and you must immediately destructure it (you can’t do anything else with the returned object directly).
- If your component can accept arbitrary additional props (spread attributes), you can capture them with the rest syntax `...`: e.g. `let { x, y = 0, ...rest } = $props();`. The `rest` object will contain any props other than x and y passed to the component. This is analogous to `$$restProps` in previous Svelte versions.
- Props defined with $props are **immutable by default** inside the child component (you shouldn’t reassign them). If you try to assign to a prop variable, Svelte will warn or error. This enforces one-way data flow unless you explicitly make a prop bindable (see $bindable).

**Example: Basic component props**

```svelte
<!-- Message.svelte -->
<script>
	let { text, emphasis = false } = $props();
</script>

<p>
	{#if emphasis}<strong>{text}</strong>{:else}{text}{/if}
</p>
```

A parent can use this component as `<Message text="Hello" />` or `<Message text="Look out!" emphasis={true} />`. If `emphasis` is not provided, it defaults to false.

In TypeScript:

```svelte
<script lang="ts">
	let { count: number = 0 } = $props();
	// 'count' is now a Number prop with default 0
</script>

<p>Count is {count}</p>
```

**Note:** In Svelte 5, `$props` is only for inside the component definition. When **passing** props from a parent, you still use HTML attributes syntax (e.g. `<MyComponent propName={value} />`). If a prop is not listed in the destructure, it will end up in the `...rest` (if used) or be ignored.

### $bindable

By default, component props are one-way: the parent passes a value in, and the child cannot directly modify it (and any attempt to bind to it in the parent will error). The `$bindable(initialValue)` rune allows a child component to mark a prop as **bindable**, meaning the parent can do two-way binding on it. Essentially, it opt-in to allow `<Child bind:someProp={parentVar} />` usage.

- You use $bindable when defining a prop that you want to allow two-way binding for. For example: `let { value = $bindable(0) } = $props();` in a child component means it has a prop `value` that is bindable, with initial default 0.
- When a parent uses `<Child bind:value={someVar} />`, Svelte will connect them: the parent’s `someVar` is passed in as `value`, and if the child changes `value` internally, the parent’s `someVar` will also update.
- Note that $bindable only allows parent to child binding if the parent explicitly uses the `bind:` syntax. If the parent just passes a normal value, it’s still one-way input.
- Bindable props still cannot be reassigned by the parent through normal prop updates (the parent should use `bind:` if it expects to get updates back).
- If a prop is bindable, the child can update it (e.g., in response to user input) without error. But if a parent doesn’t bind to it (just passes a literal or unbound value), those updates in the child will effectively be lost (and Svelte will likely warn that an update to an unbound prop was done).
- Ensure any bindable prop has a defined default or initial value (so that if the parent doesn’t supply one, the child has a starting point).

**Example: Two-way binding between parent and child**

_Child component (Counter.svelte):_

```svelte
<script>
	let { count = $bindable(0) } = $props();
</script>

<button on:click={() => count++}>
	Increment (count is {count})
</button>
```

_Parent component:_

```svelte
<script>
	let total = $state(5);
</script>

<Counter bind:count={total} /><p>Total in parent: {total}</p>
```

In this case, the `Counter` component has a bindable prop `count`. The parent binds its `total` state to the child’s `count`. When the button inside `Counter` increments `count`, that change flows out and updates `total` in the parent. The parent displays the updated total immediately. If the parent were to change `total` (not shown here), that would also update the child’s `count` prop.

Without using `$bindable`, trying to do `bind:count` would result in an error, and any internal `count++` in the child would also be disallowed. `$bindable` explicitly grants permission for two-way data flow for that prop.

### $inspect

`$inspect(variable)` is a development tool that helps debug reactive state. When used, it will log to the console whenever the inspected variable changes (or when it’s first initialized). Think of it as a built-in watcher that prints changes, so you don’t have to sprinkle `console.log` in reactive statements.

- `$inspect(x)` by default will log the variable `x` and its value whenever it updates. It will also label the log with either "init" (for the initial value) or "update".
- This only runs in development mode; in production builds, $inspect calls are ignored (so you don’t have to remove them).
- You can customize the log output by chaining `.with(callback)` after $inspect. For example: `$inspect(count).with((type, value) => console.log('count', type, value));`– here`type` would be "init" or "update".
- Another trick: `.with(debugger)` can be used to trigger a debugger breakpoint whenever the state changes (useful if you want to break in DevTools when some state becomes a certain value).

**Example:**

```svelte
<script>
	let count = $state(0);
	$inspect(count);
</script>

<button on:click={() => count++}>
	Count is {count}
</button>
```

When running the app in dev mode, this will log something like:

```
count (init): 0
count (update): 1
count (update): 2
...
```

each time the button is clicked. This helps trace that `count` is updating as expected.

For custom logging:

```svelte
<script>
	let user = $state({ loggedIn: false });
	$inspect(user).with((type, val) => {
		console.log(`[User state] ${type}:`, val);
	});
</script>
```

If `user.loggedIn` is changed somewhere, you might see:

```
[User state] init: { loggedIn: false }
[User state] update: { loggedIn: true }
```

This indicates when and how the `user` object changed.

Remember that $inspect is for debugging; you wouldn’t use it in production or for application logic. It’s safe to leave in code since it no-ops in prod, but typically you remove or disable it once done debugging.

### $host

`$host()` is a rune specifically for components that are compiled as **custom elements** (web components). It gives you access to the element instance itself (the DOM element that represents the component). This is only valid if your component is a custom element (declared with `<svelte:options tag="my-element" />` or similar). Outside of a custom element context, $host cannot be used.

- Typical use case: dispatching events from the custom element, or manipulating its attributes/properties directly.
- For example, if you want your custom element to fire an event upward, you might use `$host().dispatchEvent(new CustomEvent('someevent', {...}))`.
- Another use could be to imperatively control focus or style on the host element from within the component.

To use `$host`, your component must have `<svelte:options customElement="your-element-name" />` at the top. That tells Svelte to compile this component as a standalone web component.

**Example:**

```svelte
<svelte:options customElement="my-modal" />

<script>
	function openModal() {
		// perhaps do something then notify outside world
		$host().dispatchEvent(new CustomEvent('open'));
	}
</script>

<button on:click={openModal}>Open</button>
```

Here, `my-modal` is a custom element. When the button inside it is clicked, it dispatches a custom "open" event on the host element (which external code could listen for with `document.querySelector('my-modal').addEventListener('open', ...)`). You could also use $host for things like setting properties on the element itself: e.g., `$host().focus()`if you wanted the custom element to gain focus, or`$host().style.setProperty('--color', 'red')` to manipulate its own styles.

**Note:** `$host` is low-level and only for custom elements usage. In regular SvelteKit applications (which hydrate into normal DOM, not custom elements), you typically won’t use $host.

---

These runes allow fine-grained control of reactivity in Svelte 5. They may feel more verbose than the old `$:` reactive assignments or two-way binding of stores, but they make reactivity more explicit and robust, especially as apps grow. You can mix runes with Svelte stores and the context API as needed: for example, you might use a $state as a local state and also put it into a context or pass it to a store if you want to share it elsewhere. The key is that updates to $state and $derived will automatically propagate to anything that depends on them, and $effect will respond to those changes with side-effects.

## 5. Page Options (SSR, Prerender, etc.)

SvelteKit allows you to configure how individual pages (or groups of pages) are rendered and delivered. These **page options** are set by exporting constants from your `+page.js`, `+page.server.js`, or `+layout.js`/`.server.js` files. They control things like server-side rendering, static prerendering, trailing slashes, etc., on a per-route basis (with inheritance through layouts).

Key page options include:

- **SSR (Server-Side Rendering)**: By default, SvelteKit does SSR – meaning it generates HTML on the server for a page, and then hydrates it on the client. If you set `export const ssr = false` for a page or layout, that page will **not** be rendered on the server at all. Instead, it will be a purely client-side rendered page (essentially becoming an SPA for that route). Use this for sections of your app that don’t need SEO or first-load performance from SSR (like a highly interactive dashboard only for logged-in users). If SSR is false, the server will send a minimal HTML page and the Svelte app will bootstrap on the client to render it. (Note: if both `ssr` and `csr` are false, nothing will be rendered – an unlikely configuration except maybe for pages that do redirects or something.)
  - _Usage:_ In a `+page.js` or `+layout.js`:
    ```js
    export const ssr = false;
    ```
    You might apply this to an entire section by putting it in a `+layout.js` for that directory.
- **CSR (Client-Side Rendering)**: This controls client-side hydration. If you set `export const csr = false`, SvelteKit will not include the JavaScript for that page in the client bundle. The page will be rendered only on the server and delivered as static HTML. This is useful for truly static content pages (like a simple info page or a blog post) where you don’t need any client-side interactivity at all. The page will load faster and have no JS, but it also means navigation away from it will do a full page reload (since that page is not part of the SPA). It also means forms on that page cannot use SvelteKit’s enhanced features (they will do normal POSTs). Generally, use `csr=false` for very static pages or to reduce payload on pages where JS is unnecessary.
  - _Usage:_
    ```js
    export const csr = false;
    ```
    (Set in a page or layout file.)
  - **Important:** You would rarely disable CSR on pages that also disable SSR, because `ssr=false` + `csr=false` means a page that is neither server-rendered nor hydrated – which essentially produces an empty page. SvelteKit will warn if both are false. Typically, you use one or the other: SSR false (to do client-only SPA) or CSR false (to do server-only no-JS).
- **Prerender**: SvelteKit can prerender pages to static HTML at build time. `export const prerender = true` marks a page (or an entire section via layout) for prerendering. During the build, SvelteKit will generate an `.html` file for that page, so it can be served without needing a server at runtime (like a purely static site). This is great for content that doesn’t change per user and doesn’t need live server data.
  - If your whole app can be prerendered, you might use the `adapter-static` and turn the site into a collection of static files.
  - You can also use `prerender = 'auto'` which lets SvelteKit decide: it will prerender if possible (no unexpected side effects).
  - Setting `prerender = false` can override an inherited prerender setting, meaning “do not prerender this page” (for example, you mark the entire site to prerender in root layout, but then disable it for a specific dynamic page that cannot be prerendered).
  - _Usage:_
    ```js
    export const prerender = true; // or 'auto' or false
    ```
    Can be in a page or layout. A common pattern is to put `export const prerender = true` in your root layout, so all pages default to prerender, then explicitly turn it off for any page that requires server data at runtime or has form actions.
- **Trailing Slash**: This option controls whether URLs have a trailing slash. By default, SvelteKit normalizes URLs to **no trailing slash** (so navigating to `/about/` will redirect to `/about`). If you prefer to enforce a trailing slash or handle both, you can set:
  - `export const trailingSlash = 'never' | 'always' | 'ignore';`
  - `'never'` (default) removes trailing slashes.
  - `'always'` will add trailing slashes (and e.g. `/about` would redirect to `/about/`).
  - `'ignore'` will accept both, but note that treating `/x` and `/x/` as the same can have SEO downsides (they are technically different URLs).
  - This option can be exported from a layout to apply to a group of routes. For example, you might export it in the root layout to apply globally. Or set per group if needed.
- **Page Config**: There’s a general `config` option to pass adapter-specific configuration on a page-by-page basis. Most of the time, you won’t use this, but if an adapter (like Vercel or Cloudflare) offers special per-route configs (like edge runtime or region settings), you can export `config = { ... }`. The shape of this object depends on the adapter (for instance, Vercel’s adapter might allow `runtime: 'edge'`). See adapter docs for specifics. This is advanced usage for deployment fine-tuning.

**Example usage in a page** (inside `src/routes/example/+page.js`):

```js
export const ssr = true; // this page is server-side rendered (default)
export const csr = false; // do not include client JS (no hydration)
export const prerender = true; // generate a static HTML at build
```

This combination would make `example` page a fully static page (no JS, just HTML from build). If you navigate to it from a SvelteKit SPA, it will do a full page reload (because we disabled CSR), but if accessed directly it loads instantly as static content.

**Inheritance:** Page options set in a layout apply to all pages within that layout, unless overridden. For example, you could disable SSR for an entire section by exporting `ssr = false` in that section’s `+layout.js`. A child page could still override by setting its own `ssr` value. Similarly, prerender can be turned on in a parent layout and a child can opt out.

Use these options to optimize your app’s performance and capabilities:

- Prerender as much as possible for speed and lower server cost.
- Use SSR for pages that need dynamic data or SEO.
- Disable SSR for app-like sections to reduce server load or if they rely heavily on client-only features.
- Disable CSR for plain content pages to avoid shipping unnecessary JS.

## 6. Form Actions

SvelteKit provides a powerful way to handle form submissions through **actions**. Form actions enable server-side logic for forms while integrating smoothly with client-side navigation (progressive enhancement). Instead of writing separate API endpoints for forms, you can define actions alongside your page.

**Defining Actions:** In a page's server module (`+page.server.js`), export an `actions` object. Each key in this object corresponds to a named action, and the value is an async function that handles the form submission.

- The special key `default` represents the default action (used when the form does not specify a different action name).
- The function signature is typically `async ({ request, fetch, params, locals }) => { ... }`. You can use `request.formData()` to get the submitted form fields.
- The action function can return data that will be merged into the page’s `$page.data` (under a `form` key if using SvelteKit’s form conventions), or it can throw redirects or errors to control flow.

**Using Actions in a Form:** In your Svelte component (`+page.svelte`), create a form as usual. Set the `method` to POST (or GET if you want to handle GET submissions), and (optionally) the `action` attribute if using a named action.

- If `action` attribute is omitted or set to the page URL, SvelteKit will use the `default` action.
- If you set `action="?/name"` (the `?/` syntax with a name) on the form, it will target the action with that name in the `actions` object.
- On form submission, SvelteKit will call the corresponding action function on the server.
- With JavaScript enabled, SvelteKit intercepts the submit event and will handle it without a full page reload, updating the page data seamlessly (this is called _enhanced_ forms). Without JS (or if you force a full reload by some means), it will still work via traditional POST request (progressive enhancement ensures it gracefully degrades).

**Example: A simple contact form with an action**

_File: `src/routes/contact/+page.svelte` (Form UI)_

```svelte
<script>
	// You might import any validation logic or use data returned from the action here
	let errorMessage = null;
	export let data; // data from load or actions
</script>

<form method="POST">
	<label>
		Name: <input type="text" name="name" required />
	</label>
	<label>
		Message: <textarea name="message" required></textarea>
	</label>
	<button type="submit">Send</button>
</form>

{#if data?.form?.error}
	<p class="error">{data.form.error}</p>
{/if}
{#if data?.form?.success}
	<p class="success">Your message has been sent!</p>
{/if}
```

_File: `src/routes/contact/+page.server.js` (Actions handling)_

```js
export const actions = {
	default: async ({ request }) => {
		const formData = await request.formData();
		const name = formData.get('name');
		const message = formData.get('message');
		// Simple validation example
		if (!name || !message) {
			// Returning an object from an action will make it available in the page's data (`data.form` in this case)
			return { error: 'Name and message are required.' };
		}
		// Imagine sending the message or saving to DB here
		// ...

		// Indicate success
		return { success: true };
	}
};
```

**Explanation:** The form is standard HTML with `name` attributes. In the server action, we retrieve the fields. If validation fails (either field empty), we return an object with an `error` message. If it's successful, we return `{ success: true }`. On the client side, after submission:

- If JS is enabled, SvelteKit will call the action via `fetch` under the hood and merge the result into `data.form`. The page will update showing either the error or success message without a full reload.
- If JS is disabled, the form will do a normal POST. SvelteKit will still process the action on the server, then return the page, which will include the `data.form.error` or `data.form.success` in the HTML, so the user sees the message after the page reloads. The experience is seamless either way.
- Note the use of `data.form.error` in the component. SvelteKit convention is to put the return of the action in `data.form`. This happens automatically for the default action. (For named actions, it would be under `data.<actionName>`.)

**Named Actions:** If you have multiple form actions on one page (say, one form to create something and another to delete), you can give each an identifier:

```html
<form action="?/create" method="POST">...</form>
<form action="?/delete" method="POST">...</form>
```

And in `actions`:

```js
export const actions = {
	create: async ({ request }) => {
		/* handle create */
	},
	delete: async ({ request }) => {
		/* handle delete */
	}
};
```

This way, submitting the first form triggers the `create` action, and the second triggers the `delete` action. Each can return its own data. The results would be accessible as `data.create` or `data.delete` in the page. If you use the default action (no `action` attr or `action="?/default"` explicitly), it goes to `actions.default` and its result appears in `data.form` (for backward compatibility reasons, default uses the `form` property).

**Redirects and Errors:** Within an action, you can redirect by throwing a `redirect` (using `throw redirect(303, '/thank-you')` for example, after successfully processing a form). You can also throw an `error` with a status code to have the error handled by SvelteKit’s error handling (e.g., `throw error(400, 'Bad data')`). If you throw, the page’s navigation will reflect that (either going to a new route on redirect or showing an error). Use returns for normal responses that keep you on the page, and throws for redirecting or serious errors.

**Progressive Enhancement:** SvelteKit’s form actions are designed to work with or without JavaScript. If you want to further enhance forms (like showing loading states or handling certain responses differently on the client), you can use the `enhance` action from `$app/forms`. But for many cases, you get a lot for free just by defining actions as above.

## 7. Deployment with Adapters (Cloudflare and Others)

SvelteKit can run in a variety of environments thanks to **adapters**. An adapter is a plugin that knows how to build your SvelteKit app for a specific platform (Node.js, static sites, serverless functions, Cloudflare Workers, etc.). The adapter is configured in your `svelte.config.js`.

By default, new projects use `@sveltejs/adapter-auto`, which tries to detect your deployment target and use an appropriate adapter automatically (Node for local dev, and for example, Vercel or Netlify if it detects their environment). However, it’s often best to explicitly choose an adapter for clarity.

**Using the Cloudflare adapter (example):** If you want to deploy to Cloudflare (either Cloudflare Pages or Workers), you can use the official Cloudflare adapter.

1. **Install the adapter package:** In your project, run:

   ```bash
   npm install -D @sveltejs/adapter-cloudflare
   ```

   This adds the Cloudflare adapter as a dev dependency.

2. **Configure `svelte.config.js`:** Import and set the adapter:

   ```js
   // svelte.config.js
   import adapter from '@sveltejs/adapter-cloudflare';

   export default {
   	kit: {
   		adapter: adapter({
   			// options can go here if needed (usually none for Cloudflare)
   		})
   	}
   };
   ```

   This tells SvelteKit to use Cloudflare’s adapter when building. The adapter will produce a worker script (for Cloudflare Workers) or the necessary output for Cloudflare Pages depending on the context.

3. **Build and Deploy:** Now, when you run `npm run build`, SvelteKit will use the Cloudflare adapter to output your application in a format ready for Cloudflare. For Cloudflare Pages, typically you point it to the output (e.g., the `build` folder). For Cloudflare Workers, you might take the generated worker code and deploy it via the Wrangler CLI or via the Cloudflare dashboard.

The Cloudflare adapter knows how to deploy to both Pages and Workers (if using Cloudflare Pages, it will create a `_routes.json` for edge functions under the hood). If you use adapter-auto and deploy to Cloudflare Pages, it should pick this adapter automatically, but using it explicitly as above is fine.

**Other adapters:**

- **Node:** `@sveltejs/adapter-node` – for running a SvelteKit app as a Node.js server (for example, on your own VPS or in environments like AWS EC2, or Heroku). It outputs a Node server (an `index.js` you can run with Node).
- **Static:** `@sveltejs/adapter-static` – for building a completely static site (prerendering all pages). Use this for purely content-driven sites or if you want to host on something like GitHub Pages or any static file server. Remember to mark pages as `prerender = true` or use the entries option to tell it which dynamic routes to prerender.
- **Vercel:** `@sveltejs/adapter-vercel` – for deploying to Vercel (serverless functions and static as needed). Vercel’s adapter will split your app into functions according to their edge/serverless rules.
- **Netlify:** `@sveltejs/adapter-netlify` – similar to Vercel’s, but tailored for Netlify’s build process (Functions and Edge Functions).
- **Cloudflare Workers (standalone):** The Cloudflare adapter handles Workers. There is also `adapter-cloudflare-workers`, but it's essentially the same as using the cloudflare adapter (just more explicit).
- **Azure, AWS, etc.:** There are community adapters (e.g., `adapter-azure-swa` for Azure Static Web Apps, or unofficial ones for AWS). For most cases, the official adapters or adapter-node can cover your needs.

To use any adapter, the pattern is the same:

- Install it (`npm i -D @sveltejs/adapter-something`).
- Update `svelte.config.js` with `adapter: adapterSomething({...})`.
- Make sure any adapter-specific config (if needed) is provided (check the adapter docs, e.g., Netlify might allow specifying whether to use Edge functions).
- Build the project and follow deployment instructions of that platform (often just pushing to a git repo or using a CLI to deploy).

**Example `svelte.config.js` for a Node server:**

```js
import adapter from '@sveltejs/adapter-node';
export default {
	kit: {
		adapter: adapter({ out: 'build' })
	}
};
```

After building, you would run `node build` to start the server.

**Zero-config deployments:** Some platforms (like Vercel, Netlify, Cloudflare Pages) integrate with adapter-auto so that you might not even need to edit the config if adapter-auto detects them. But being explicit never hurts.

In summary, choose the adapter that matches your deployment target, configure it, and SvelteKit will handle producing the correct output. Adapters abstract away the differences in hosting environments so your app code remains the same.

## 8. Hooks and Lifecycle

This section covers two different kinds of "hooks":

- SvelteKit app hooks (functions that run during the request lifecycle on server or client, affecting all routes).
- Svelte component lifecycle functions (onMount, onDestroy, etc., which run during a component's existence in the browser).

### SvelteKit Hooks (Server and Client)

SvelteKit allows you to run global code for every request or navigation via hook functions defined in `src/hooks.server.js` and `src/hooks.client.js`. These hooks are not the same as React hooks; they are more like middleware or event handlers for the app.

**hooks.server.js** can export the following functions:

- `handle(event, resolve)`: The most important hook. It wraps the entire request processing. You can use it to modify the request or response. For example, you might check `event.cookies` for a session token and set `event.locals.user` based on it, or block/redirect certain requests. `event` contains details like `event.request`, `event.url`, `event.params`, `event.locals` (an object you can populate with data to pass to load functions or actions), etc. `resolve(event)` is a function that runs the next step (either the next handle if you composed multiple via `sequence`, or ultimately SvelteKit’s own router to produce a response).
- `handleError({ error, event })`: A hook to catch and handle errors thrown during load/data fetching or actions. You can use this to log errors or return a custom error response. For example, you might send errors to an error tracking service here. By returning nothing, SvelteKit will use the default error handling (which typically renders an +error.svelte page). You can also modify the error or event in this hook.
- `handleFetch({ request, event, fetch })`: This allows you to intercept outbound fetch calls that occur during load or actions. For instance, if in your load you call `fetch('/api/thing')`, the handleFetch hook can see that request and, say, add an authorization header or rewrite the request to an external API. You typically call `return fetch(request)` after adjusting it. This hook runs on both server and client when fetch is called from load or actions.

**hooks.client.js** can export:

- `handleError({ error, event })`: similar to the server one, but for errors happening during client-side navigation (after the app is loaded).
- `handleFetch(...)`: similar to server, but for fetches on the client side.

(There's no `handle` in hooks.client because client-side routing is already the resolved output of load functions; you can’t wrap the entire navigation the same way as on server. The server handle is where you often do auth checks and maybe redirect to login if needed — if you do that, the client never sees the protected page at all unless authorized.)

**Using handle (authentication example):**

```js
// hooks.server.js
export const handle = async ({ event, resolve }) => {
	// Example: attach a user object if session cookie is present
	const session = event.cookies.get('session_id');
	if (session) {
		// Suppose we have a function getUserBySession to validate cookie
		event.locals.user = await getUserBySession(session);
	} else {
		event.locals.user = null;
	}

	// Continue processing the request
	return resolve(event);
};
```

This snippet checks for a cookie named `session_id`. If present, it fetches a user and attaches it to `event.locals.user`. Now, any load function (on server) can read `event.locals.user` to know who is logged in. You might then use this, for example, to protect certain routes:

```js
// A more advanced handle with route protection:
export const handle = async ({ event, resolve }) => {
	const { pathname } = event.url;
	if (pathname.startsWith('/admin')) {
		// protect all /admin routes
		if (!event.locals.user) {
			// not logged in, redirect to login
			return Response.redirect('/login');
		} else if (!event.locals.user.isAdmin) {
			// logged in but not an admin, return 403
			return new Response('Not authorized', { status: 403 });
		}
	}
	return resolve(event);
};
```

This way, before any `/admin` page is loaded, the hook either redirects or blocks access if conditions aren't met. If the user is allowed, it calls `resolve(event)` which proceeds to actually render the page.

**Using handleError:**

```js
// hooks.server.js
export const handleError = ({ error, event }) => {
	console.error(`Error during ${event.request.method} ${event.url.pathname}:`, error);
	// Optionally, return { message: 'Internal Error' } or similar to customize error displayed
	// If nothing is returned, SvelteKit will use error message in +error.svelte if available.
};
```

This logs every error. In production, you might want to send this to a logging service.

**Using handleFetch:**

```js
// hooks.server.js
export const handleFetch = async ({ request, event, fetch }) => {
	// If the outgoing request is to our own API and we have a session token, attach it
	if (request.url.startsWith('https://api.myapp.com/')) {
		const token = event.cookies.get('token');
		if (token) {
			request = new Request(request, {
				headers: {
					...Object.fromEntries(request.headers),
					Authorization: `Bearer ${token}`
				}
			});
		}
	}
	return fetch(request);
};
```

This example checks if a fetch request is going to a certain API endpoint and adds an Authorization header from a cookie. This saves you from adding the header in every load function manually.

To summarize, SvelteKit hooks let you implement cross-cutting concerns (auth, logging, custom responses) globally.

### Component Lifecycle Hooks (Svelte Component Lifecycles)

Within any Svelte component (whether in SvelteKit or not), you have functions that run at specific times in the component’s life in the browser. These are imported from `svelte` (the core library) and called in the component’s script. The main ones are:

- `onMount`
- `onDestroy`
- `beforeUpdate` and `afterUpdate` (Note: in Svelte 5, `beforeUpdate`/`afterUpdate` are deprecated; $effect typically covers their use cases.)
- `tick` (not a hook per se, but a function to await next DOM update)

**onMount:** Runs after the component is first rendered in the DOM (on the client). It’s where you should put any code that needs to access the browser environment, do subscriptions, or start intervals/timeouts, etc. It only runs in the browser, never during SSR.

**onDestroy:** Runs just before the component is removed from the DOM (e.g., when you navigate away or if the parent component stops rendering it). Use it to clean up anything set up in onMount or elsewhere (remove event listeners, clear intervals, etc.).

**beforeUpdate / afterUpdate:** (In Svelte 3/4, these run before and after the DOM updates when reactive state changes. In Svelte 5, these are considered legacy; using $effect is preferred for reacting after updates. `afterUpdate` is somewhat like a $effect without dependencies specified, and can often be replaced by $effect. If you need beforeUpdate, consider logic adjustments as needed, or use $effect.pre to run before DOM update.)

**tick:** An async utility that returns a promise which resolves on the next microtask after the DOM update cycle. You `await tick()` when you need to wait for Svelte to flush pending state changes to the DOM. This is often used in cases where you programmatically change some state and then immediately need to measure or manipulate the updated DOM.

**Example:**

```svelte
<script>
	import { onMount, onDestroy, tick } from 'svelte';
	let timer;
	let count = 0;

	onMount(() => {
		// Start an interval when component mounts
		timer = setInterval(() => {
			count += 1;
			// count is $state or normal let? In Svelte 5, to be reactive, let's assume:
			// count = $state(0) above. Then count += 1 would trigger re-render each second.
		}, 1000);

		console.log('Component mounted');
		// Return a cleanup function directly from onMount is another way to handle onDestroy:
		return () => {
			clearInterval(timer);
			console.log('Component unmounted');
		};
	});

	// Alternatively, separate cleanup
	onDestroy(() => {
		clearInterval(timer);
	});

	async function scrollToBottom() {
		// assume this function is called after adding some content that affects scrollHeight
		await tick(); // wait for DOM update
		window.scrollTo(0, document.body.scrollHeight);
	}
</script>

<p>Seconds elapsed: {count}</p>
<button on:click={scrollToBottom}>Scroll to bottom after DOM update</button>
```

**Explanation:**

- `onMount(() => { ... })`: schedules the callback to run when the component is mounted on the page. We start a `setInterval` that increments `count` every second. We also log "Component mounted". If `onMount` returns a function, that function will be automatically treated as the destroy callback. In this case, we return a function that clears the interval and logs "Component unmounted". This means we don't even need a separate `onDestroy` – but we included one to show it's an alternative.
- `onDestroy(() => { ... })`: ensures the interval is cleared if the component goes away. (This is important to avoid memory leaks or actions continuing when component is gone.)
- `tick()`: inside `scrollToBottom`, after triggering some state change that would add content to the page, we await `tick()` to wait until the DOM reflects the updated state, then perform a scroll. Without tick, if we tried to measure or scroll immediately, the DOM might not yet include the new content.

**Note:** In Svelte 5, since reactive statements are replaced with runes, one typically uses $effect rather than afterUpdate. The above example could also use `$effect`to react to changes in`count` if needed, but for an interval, onMount is appropriate.

**Lifecycle summary:**

- Use `onMount` for code that should run once on initialization (or to kick off subscriptions/intervals).
- Use `onDestroy` for cleanup.
- Use `tick` when you need to wait for DOM changes.
- You can use many `onMount` or `onDestroy` calls in the same component if needed (they will all run).
- onMount's cleanup function or a separate onDestroy will run before component is destroyed or when you navigate away (if the component is a page).
- If a component is server-side rendered, `onMount` will run only when it actually mounts on client; any code in onMount that touches `window` or `document` is fine (it won’t run on server).
- Avoid heavy logic in onMount that could block the main thread; if needed, do it asynchronously.

## 9. Context, Props, and Bindings

These are about passing data around and linking data between components and the DOM.

### Context API (setContext and getContext)

The Svelte context API allows you to share data between components without using props, specifically from an ancestor to any deeply nested child, without manually passing it through every intermediate layer. This is useful for global-like data (current user, theme, router, etc.) or for a group of related components (e.g., a list and its items might share a context).

- **setContext(key, value):** call this in a parent component to provide a context value. `key` can be any unique identifier (string or Symbol often).
- **getContext(key):** call this in a child component to retrieve the value provided by an ancestor with that key. If no ancestor provided that context, `getContext` returns undefined.
- Context follows the component hierarchy (based on composition, not the URL routing or anything).
- Typically, you'll set context in a top-level layout or root component for app-wide context, or in a specific parent component for a contextual grouping.

**Example: Provide a theme context to child components:**

```svelte
<!-- App.svelte (root component, could be a layout) -->
<script>
	import { setContext } from 'svelte';
	// Provide a theme object to all descendants
	const theme = { color: 'purple', background: 'lavender' };
	setContext('theme', theme);
</script>

<main>
	<Header />
	<Content />
</main>
```

```svelte
<!-- Child component that wants to use the theme, e.g., Button.svelte -->
<script>
	import { getContext } from 'svelte';
	const theme = getContext('theme');
</script>

<button style="color: {theme.color}; background: {theme.background};"> I am themed </button>
```

Any component under `App.svelte` can call `getContext('theme')` and get that same `theme` object. This saves passing `theme` as a prop through potentially many layers.

The context is often used with stores. For instance, you could do `setContext('user', userStore)` and then any child can do `const user = getContext('user')` to get the store and use `$user`.

**When to use context:**

- When a value is truly global or cross-cutting and a lot of components need it (like a localization dictionary, or user info).
- When a specific subtree of components share something and it doesn't make sense to pass it through all intermediate components (which might not care about it at all).
- Avoid overusing context; props are simpler for straightforward parent-child communication. Context is best for avoiding prop drilling in deeply nested scenarios.

### Props (Component Properties)

Props are how a parent component passes data to a child component. In Svelte 5, as discussed, props are defined using the `$props()` rune inside the child component. From the parent's side, using props hasn't changed: you pass them as attributes on the component.

**Defining props in a component (child):**

```svelte
<script>
	let { title, count = 0 } = $props();
</script>

<h2>{title}: {count}</h2>
```

This component has two props: `title` (required, since no default given) and `count` (optional, defaults to 0).

**Passing props from parent:**

```svelte
<Counter title="Inbox" count={messages.length} />
```

Here `Counter` is the component above. We provide a string for `title` and a JS expression for `count`.

- You can pass any serializable or object data as props (objects, arrays, functions, etc.). Functions can be passed as callbacks too.
- If a prop is not provided by the parent and has no default, it will be `undefined` (which in TypeScript would violate a non-undefined type).
- Props in Svelte are always one-way by default: parent -> child. The child should not try to modify its props (they are essentially read-only inside the child, except if marked bindable).
- In SvelteKit, page components receive their `data` and `params` as props from the router. For instance, in a `+page.svelte`, doing `let { data, params } = $props();` would give you the data returned from load and route parameters. (In older SvelteKit, this was `export let data` etc.)

**Rest props:** If you use `...rest` in the destructuring, that allows your component to accept arbitrary additional HTML attributes. Those go into `rest` and you can then spread them on an element (often used when creating a wrapper component around a native element). For example:

```svelte
<script>
	let { value = '', ...rest } = $props();
</script>

<input bind:value {...rest} />
```

This Input component takes any extra props and passes them to the underlying `<input>` (so things like `placeholder`, `id`, `class` provided to the component will be forwarded to the actual input element).

### Bindings (bind:)

Binding in Svelte refers to two main things:

1. Binding between a DOM element and a component state (usually form inputs).
2. Binding between a child component’s prop and a parent component’s variable (two-way component bindings).

We discussed the second case under $bindable. Here we'll cover general binding usage.

**Element bindings:** Svelte can bind certain properties of form elements (and some other elements) directly to a component variable. This provides two-way binding: if the user interacts with the form element, the variable updates; if the variable changes in code, the form element’s state updates.

Common bindings:

- `bind:value` on `<input>`, `<textarea>`, `<select>` – binds to their value property. (For checkboxes and radios, the value might be slightly special: checkboxes often you bind `checked` instead, or bind:value for a number input yields a string unless input type=number, etc.)
- `bind:checked` for checkboxes (true/false binding).
- `bind:group` for radio groups or checkbox groups (to bind a set of inputs to an array or single value).
- `bind:files` on `<input type="file">` (to get FileList).
- `bind:open` on `<details>` element (to bind the open/closed state).
- `bind:this` on any element or component – to get a reference to the DOM element or component instance (this is not two-way data, just capturing a reference).

**Example of input binding:**

```svelte
<script>
	let name = $state(''); // using $state to make it reactive in Svelte 5
	let isSubscribed = $state(false);
</script>

Name: <input type="text" bind:value={name} />
<p>Hello {name}!</p>

<label>
	<input type="checkbox" bind:checked={isSubscribed} />
	Subscribe to newsletter
</label>
<p>{isSubscribed ? 'Thank you for subscribing!' : 'You are not subscribed.'}</p>
```

Here, typing in the text box updates `name` automatically, and that reflects in the greeting. Clicking the checkbox updates `isSubscribed` and thus toggles the message. If we were to programmatically set `name` or `isSubscribed`, the form elements would update too. This is classic two-way binding in Svelte and reduces boilerplate of manually writing `on:input` event handlers.

Under the hood, Svelte sets up event listeners on these elements (like `input` or `change`) to update the variable, and also updates the element if the variable changes.

**Component bindings:** (two-way binding to child props):
We covered this with $bindable. Quick recap: if a child component declares a prop with $bindable, the parent can use `bind:propName` to bind a variable to that prop. This is essentially an abstraction of the child calling something like an event or function to update the parent. Use it sparingly (mostly for form-like components or very tight coupling scenarios).

**bind:this:** This allows you to get a reference to a DOM node or component instance. For DOM nodes, you get the actual HTMLElement (e.g., HTMLInputElement). For components, you get the component's API (which is basically an object with .$set, .$on, .$destroy, etc., plus any exported functions or properties if using an Imperative API).

Example of bind:this for a DOM element:

```svelte
<script>
	let sectionEl;
	function scrollToSection() {
		sectionEl.scrollIntoView({ behavior: 'smooth' });
	}
</script>

<button on:click={scrollToSection}>Go to Section</button>

<div style="height: 2000px;">(spacer)</div>
<!-- just to create scroll -->
<section bind:this={sectionEl}>
	<h2>Target Section</h2>
</section>
```

Here `sectionEl` will be set to the DOM node of the `<section>` once it’s rendered. Clicking the button calls `scrollToSection()`, which uses the DOM method `scrollIntoView` on that element to scroll the page to it.

For a component instance:

```svelte
<!-- Parent.svelte -->
<script>
	import Child from './Child.svelte';
	let childInstance;
	function updateChild() {
		// call a method on child component instance if it has any
		childInstance?.someMethod();
	}
</script>

<Child bind:this={childInstance} />
<button on:click={updateChild}>Update Child</button>
```

If Child.svelte has an `export function someMethod()` or similar, you can call it via the `childInstance`. This is part of the Imperative component API (useful in some advanced cases, but often there's a more reactive way; still, it's available).

**Bindings summary:**

- Use `bind:value` (and siblings) for form inputs to simplify state management.
- Use `bind:this` when you need direct access to an element or component (scrolling, focus management, etc.).
- Two-way component bindings (`bind:prop`) require coordination with the child using $bindable.

## 10. Templates and Syntax

Svelte’s template syntax provides a concise way to manage conditional rendering, lists, asynchronous data, and more. It also includes directives for reacting to events, applying classes, etc. This section covers the most commonly used template features:

### Conditional Blocks: `{#if ...}{:else}`

Use `{#if}` blocks to conditionally render content.

```svelte
{#if loggedIn}
	<p>Welcome back!</p>
{:else}
	<p>Please log in.</p>
{/if}
```

If `loggedIn` is truthy, the first paragraph is rendered; otherwise the else content is rendered. You can also have an `{:else if ...}` chain for multiple conditions:

```svelte
{#if count === 0}
	<p>No items.</p>
{:else if count === 1}
	<p>1 item.</p>
{:else}
	<p>{count} items.</p>
{/if}
```

Svelte will update the DOM efficiently when the condition changes (adding or removing elements as needed).

### Each Blocks: `{#each ...}`

Use `{#each}` to loop over an array (or any iterable) and render a block for each item.

```svelte
<ul>
	{#each todos as todo, index}
		<li>{index + 1}: {todo.text}</li>
	{/each}
</ul>
```

In this example, `todos` is assumed to be an array. Inside the each block, we have access to each `todo` item and its `index`. The syntax `{#each array as item, i}` is how you get the index; you can omit `, i` if you don't need the index.

If you need to destructure each item (say each item is an object and you want its properties), you can do: `{#each people as { name, age }}` which declares `name` and `age` for each iteration.

**Keyed each:** By default, Svelte tracks list items by their position. If the list can rearrange or update out of order, it's better to provide a unique key to help Svelte maintain DOM stability. You do this by adding a key after the `as` clause:

```svelte
{#each todos as todo, index (todo.id)}
	<li>{todo.text}</li>
{/each}
```

Here `(todo.id)` is the key. Svelte will use `todo.id` to track each item, so if the array is reshuffled, it can move DOM elements instead of recreating them, and preserve any internal state in those subcomponents if present.

### Await Blocks: `{#await ...}`

Await blocks help handle Promises directly in markup. They have three segments: pending, then, and catch.

```svelte
{#await promise}
	<p>Loading...</p>
{:then value}
	<p>Resolved: {value}</p>
{:catch error}
	<p>Error: {error.message}</p>
{/await}
```

- The part after `{#await promise}` up to `{:then}` is the pending state, shown while the promise is unresolved.
- After `{:then value}`, you specify a variable name (`value` here) to represent the resolved value, and that block shows when the promise fulfills.
- After `{:catch error}`, you get the error if the promise rejects.

The `{:catch}` block is optional; if omitted, errors will bubble up (potentially to an error boundary or the console). The `{:then}` block is required if you want to do something with the result. If you only want to handle pending and then (and let errors throw), you can omit catch.

You can also use an await block without a separate pending block:

```svelte
{#await promise then data}
	<p>{data}</p>
{/await}
```

This will automatically show nothing (or keep previous content) while pending, then show data when ready.

### Key Blocks: `{#key ...}`

Key blocks are used to force recreation of a block when some expression changes.

```svelte
{#key locale}
	<!-- content that should fully re-render when locale changes -->
	<ComponentThatDependsOnLocale {locale} />
{/key}
```

In this example, whenever `locale` changes, Svelte will destroy whatever is inside the key block and recreate it. This is useful if simply updating props isn't enough and you need a full reset of a component or chunk of DOM when a value changes. A typical use is for resetting an animation or restarting a component’s lifecycle.

Another use: If you have two branches that use the same components but you want them to have distinct state, you could key them by something:

```svelte
{#if showWidgetA}
	{#key 'A'}
		<Widget />
	{/key}
{:else}
	{#key 'B'}
		<Widget />
	{/key}
{/if}
```

Now Widget will be recreated fresh when switching between A and B, rather than reusing the same instance.

### Snippet Blocks: `{#snippet name(props)}` and `{@render name(props)}`

Snippets (introduced in Svelte 5) allow you to define a reusable piece of markup within the same component. It's like creating an inline component/template that you can use multiple times or pass around, without splitting into a separate component file.

**Define a snippet:**

```svelte
{#snippet greet(name)}
	<span>Hello, {name}!</span>
{/snippet}
```

This defines a snippet named `greet` that takes a parameter `name`.

**Render a snippet:**

```svelte
<p>{@render greet('Alice')}</p><p>{@render greet('Bob')}</p>
```

This will output:

```
<p><span>Hello, Alice!</span></p>
<p><span>Hello, Bob!</span></p>
```

So, instead of duplicating the `<span>Hello, ...</span>` markup, we defined it once and reused it.

Snippets can also be passed to child components as if they were a chunk of JSX/slot. For example, you could have a component expecting a snippet prop and use `{@render snippetName(props)}` inside it. This gets advanced, but essentially snippets can be passed around as first-class template chunks.

The typical use case is to avoid repeating markup patterns or to create something like local "slot" content to pass to a child without making a new component for it.

If your snippet doesn't need parameters, you can define it without `(args)`.

### Raw HTML: `{@html ...}`

Sometimes you have a string of HTML (perhaps from an API or CMS) that you want to insert into your component. Svelte’s normal text interpolation (`{variable}`) escapes HTML for security (to prevent XSS). If you trust the source and want to insert it as real HTML, you can use `{@html someString}`.

```svelte
<script>
	let htmlString = '<em>This is italic and <strong>this is bold</strong></em>';
</script>

<p>{@html htmlString}</p>
```

This will render as:

<p><em>This is italic and <strong>this is bold</strong></em></p> 
with actual emphasis and bold, rather than showing the raw `<em>` tags.

**Important:** Only use `@html` with content you are certain is safe (or that you've sanitized). Otherwise, you open up the possibility of script injection or malicious HTML. Svelte does not sanitize for you; it simply inserts the HTML.

`@html` can also be used for small snippets of HTML you generate within the app, but often string interpolation is easier unless you explicitly need the string to be parsed as HTML.

### Debugging: `{@debug ...}`

You can insert `{@debug variable}` in a component to pause execution and debug when that line is hit. It’s like putting a breakpoint that triggers whenever the component renders (or re-renders) and the specified variable is in scope.

For example:

```svelte
<script>
  let user = { name: 'Jane', age: 30 };
  {@debug user}
</script>
```

When this component runs (in dev mode), it will trigger a `debugger` statement in the browser dev tools if open, pausing execution and allowing you to inspect `user` (and other in-scope variables). It also logs the value to the console by default.

This is a development aid and should be removed or disabled in production (the Svelte compiler actually strips it out in production builds).

### Constants in Template: `{@const ...}`

`{@const}` allows you to define a constant value within the template scope. It can be useful to avoid recomputing a value multiple times in the template or to give a name to a sub-expression.

Example:

```svelte
{#each items as item}
	{@const fullName = `${item.first} ${item.last}`}
	<p>{fullName} - {fullName.length} characters</p>
{/each}
```

Here, instead of writing `${item.first} ${item.last}` twice (and computing it twice), we compute it once in a const. The const is scoped to the each block iteration. It’s not reactive (it's constant during that render), but if `item.first` or `.last` changes and triggers a rerender of that block, the const will recompute then.

`{@const}` can also be used at the top level of a block or component to shorten repeated lookups or calculations.

### Event Directives: `on:`

Use `on:eventName={handler}` to listen for DOM events.

```svelte
<button on:click={handleClick}>Click me</button>
```

This attaches a click event listener that calls `handleClick` (a function in your script) when clicked.

You can use any standard DOM event (on:input, on:submit, on:mouseover, etc.), and custom events from components (if a child component `dispatch`es an event, you can listen with on:<event> on that component tag).

**Event Modifiers:** You can modify event behavior by adding modifiers after the event name, separated by `|`. For example:

- `on:click|preventDefault={submit}` will call `submit` but first prevent the default click behavior (e.g., preventing a form submit or link navigation).
- `on:keydown|once={doSomething}` would remove the listener after the first event.
- Other modifiers: `stopPropagation`, `passive`, `nonpassive`, `capture`, `self` (only trigger if event.target is the element itself, not a child).

Example:

```svelte
<form on:submit|preventDefault={save}>
	<!-- form fields -->
	<button type="submit">Save</button>
</form>
```

Here, `preventDefault` stops the form from actually submitting and reloading the page; instead our `save` function will handle it (likely to send via fetch or form actions etc.), which is typical in SPA forms.

### Bindings (bind:) in Template

We covered this in section 9, but to reiterate:

- `bind:` is used on input elements to bind to component state.
- `bind:this` to get element references.
- Example revisited:
  ```svelte
  <input type="range" min="0" max="100" bind:value={volume} />
  <span>Volume: {volume}</span>
  ```
  The span will update as the slider moves, and if you set `volume` in script, the slider moves accordingly.

### Class Directives: `class:`

This directive toggles a CSS class based on a condition.

```svelte
<div class:highlight={isActive}>Clickable item</div>
```

If `isActive` is true, the `highlight` class will be added to the div; if false, it will be removed. This is a convenient shorthand for binding class presence.

You can also bind multiple classes:

```svelte
<p class:bold={important} class:italic={note}></p>
```

Or even use an expression:

```svelte
<div class:selected={currentId === id}>...</div>
```

This is better than manually managing class strings because Svelte will ensure it's added/removed efficiently.

### Style Directives: `style:`

Similar to class, you can bind style properties.

```svelte
<div style:color={textColor} style:background-color={bgColor}>Colored box</div>
```

If `textColor = 'red'`, it sets style color to red. This is reactive: change `textColor` and it updates the style.

Use kebab-case for CSS properties after the colon (background-color, not backgroundColor).

### Action Directive: `use:`

The `use:` directive allows you to apply an _action_ to an element. Actions are functions that run when an element is created, and can set up some behavior or side effect on that element (like adding an event listener, integrating a third-party library, etc.).

An action is a function exported from a module (often returning an object with an update or destroy method if needed).

For example, a simple action to autofocus an element:

```js
// actions.js
export function autofocus(node) {
	node.focus();
	return {
		destroy() {
			// optional cleanup if needed
		}
	};
}
```

Use in a component:

```svelte
<script>
	import { autofocus } from './actions.js';
</script>

<input use:autofocus />
```

This will call `autofocus(inputElement)` when the input is added to the DOM, focusing it.

Actions can also take parameters:

```js
export function tooltip(node, text) {
	let title = text;
	const show = () => {
		/* show tooltip with title */
	};
	node.addEventListener('mouseover', show);
	return {
		update(newText) {
			title = newText;
		},
		destroy() {
			node.removeEventListener('mouseover', show);
		}
	};
}
```

Use: `<button use:tooltip={'Save changes'}>Save</button>`. If the text changes (if you used a variable), the `update` will be called.

Actions are an advanced but powerful way to integrate non-Svelte behavior.

### Transition and Animation: `transition:`, `in:`, `out:`, `animate:`

Svelte has built-in support for transitions (enter/leave animations) and animations on list change.

- **transition:** Use on an element that appears or disappears (like inside an `{#if}` block) to animate it.

  ```svelte
  <script>
  	import { fly } from 'svelte/transition';
  	let show = false;
  </script>

  <button on:click={() => (show = !show)}>Toggle</button>
  {#if show}
  	<div transition:fly={{ y: 20, duration: 300 }}>This flies in and out.</div>
  {/if}
  ```

  When `show` becomes true, the `<div>` will fly in (from 20px below). When `show` becomes false, it flies out. `fly` is one of the built-in transitions (others include fade, slide, scale, draw, etc., or you can write custom).

- **in: and out::** If you need different transitions on intro vs outro, or the element might exist initially (like in a each block or keyed list), you can use `in:` for entering and `out:` for leaving.

  ```svelte
  <div in:fade={{ duration: 200 }} out:fade={{ duration: 200 }}>Fades in/out</div>
  ```

  (Using fade from `svelte/transition`.)

- **animate:** For animating reordering of list elements in an `{#each}`, Svelte provides an `animate:flip` (and others) which uses FLIP animation technique. Use `animate:flip` on an each block element to have smooth movement when the order changes.
  ```svelte
  <ul>
  	{#each items as item (item.id)}
  		<li animate:flip>{item.text}</li>
  	{/each}
  </ul>
  ```
  If you reorder `items`, instead of jumping, the `<li>` elements will smoothly transition to their new positions.

Transitions and animations require importing from `svelte/transition` or `svelte/animate`. They only work in the client (no effect during SSR, though SSR will apply the final state). They greatly enhance UI without needing a separate animation library.

### Comments in Templates:

You can use `<!-- HTML comments -->` in Svelte templates; they will be ignored by Svelte (not rendered). Svelte also allows `{/* ... */}` JavaScript-style comments inside expressions or scripts which are removed.

### Special Elements: `<svelte:head>`, `<svelte:options>`, etc.

Svelte provides some special elements:

- `<svelte:head>`: In a SvelteKit page or layout, content inside `<svelte:head>` goes to the document head (for setting `<title>`, meta tags, etc.). Example:
  ```svelte
  <svelte:head>
  	<title>{post.title} – My Blog</title>
  	<meta name="description" content={post.summary} />
  </svelte:head>
  ```
- `<svelte:options>`: As seen with customElement or accessors. You can also use it to set `accessors` (to allow props to be get/set on component instances in JavaScript, usually not needed), or `immutable` (tells the compiler that your props won't change, which can enable certain optimizations).
- `<svelte:window>` and `<svelte:body>`: to directly attach event listeners to window or document body. e.g.:
  ```svelte
  <svelte:window on:mousemove={handleMouse} />
  ```
  This listens to the window’s mousemove events.
- `<svelte:fragment>` (legacy in Svelte 5, replaced by snippet perhaps).
- `<svelte:element>`: dynamic element - allows you to instantiate an HTML element specified by a string. Example:
  ```svelte
  <script>
  	let tag = 'h1';
  </script>

  <svelte:element this={tag}>Hello</svelte:element>
  ```
  If `tag` is "h1", this renders `<h1>Hello</h1>`. If you change `tag` to "p", it becomes `<p>Hello</p>` dynamically.
- `<svelte:component>` (legacy in Svelte 5, replaced by a different approach possibly).
- `<svelte:head>` we covered; use it in layouts/pages for SEO.
- `<svelte:boundary>`: This is new in Svelte 5 for error boundaries within a component subtree. You wrap part of a component with `<svelte:boundary>` to catch errors and possibly show fallback UI.

Given the question scope, it's enough to know `<svelte:head>` and `<svelte:options customElement>` as we used in $host example.

---

This reference has covered the main aspects of SvelteKit 2.0 and Svelte 5:

- We started with project structure and routing, explaining the setup of pages and advanced routing features.
- We went through state management (with Svelte stores and SvelteKit considerations) and the new Svelte 5 runes for reactivity.
- We detailed page options for controlling rendering mode and how to handle forms with actions.
- Deployment using adapters was described, and the usage of hooks for global logic and Svelte component lifecycles for component-level logic.
- We explained context, props, and binding for passing data around and linking state to UI.
- Finally, we covered the template syntax for conditional rendering, loops, awaiting async data, reusing snippets, inserting raw HTML, and reacting to events and applying classes or transitions.

Use this document as a cheat sheet or reference for building SvelteKit 2 / Svelte 5 applications. It should help you recall the correct syntax and best practices for common tasks and patterns while using a GPT-based assistant or during development. Happy coding with Svelte!
