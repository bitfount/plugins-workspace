<script>
  import { getCurrent } from "@tauri-apps/api/webview";
  import { invoke } from "@tauri-apps/api/core";
  import { onMount, onDestroy } from "svelte";

  const webview = getCurrent();

  export let onMessage;
  let unlisten;

  onMount(async () => {
    unlisten = await webview.listen("rust-event", onMessage);
  });
  onDestroy(() => {
    if (unlisten) {
      unlisten();
    }
  });

  function log() {
    invoke("log_operation", {
      event: "tauri-click",
      payload: "this payload is optional because we used Option in Rust",
    });
  }

  function performRequest() {
    invoke("perform_request", {
      endpoint: "dummy endpoint arg",
      body: {
        id: 5,
        name: "test",
      },
    })
      .then(onMessage)
      .catch(onMessage);
  }

  function emitEvent() {
    webview.emit("js-event", "this is the payload string");
  }
</script>

<div>
  <button class="btn" id="log" on:click={log}>Call Log API</button>
  <button class="btn" id="request" on:click={performRequest}>
    Call Request (async) API
  </button>
  <button class="btn" id="event" on:click={emitEvent}>
    Send event to Rust
  </button>
</div>
