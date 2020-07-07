<script>
  import { Badge, Button } from "sveltestrap";
  import PlayButton from "./playButton.svelte";

  export let clip;
  export let index;
  export let del;
  export let play;
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

<Button block color="secondary" class="btn-block-level text-left">

  <span class="float-right">
    <strong class="muted mx-3 ">{Number(clip.duration).toFixed(1)}s</strong>
    <PlayButton {clip} {del} {play} />
  </span>
  <strong class="faint">
    <span class="fa fa-lg fa-film " />
  </strong>

  <Badge pill color="dark">{index + 1}</Badge>

  <span class="muted">{clip.start.toFixed(2)}</span>
  :
  <span class="muted">{clip.stop.toFixed(2)}</span>

</Button>
