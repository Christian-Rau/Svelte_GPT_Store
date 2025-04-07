Svelte 5 Runes Documentation for AI Bots  
Generated on: April 06, 2025, 01:15 AM PDT  
Source URLs: https://svelte.dev/docs/svelte/what-are-runes, https://svelte.dev/docs/svelte/$state, https://svelte.dev/docs/svelte/$derived, https://svelte.dev/docs/svelte/$effect, https://svelte.dev/docs/svelte/$props, https://svelte.dev/docs/svelte/$bindable, https://svelte.dev/docs/svelte/$inspect, https://svelte.dev/docs/svelte/$host  

What are runes?  
Runes are compiler instructions in Svelte 5, used in .svelte, .svelte.js, and .svelte.ts files to manage reactivity explicitly. They replace Svelte 4’s implicit reactivity with a signal-based system, offering universal reactivity across components and modules. Runes are syntactic keywords, not importable values, and include $state, $derived, $effect, $props, $bindable, $inspect, and $host.  
Code Example: Basic Rune Usage  
<script>  
let count = $state(0);  
let doubled = $derived(count * 2);  
$effect(() => console.log(doubled));  
</script>  
<button onclick={() => count++}>{count}</button>  
Explanation: $state defines reactive state, $derived computes dependent values, $effect runs side effects when dependencies change.  

$state  
$state declares reactive variables that trigger UI updates on change. It supports primitive types and creates deeply reactive proxies for objects and arrays. Use $state.raw for non-reactive state requiring reassignment, and $state.snapshot for static copies.  
Code Example: Simple Counter  
<script>  
let count = $state(0);  
</script>  
<button onclick={() => count++}>Clicks: {count}</button>  
Code Example: Deep Reactivity  
<script>  
let obj = $state({ value: 0 });  
</script>  
<button onclick={() => obj.value++}>{obj.value}</button>  
Code Example: Raw State  
<script>  
let list = $state.raw([1]);  
function add() { list = [...list, list.length + 1]; }  
</script>  
<button onclick={add}>{list.join(', ')}</button>  
Explanation: count++ updates the DOM directly. obj.value mutations are tracked. list requires reassignment for updates.  

$derived  
$derived defines values computed from other reactive state, recalculating only when dependencies change. It tracks dependencies at runtime, supports complex logic via $derived.by, and allows temporary reassignment.  
Code Example: Basic Derived Value  
<script>  
let x = $state(5);  
let doubled = $derived(x * 2);  
</script>  
<p>{x} doubled is {doubled}</p>  
Code Example: Complex Derivation  
<script>  
let numbers = $state([1, 2, 3]);  
let sum = $derived.by(() => numbers.reduce((a, b) => a + b, 0));  
</script>  
<p>Sum: {sum}</p>  
Explanation: doubled updates when x changes. sum recalculates when numbers mutates.  

$effect  
$effect executes side effects after state changes or component mounting, running only in the browser. It tracks synchronous reactive dependencies and avoids state updates within to prevent loops. Variants include $effect.pre and $effect.root.  
Code Example: Canvas Drawing  
<script>  
let size = $state(50);  
let canvas;  
$effect(() => {  
  let ctx = canvas.getContext('2d');  
  ctx.clearRect(0, 0, canvas.width, canvas.height);  
  ctx.fillRect(0, 0, size, size);  
});  
</script>  
<canvas bind:this={canvas} width=100 height=100 />  
Explanation: fillRect reruns when size changes, updating the canvas.  

$props  
$props declares component properties, replacing export let. Props are immutable by default, with TypeScript and default value support. Use rest syntax for remaining props.  
Code Example: Basic Props  
<script>  
let { name } = $props();  
</script>  
<p>Hello, {name}</p>  
Code Example: Typed Props with Defaults  
<script>  
let { count: number = 0, ...rest } = $props();  
</script>  
<p>Count: {count}</p>  
Explanation: name is a required prop. count has a default and type, rest captures extra props.  

$bindable  
$bindable marks props as bindable for two-way data flow. It requires explicit binding in the parent and supports fallback values.  
Code Example: Bindable Input  
<script>  
let { value = $bindable(0) } = $props();  
</script>  
<input bind:value={value} />  
Usage in Parent:  
<script>  
let num = $state(5);  
</script>  
<Component bind:value={num} />  
<p>{num}</p>  
Explanation: value binds to num, syncing changes bidirectionally.  

$inspect  
$inspect logs reactive state changes in development mode, tracking deep mutations. Customize with .with for alternative logging. It’s ignored in production.  
Code Example: Basic Inspection  
<script>  
let count = $state(0);  
$inspect(count);  
</script>  
<button onclick={() => count++}>{count}</button>  
Code Example: Custom Logging  
<script>  
let obj = $state({ x: 0 });  
$inspect(obj).with((type, val) => console.log(type, val));  
</script>  
<button onclick={() => obj.x++}>{obj.x}</button>  
Explanation: Logs count on init and updates. Logs type and obj on deep changes.  

$host  
$host accesses the host element in custom elements, requiring <svelte:options customElement>. It’s invalid outside this context.  
Code Example: Custom Element Event  
<svelte:options customElement="my-element" />  
<script>  
function trigger() {  
  $host().dispatchEvent(new CustomEvent('click'));  
}  
</script>  
<button onclick={trigger}>Click</button>  
Explanation: Dispatches event from the custom element’s host.  

