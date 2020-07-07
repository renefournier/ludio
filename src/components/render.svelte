<script>
  import { Button } from "sveltestrap";

  export let clips;
  export let render;
</script>

<div class="mx-auto p-0">

  <Button class="w-100" on:click={render}>
    <span class="fas fa-download pr-1" />
    EXPORT
    <small class="faint">
      <strong class="ml-2">
        <span class="fa fa-keyboard " />
        e
      </strong>
    </small>
  </Button>

</div>
