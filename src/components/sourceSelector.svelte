<script>
export let videos;
export let source_id;
export let changeSource;
</script>

<select
    type="select"
    name="source_id"
    id="source_id"
    class="source_selector"
    bind:value={source_id}
    on:change={() => changeSource(source_id)}>
    {#each videos as video, index}
        <option value={index}>{video}</option>
    {/each}
    </select>

<style>

  .source_selector {
    background: rgba(0, 0, 0, 0.25);
    border: 1px solid rgba(255, 255, 255, 0.25);
    color: rgba(255, 255, 255, 0.75);
    font-weight: bold;
    padding: 0.2rem 0.3rem;
    margin: 0.35rem auto 0;
    border-radius: 5px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.25);
    max-width: 150px;
    text-overflow: ellipsis;

  }
</style>