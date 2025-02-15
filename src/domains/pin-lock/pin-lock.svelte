<script lang="ts">
  import Text from '@/components/text/text.svelte'
  // Components
  import Keypad from './keypad.svelte'
  // Stores
  import nid from '../../modules/nid/nid'
  import { Interact } from '../../store/interact'
  import Interactions from '../interactions/interactions.svelte'
  import { Lang } from '../../store/lang'
  import Button from '@/components/button/button.svelte'
  import Icon from '@/components/icon/icon.svelte'
  import Backdrop from '@/components/backdrop/backdrop.svelte'

  let _pin = ''

  $: if ($Interact.pin.show === false) {
    _pin = ''
  }

  $: if (_pin.length > 6) {
    _pin = _pin.substr(0, 6)
  }

  const methods = {
    submit() {
      // encode the pin and send it up
      let final = _pin || ''

      if (final.length < 7 && final.length > 0) {
        $Interact.pin.onPin(nid(_pin))
      } else {
        Interact.error('Pin must be between 1 and 6 characters')
      }
    },
    cancelInput() {
      Interact.cancelPin()
    },
  }
</script>

<style global>
  .lock-screen h1 {
    color: #fff;
    margin-bottom: 20px;
    border-bottom: solid 1px rgba(255, 255, 255, 0.2);
    padding-bottom: 20px;
    width: 100%;
    text-align: center;
  }
  .lock-screen .pin-holder {
    background-color: rgba(255, 255, 255, 0.12);
    max-width: 280px;
    width: 280px;
    max-height: 40px;
    border-radius: 6px;
    margin-bottom: 12px;
    justify-content: center;
  }
  .lock-screen .pin-close-btn {
    position: fixed !important;
    top: 20px;
    left: 20px;
  }
</style>

<Backdrop id="lock-screen" visible={$Interact.pin.show}>
  <div
    aria-modal
    aria-label="Lock Screen"
    aria-hidden={!$Interact.pin.show}
    style="zIndex: 10000"
    class="lock-screen p-4 rounded-xl shadow-xl items-center justify-center
    flex-col full-screen bg-primary-500 {$Interact.pin.show ? 'visible' : 'hidden'}">
    <Text center size="sm" faded className="text-white mb-2">
      {Lang.t('settings.pin-requirements', '1 to 6 digits')}
    </Text>
    <h1>{$Interact.pin.title}</h1>
    {#if $Interact.pin.show}
      <!-- Pin Display -->
      <div
        class="flex items-center justify-center h-10 text-2xl text-white pin-holder">
        {#each _pin.split('') as d}•{/each}
      </div>
      <!-- Keypad Input -->
      <Keypad bind:value={_pin} on:submit={methods.submit} />
      {#if $Interact.pin.canClose}
        <Button
          icon
          color="clear"
          className="pin-close-btn"
          ariaLabel="Cancel"
          on:click={methods.cancelInput}>
          <Icon name="close" className="fill-white" size={32} />
        </Button>
      {/if}
    {/if}

  </div>
</Backdrop>
