<script>
  import { Badge, Button } from "sveltestrap";
  import PlayButton from "./playButton.svelte";

  export let clip;
  export let index;
</script>

<style>
  .clip {
    display: block;
    padding: 0.5rem 1rem;
    border-radius: 1rem;
    box-shadow: 0 2px 10px -3px rgba(0, 0, 0, 0.2);
    background: white;
    margin: 0.25rem;
  }
</style>

<div class="clip">
  <span class="float-right">
    <span class="muted mx-3 ">{Number(clip.duration).toFixed(1)}s</span>

    <PlayButton {clip} state="true" />

  </span>

  <Badge pill color="light">{index + 1}</Badge>

  <strong class="muted">
    <span class="fa fa-lg fa-film mx-2" />
  </strong>

  <span class="muted">{clip.start.toFixed(2)}</span>
  :
  <span class="muted">{clip.stop.toFixed(2)}</span>

</div>
