<script>
  import { Button } from "sveltestrap";

  export let source; // source media URL
  export let preview; // previewPlayer instance
  export let size = "lg"; // size of button
  export let clip; // either an object (single clip), or array (all clips)
  export let state = null;
</script>

<style>
  .player > * {
    cursor: pointer;
  }
  .player:hover > * {
    color: #00b4ff !important;
  }
</style>

<strong class="player" on:click={() => console.log(clip)}>
  <span class="fa fa-{size} fa-play-circle text-secondary" />
</strong>
