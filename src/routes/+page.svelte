<script lang="ts">
  import { onMount } from "svelte";
  import { io, Socket } from "socket.io-client";
  import { PUBLIC_DEV } from "$env/static/public";
  import Typewriter from "$lib/components/Typewriter.svelte";

  interface Message {
    bot: boolean;
    text: string;
  }

  let socket: Socket;
  let connecting = $state(true);
  let message = $state("");
  let roomId = "";
  let generating = $state(false);
  let messages: Message[] = $state([]);
  let reasoning = $state(false);

  onMount(() => {
    socket = io(PUBLIC_DEV === "true" ? "http://localhost:3000" : "wss://chat-jippity-server.onrender.com", { transports: ["websocket"] });

    socket.on("connect", () => {
      console.log("connected to server with id: ", socket.id);

      socket.emit("find-match", { type: "human" });

      socket.on("match-found", ({ roomId: room, partnerId }) => {
        console.log("room id: ", room);
        console.log("partner id: ", partnerId);
        roomId = room;

        connecting = false;
      });

      socket.on("receive-reply", data => {
        console.log("message received from human: ", data);
        messages.push({ bot: true, text: data });
        generating = false;
      });

      socket.on("partner-left", () => {
        console.log("partner left");
        socket.emit("find-match", { type: "human" });
        connecting = true;
        reset();
      });

      socket.on("disconnect", () => {
        console.log("disconnected from server");
      });
    });
  });

  function send() {
    if (message.length > 0) {
      socket.emit("send-message", { roomId, message, reasoning });
      messages.push({ bot: false, text: message });
      message = "";
      generating = true;
    }
  }

  function reset() {
    roomId = "";
    message = "";
    messages = [];
  }
</script>

<main class="h-screen flex flex-col items-center gap-4">
  {#if connecting}
    <div class="flex justify-center items-center h-full gap-4">
      <iconify-icon icon="mingcute:loading-3-fill" class="text-2xl animate-spin"></iconify-icon>
      <h2 class="font-semibold text-2xl">Connecting to server...</h2>
    </div>
  {:else}
    <nav class="flex h-16 items-center px-4 gap-4 w-full">
      <button class="rounded-lg hover:bg-bg-1 flex justify-center items-center size-10 hover:cursor-pointer" aria-label="menu">
        <iconify-icon icon="mingcute:pencil-2-line" class="text-2xl"></iconify-icon>
      </button>

      <h1 class="text-xl font-semibold">ChatJippity</h1>
    </nav>

    {#if messages.length > 0}
      <div class="grow w-[40rem] space-y-2 overflow-y-auto">
        {#each messages as m}
          {#if m.bot}
            <div class="flex w-full justify-start">
              <div class="rounded-3xl px-6 py-3 h-16 flex">
                <Typewriter target={m.text} />
              </div>
            </div>
          {:else}
            <div class="flex w-full justify-end">
              <div class="rounded-3xl bg-bg-1 px-5 py-3 flex">
                <p>{m.text}</p>
              </div>
            </div>
          {/if}
        {/each}

        {#if generating}
          <div class="flex w-full justify-start">
            <div class="rounded-3xl px-6 py-3 h-16 flex items-center animate-pulse">
              <p>Generating...</p>
            </div>
          </div>
        {/if}
      </div>
    {/if}

    <div class="{messages.length > 0 ? "" : "grow"} flex flex-col justify-center items-center p-4">
      <div class="flex flex-col gap-2 items-center">
        {#if messages.length === 0}
          <h2 class="text-3xl font-semibold mb-2">What can I help with?</h2>
        {/if}

        <div class="rounded-4xl bg-bg-1 px-6 py-4 w-[40rem]">
          <form on:submit|preventDefault={send} class="flex flex-col gap-4 h-full">
            <input
              type="text"
              name="message"
              id="message"
              class="outline-none grow"
              bind:value={message}
              placeholder="Ask anything"
              disabled={generating}
            >

            <div class="flex items-center justify-between">
              <button
                type="button"
                on:click={() => (reasoning = !reasoning)}
                class="flex items-center gap-1 px-2.5 py-1.5 rounded-full cursor-pointer border-2 {reasoning ? "bg-accent text-accent-1 border-accent" : "border-border"}"
              >
                <iconify-icon icon="mingcute:bulb-2-line" class="text-xl"></iconify-icon>
                <span class="text-sm">Reason</span>
              </button>

              <button
                class="rounded-full bg-white text-bg h-full aspect-square flex justify-center items-center hover:brightness-75 hover:cursor-pointer"
                aria-label="send"
                type="submit"
              >
                <iconify-icon icon="mingcute:arrow-up-fill" class="text-2xl"></iconify-icon>
              </button>
            </div>
          </form>
        </div>

        <p class="text-muted text-xs">ChatJippity doesn't make mistakes. No need to check important info.</p>
      </div>
    </div>
  {/if}
</main>
