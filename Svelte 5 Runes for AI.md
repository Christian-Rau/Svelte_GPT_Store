Svelte 5 Runes Documentation for AI Bots  
Generated on: April 06, 2025, 12:54 AM PDT  
Source URLs: https://svelte.dev/docs/svelte/what-are-runes, https://svelte.dev/docs/svelte/$state, https://svelte.dev/docs/svelte/$derived, https://svelte.dev/docs/svelte/$effect, https://svelte.dev/docs/svelte/$props, https://svelte.dev/docs/svelte/$bindable, https://svelte.dev/docs/svelte/$inspect, https://svelte.dev/docs/svelte/$host  

What are runes?  
Runes are function-like symbols in Svelte 5 that provide instructions to the Svelte compiler. They control reactivity and state management in .svelte, .svelte.js, and .svelte.ts files. Runes are part of the Svelte language syntax, not imported values, and are valid only in specific contexts. They enable explicit reactivity, replacing Svelte 4’s implicit reactivity system, and work universally across components and modules. Key runes include $state, $derived, $effect, $props, $bindable, $inspect, and $host.  

$state  
The $state rune declares reactive state. Variables defined with $state trigger UI updates when changed. Example: let count = $state(0); increments with count++ update the DOM automatically. For objects and arrays, $state creates deeply reactive proxies, tracking property changes (e.g., obj.prop = 1 or arr.push(2)). Non-proxy types like Set or Map require reactive versions from svelte/reactivity. $state.raw creates non-reactive state that must be reassigned, not mutated. Example: let items = $state.raw([1]); items = [...items, 2]. $state.snapshot captures a static copy of reactive state.  

$derived  
The $derived rune declares state dependent on other reactive values. It recalculates when dependencies change, without side effects. Example: let count = $state(0); let doubled = $derived(count * 2); updates doubled when count changes. $derived tracks dependencies at runtime, not compile time, allowing complex logic in functions via $derived.by. Example: let total = $derived.by(() => numbers.reduce((a, b) => a + b, 0)). Mutations to derived values affect the source if deeply reactive. Reassignment is possible for temporary overrides (e.g., optimistic UI updates).  

$effect  
The $effect rune runs side-effect code after state changes or component mounting, only in the browser. Example: let size = $state(50); $effect(() => context.fillRect(0, 0, size, size)); redraws when size changes. It tracks synchronous reactive dependencies (e.g., $state, $derived) and reruns when they update. Avoid state updates inside $effect to prevent infinite loops; use $derived for derivations. $effect.pre runs before DOM updates, $effect.root creates manually controlled effects, and $effect.tracking checks dependency tracking.  

$props  
The $props rune defines component properties. It replaces export let syntax. Example: let { name } = $props(); accepts a name prop. Props are not bindable by default. Use destructuring for multiple props: let { a, b } = $props(). TypeScript support: let { a: number } = $props(). Default values: let { x = 10 } = $props(). Rest props use ...rest syntax: let { x, ...rest } = $props(). Props are immutable unless marked with $bindable.  

$bindable  
The $bindable rune marks props as bindable, allowing two-way binding from parent components. Example: let { value = $bindable(0) } = $props(); enables <Component bind:value={parentValue}>. Without $bindable, binding causes an error. Fallback values require non-undefined bound values to avoid ambiguity. Example: let { foo = $bindable('bar') } = $props(); needs explicit foo prop when bound. Mutation of non-bindable props triggers warnings.  

$inspect  
The $inspect rune logs reactive state changes in development mode. Example: let count = $state(0); $inspect(count); logs count on init and updates. It tracks deep changes (e.g., obj.prop). Customize with .with: $inspect(count).with((type, val) => console.trace(val)); type is 'init' or 'update'. Useful for debugging dependency changes. Ignored in production builds. Example: $inspect(obj).with(debugger) pauses execution on change.  

$host  
The $host rune accesses the host element in custom element components. Requires <svelte:options customElement="name" />. Example: $host().dispatchEvent(new CustomEvent('event')); sends events from the custom element. Only valid inside custom element instances, errors elsewhere. Useful for DOM manipulation or event dispatching in web components. Example: let el = $host(); el.style.color = 'red'.  

---

