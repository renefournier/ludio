<script>
  import { Button } from "sveltestrap";

  export let source; // source media URL
  export let preview = null; // previewPlayer instance
  export let size = "lg"; // size of button
  export let clip; // either an object (single clip), or array (all clips)
  export let state = null;
  export let del;
  export let play;
</script>

<style>
  .player > * {
    cursor: pointer;
  }
  .player:hover > * {
    color: #00b4ff !important;
  }
</style>

<strong class="player muted" on:click={del}>
  <span class="fa fa-trash " />
</strong>

<strong class="player ml-2" on:click={play}>
  <span class="fa fa-{size} fa-play-circle " />
</strong>
