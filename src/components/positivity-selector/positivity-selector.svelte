<script lang="ts">
  import Icon from '@/components/icon/icon.svelte'
  import { createEventDispatcher } from 'svelte'
  import { X } from 'svelte-hero-icons'
  import appConfig from '../../config/appConfig'
  import tick from '../../utils/tick/tick'
  import NButtonGroup from '../button-group/button-group.svelte'
  import Button from '../button/button.svelte'

  const dispatch = createEventDispatcher()

  export let score = 0
  export let className = ''
  export let style = ''
  export let size = 'md'
  export let transparent = false
  export let id = undefined
  export let showClose: boolean = false

  async function onChange(sc) {
    score = sc
    await tick(200)
    dispatch('change', score)
  }
</script>

<style global lang="postcss">
  .n-positivity-selector {
    @apply rounded-full;
    @apply z-50;
    @apply flex items-center justify-evenly;
    @apply p-1;
  }

  .n-positivity-selector .nbtn {
    opacity: 0.8;
    @apply rounded-full;
    @apply transform scale-75;
    @apply transition-all;
    @apply duration-200;
  }
  .n-positivity-selector .nbtn.active {
    @apply shadow-lg;
    @apply ring-gray-500 ring-opacity-20 dark:ring-gray-200 ring;
    @apply scale-100;
    @apply opacity-100;
  }
  .n-positivity-selector .nbtn-sm {
    font-size: 20px;
    width: 30px !important;
    height: 30px !important;
  }
  .n-positivity-selector .nbtn-md {
    font-size: 24px;
    width: 36px !important;
    height: 36px !important;
  }

  .n-positivity-selector .nbtn {
    @apply flex-grow-0;
    @apply flex-shrink-0;
  }

  .n-positivity-selector .nbtn-lg {
    font-size: 30px;
    width: 42px !important;
    height: 42px !important;
  }
</style>

<div
  {id}
  class="n-positivity-selector {!transparent ? 'box-shadow bg-solid' : 'bg-transparent'}
  flex items-center {className} size-{size}"
  {style}>
  {#each appConfig.positivity as posEmoji}
    <Button
      on:click={() => {
        onChange(posEmoji.score)
      }}
      {size}
      style="padding:0"
      shape="rounded"
      color="transparent"
      className={score == posEmoji.score ? 'active' : 'inactive'}>
      {posEmoji.emoji}
    </Button>
  {/each}
  {#if showClose}
    <Button
      color="transparent"
      shape="rounded"
      on:click={() => {
        dispatch('close')
      }}>
      <Icon icon={X} />
    </Button>
  {/if}
</div>
