<script>
  export let steps = 0;
  export let current = 0;
  export let dark = false;
  export let stepClass = "";
  export let single = false;
  export let style = "";

  let _steps = [];
  $: if (steps || current) {
    _steps = [];
    for (let i = 0; i < steps; i++) {
      if (!single) {
        _steps.push(current >= i);
      } else {
        _steps.push(current == i);
      }
    }
  }
</script>

<style lang="postcss">
.n-stepper {
	 display: flex;
	 flex-direction: row;
	 align-items: center;
	 justify-content: stretch;
	 padding: 4px;
}
 .n-stepper .step {
	 transition: all 0.2s ease-in-out;
	 border: solid 1px var(--color-translucent);
	 opacity: 0.5;
	 height: 8px;
	 width: 8px;
	 flex-grow: 0;
	 flex-shrink: 0;
	 margin: 0 2px;
	 min-width: 8px;
	 border-radius: 4px;
}
 .n-stepper .step.primary-bright {
	 border: solid 1px var(--color-primary-bright) !important;
}
 .n-stepper .step.active {
	 background-color: var(--color-translucent);
	 opacity: 1;
	 height: 8px;
}
 .n-stepper .step.active.primary-bright {
	 background-color: var(--color-primary-bright) !important;
}
 .n-stepper.dark .step {
	 background-color: var(--color-primary);
	 opacity: 0.2;
}
 .n-stepper.dark .step.active {
	 opacity: 1;
}
 
</style>

<div class="n-stepper {dark ? 'dark' : 'light'}" {style}>
  {#each _steps as step}
    <div class="step {stepClass} {step ? 'active' : 'inactive'}" />
  {/each}
</div>
