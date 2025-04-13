<script lang="ts">
	import "../app.css";
  import "iconify-icon";

	let { children } = $props();
</script>

<svelte:head>
  <title>ChatJippity</title>
</svelte:head>

{@render children()}
