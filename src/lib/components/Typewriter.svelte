<script lang="ts">
  import { onMount } from "svelte";
  import { fade } from "svelte/transition";

  const speed = 20;
  let i = 0;
  let text = $state("");
	let { target, onEnd, onStart }: {
    target: string,
    onEnd?: () => void,
    onStart?: () => void
  } = $props();
  let ended = $state(false);

  function typewriter() {
    if (i < target.length && !ended) {
      text += target.charAt(i);
      i++;
      setTimeout(typewriter, speed);
    } else {
      ended = true;
      if (onEnd) onEnd();
    }
  }

  onMount(() => {
    if (onStart) onStart();

    setTimeout(() => {
      typewriter();
    }, 1000);
  });
</script>

<p class="flex items-center gap-1">
  <span>{text}</span>

  {#if !ended}
    <span class="rounded-full bg-fg size-4 animate-pulse" transition:fade={{ duration: 200 }}></span>
  {/if}
</p>
