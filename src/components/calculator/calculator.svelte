<script lang="ts">
  import { dropRight } from 'lodash'
  // Inspirated by https://codepen.io/ethanryan/details/MryqXv

  // Math will do the calculatng
  import math from '../../utils/math/math'

  import { createEventDispatcher, onMount, onDestroy } from 'svelte'
  import Button from '../button/button.svelte'
  import is from '../../utils/is/is'
  import { wait } from '../../utils/tick/tick'

  const dispatch = createEventDispatcher()

  let globalAnswer: any = '0' //declaring global variable here... this is bad practice
  let buffer = []
  let fontSize = 40

  export let value = 0
  export let displayFormat = undefined
  export let defaultEphemeral = true

  let tapped = false

  onMount(() => {
    if (is.truthy(value)) {
      buffer = [value]
      change()
    }
  })

  const buttons = [
    ['C', '+/-', '%', '/'],
    [7, 8, 9, '*'],
    [4, 5, 6, '-'],
    [1, 2, 3, '+'],
    ['⌫', 0, '.', null],
  ]

  function handleKeydown(evt) {
    const key = evt.key
    const valid = buttons.find((row) => {
      return row.find((b) => `${b}` === `${key}`)
    })
    if (valid) {
      insertBuffer(key)
    } else if (key == 'Backspace') {
      insertBuffer('⌫')
    }
  }

  function change() {
    // buffer = buffer.filter((b) => is.truthy(b));

    if (buffer.length == 1) {
      globalAnswer = buffer[0]
    } else if (buffer.length) {
      globalAnswer = calculateBuffer()
    } else {
      globalAnswer = 0
    }
    dispatch('change', globalAnswer)
    // if (globalAnswer != value) {
    //   dispatch("change", globalAnswer);
    // }
    getFontSize()
  }

  function calculateBuffer() {
    return math.calculate(buffer)
  }

  function click(input) {
    // If we should clear a default and one exists
    if (!tapped && is.truthy(value) && defaultEphemeral) {
      value = 0
      clearBuffer()
      tapped = true
    }
    insertBuffer(input)
  }

  function clearBuffer() {
    buffer = []
  }

  function isNumber(input) {
    return !isNaN(input) && input !== null
  }

  function deleteLast() {
    // buffer = _.dropRight(buffer);
    if (buffer.length) {
      let value = `${buffer[buffer.length - 1]}`
      buffer[buffer.length - 1] = dropRight(value.split('')).join('')
      change()
    }
  }

  // Insert a key or operator into the buffer
  function insertBuffer(insert) {
    // Set last buffer
    let lastBuffer = buffer.length ? buffer[buffer.length - 1] : null
    // if its a number and so is the last buffer - merge the nubmers
    if (isNumber(insert) && isNumber(lastBuffer)) {
      buffer[buffer.length - 1] = parseFloat(`${lastBuffer}${insert}`)
      change()
      // If the insert is a number and the last buffer - set the decimal
    } else if (isNumber(insert) && lastBuffer == '.') {
      let preDecimal = buffer[buffer.length - 2]
      let postDecimal = insert
      if (!math.isFloat(preDecimal)) {
        buffer[buffer.length - 2] = `${preDecimal}.${postDecimal}`
        buffer.pop()
      }
      change()
      // If the insert is a decimal - store it, but don't change anything
    } else if (isNumber(insert)) {
      buffer.push(insert)
      change()
    } else {
      switch (insert) {
        case 'C':
          buffer = []
          change()
          break
        case '⌫':
          deleteLast()
          break
        case '+/-':
          if (globalAnswer > 0) {
            buffer = [-Math.abs(globalAnswer)]
          } else {
            buffer = [Math.abs(globalAnswer)]
          }
          change()
          break
        case '%':
          buffer = [globalAnswer / 100]
          change()
          break
        default:
          if (['+', '*', '/', '-', '.'].indexOf(insert) > -1) {
            buffer.push(insert)
            change()
          }
          break
      }
    }
    buffer = buffer
  }

  async function getFontSize() {
    await wait(10)
    let len = globalAnswer.toString().length
    if (len < 10) {
      fontSize = 40
    } else if (len >= 10 && len < 18) {
      fontSize = 30
    } else {
      fontSize = 20
    }
  }
