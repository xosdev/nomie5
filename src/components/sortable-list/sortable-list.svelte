<script lang="ts">
  import { onMount } from "svelte";
  import Sortable from "sortablejs";
  // DISPATCH REORDER
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  // PROPS
  export let items: Array<any>;
  export let key: string;
  export let handle: any = undefined;

  let sortableList;

  const reorder = (to, from) => {
    let newList = [...items];
    let tempItem = "temp";
    let movedItem = newList[from];
    // Cut in the temp Item
    newList.splice(to, 0, tempItem);
    // FILTER OUT MOVED ITEM
    newList = newList
      .filter((item) => {
        return item !== movedItem;
      })
      .map((item) => {
        return item !== tempItem ? item : movedItem;
      });
    dispatch("update", newList);
  };

  function getKey(item, index) {
    if (typeof item == "string") {
      return index;
    } else if (key) {
      return item[key] || index;
    }
  }

  function main() {
    Sortable.create(sortableList, {
      handle: handle,
      onEnd: function (evt) {
        reorder(evt.newDraggableIndex, evt.oldDraggableIndex);
        // main();
      },
    });
  }

  $: if (sortableList) {
    main();
  }
</script>

<style global>
  ul.sortable-list {
    list-style: none;
    padding: 0;
    margin: 0;
    max-width: 100%;
    overflow: hidden;
  }
  .sortable-list li {
    margin: 0;
    padding: 0;
  }
  .sortable-list .sortable-chosen {
    box-shadow: 0px 10px 15px rgba(0, 0, 0, 0.3);
  }
  .sortable-list .sortable-ghost {
    opacity: 0.7;
  }
</style>

{#if items && items.length}
  <ul class="sortable-list" bind:this={sortableList}>
    {#each items as item, index (getKey(item, index))}
      <li>
        <slot {item} {index} />
      </li>
    {/each}
  </ul>
{/if}
