<script>
  import { slide } from "svelte/transition";
  export let img = undefined;
  export let title = "";
  export let message = [];
</script>

<style global lang="postcss">
  .slide {
		@apply flex;
		@apply flex-col;
		@apply w-full;
	 padding: 20px 20px;
	 justify-content: center;
	 align-content: center;
	 max-width: 400px;
	 height: calc(100vh - 100px);
	 max-height: 700px;
}
.slide .top {
	@apply flex;
	@apply items-center;
	@apply justify-center;
}
 .slide img {
	 max-width: 100%;
	 border-radius: 6px;
	 overflow: hidden;
	 box-shadow: 0px 10px 15px -8px rgba(0, 0, 0, 0.32);
	 flex-shrink: 0;
	 flex-grow: 0;
}
 .slide h1 {
	 font-size: 1.2em;
	 font-weight: bolder;
	 text-align: center;
	 max-width: 240px;
	 margin-left: auto;
	 margin-right: auto;
}
 .slide .content {
	 padding: 0 0 20px;
	 font-size: 1em;
}
 .slide p {
	 font-size: 0.8em;
	 font-weight: 200;
	 opacity: 0.77;
	 text-align: center;
}
 
</style>

<div class="slide" transition:slide>
  {#if img}
    <div>
      <img src={img} width="100%" alt={title} />
    </div>
  {/if}
  {#if title}
    <h1 class={img ? 'mt-3' : ''}>{title}</h1>
  {/if}
  <div class="content">
    {#if message.length}
      {#each message as message, mi}
        <p>{message}</p>
      {/each}
    {/if}
    <slot />
  </div>
</div>
