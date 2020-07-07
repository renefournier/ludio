<script>
  import { Button } from "sveltestrap";

  export let start;
  export let stop;
  export let markIn;
  export let markOut;
</script>

<div class="mx-auto p-0">
  {#if start === null && stop === null}
    <Button class="w-100" on:click={markIn}>
      <span class="fas fa-video" />
      IN
      <small class="faint">
        <strong class="ml-2">
          <span class="fa fa-keyboard " />
          i
        </strong>
      </small>
    </Button>
  {:else if start !== null && stop === null}
    <Button class="w-100" success color="success" on:click={markOut}>
      <span class="fas fa-video-slash" />
      OUT
      <small class="faint">
        <strong class="ml-2">
          <span class="fa fa-keyboard " />
          o
        </strong>
      </small>
    </Button>
  {/if}
</div>
