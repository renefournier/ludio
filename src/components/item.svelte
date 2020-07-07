<script>
  export let item;
  export let index;
  export let del;
  export let play;
  import Clip from "./clip.svelte";
</script>

<Clip clip={item} {index} {del} {play} />
