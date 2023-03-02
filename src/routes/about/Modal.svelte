<script>
  export let showModal; // boolean
  export let startTime; // boolean
  let dialog; // HTMLDialogElement

  $: if (dialog && showModal) dialog.showModal();

  const clickHandler = () => {
    dialog.close();
    startTime = Date.now();
    console.log("HERE");
    startTime = Date.now();
    console.log("START TIME");
    console.log(startTime);
  };
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<dialog bind:this={dialog} on:close={() => (showModal = false)}>
  <div on:click|stopPropagation>
    <slot name="header" />
    <hr />
    <slot />
    <hr />
    <!-- svelte-ignore a11y-autofocus -->
    <button autofocus on:click={clickHandler}>START TIME</button>
  </div>
</dialog>

<style>
  dialog {
    max-width: 64em;
    height: 64em;
    border-radius: 0.2em;
    border: none;
    padding: 0;
  }
  dialog::backdrop {
    background: rgba(0, 0, 0, 0.3);
  }
  dialog > div {
    padding: 1em;
  }
  dialog[open] {
    animation: zoom 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  }
  @keyframes zoom {
    from {
      transform: scale(0.95);
    }
    to {
      transform: scale(1);
    }
  }
  dialog[open]::backdrop {
    animation: fade 0.2s ease-out;
  }
  @keyframes fade {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }
  button {
    display: block;
  }
</style>
