<script>
  import { Button } from "sveltestrap";

  export let currentTime;
  export let start;
  export let stop;
  export let markIn;
  export let markOut;
</script>

<div class="mx-auto p-1">
  {#if start === null && stop === null}
    <Button lg block size="lg" on:click={markIn}>
      <span class="fas fa-video" />
      IN
      {#if currentTime}
        <strong class="mx-5">{currentTime}s</strong>
      {/if}
      <small class="muted">
        <strong class="ml-4">
          <span class="fa fa-keyboard " />
          i
        </strong>
      </small>
    </Button>
  {:else if start !== null && stop === null}
    <Button lg block success size="lg" color="success" on:click={markOut}>
      <span class="fas fa-video-slash" />
      OUT
      {#if currentTime}
        <strong class="mx-5">{currentTime}s</strong>
      {/if}
      <small class="muted">
        <strong class="ml-4">
          <span class="fa fa-keyboard " />
          o
        </strong>
      </small>
    </Button>
  {/if}
</div>
