<script>
  export let item;
  export let index;

  import Clip from "./clip.svelte";
</script>

<Clip clip={item} {index} />
