<script>
  import { Badge, Button } from "sveltestrap";
  import PlayButton from "./playButton.svelte";

  export let clip;
  export let index;
  export let del;
  export let play;
</script>

<style>

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