</script>

<style lang="postcss" global>
  .n-calculator .buttons {
    @apply grid grid-cols-4;
    @apply w-full;
    @apply items-center justify-center;
    @apply px-4;
  }

  .n-calculator {
    @apply h-full;
    @apply flex flex-col;
    @apply p-2;
    @apply items-center;
    @apply justify-center;
  }
  /* @include media-breakpoint-down(xs) {
	 .buttons {
		 grid-template-columns: 25% 25% 25% 25%;
		 grid-template-rows: 60px 60px;
	}
}
 */
  @keyframes numberUp {
    from {
      transform: translateY(8px);
      opacity: 0;
    }
    to {
      transform: translateY(0);
      opacity: 1;
    }
  }
  .calc-screen {
    position: relative;
    display: flex;
    flex-direction: column;
    background-color: black;
    border: solid 1px rgba(255, 255, 255, 0.1);
    color: white;
    text-align: right;
    border-radius: 4pt;
    padding: 4pt 8pt;
    padding-bottom: 4pt;
    width: 100%;
    margin: 0 auto 8pt;
  }
  .calc-screen .value {
    line-height: 100%;
    height: 50px;
    text-align: right;
  }
  .calc-screen .preview {
    position: absolute;
    top: 4pt;
    left: 8pt;
    opacity: 0.8;
    color: #999;
    font-size: 0.8em;
  }
  .calc-screen .buffer {
    height: 20px;
    min-height: 22px;
    font-size: 0.8em;
    color: #999;
  }
  .numberUp {
    display: inline-block;
    animation: numberUp 0.4s ease-in-out;
    -webkit-animation: numberUp 0.4s ease-in-out;
  }
  .n-calculator .nbtn {
    touch-action: manipulation;
    border: none;
    color: var(--color-inverse);
    margin: 4px;
    border-radius: 50%;
    line-height: 100%;
    @apply w-16 lg:w-20;
    @apply h-16 lg:h-20;
    @apply flex-grow-0;
    @apply flex-shrink-0;
    @apply text-lg md:text-xl;
    @apply mx-auto;
    /* @include media-breakpoint-down(xs) {
		 height: 50px;
		 width: 50px;
		 font-size: 20px;
	}
	 */
  }
  .n-calculator .nbtn.r-0 {
    color: #fff;
    background-color: var(--color-solid-1);
  }
  .n-calculator .nbtn.r-0.b-0 {
    color: #fff;
    background-color: var(--color-red);
  }
  .n-calculator .nbtn.r-0.b-1 {
    color: #fff;
    background-color: #444;
  }
  .n-calculator .nbtn.r-0.b-2 {
    color: #fff;
    background-color: #444;
  }
  .n-calculator .nbtn.b-3 {
    color: #fff;
    background-color: var(--color-orange);
  }
  .n-calculator .nbtn.b-0.r-4 {
    background-color: transparent;
    box-shadow: none;
  }
</style>

<svelte:window on:keydown={handleKeydown} />
<div class="w-full n-calculator">
  <div class="calc-screen">
    {#if displayFormat}
      <div class="preview">{displayFormat(globalAnswer)}</div>
    {/if}
    <div class="buffer">
      {#if buffer.length > 1}
        {#each buffer as bit}
          <span>{bit}</span>
        {/each}
      {/if}
    </div>
    <div class="value" style="font-size:{fontSize}px">
      {#each globalAnswer.toString().split('') as bit}
        <span class="numberUp">{bit}</span>
      {/each}
    </div>
  </div>
  <div class="buttons">
    {#each buttons as buttonRow, rIndex}
      {#each buttonRow as button, bIndex}
        {#if button !== null}
          <Button
            shape="circle"
            color="light"
            size="lg"
            delay={0}
            className="r-{rIndex} b-{bIndex}"
            on:click={() => {
              click(button)
            }}>
            {button}
          </Button>
        {:else}
          <span class="empty" />
        {/if}
      {/each}
    {/each}
  </div>
</div>
