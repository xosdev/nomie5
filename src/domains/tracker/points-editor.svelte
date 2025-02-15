<script>
  // Svelte
  import { createEventDispatcher } from 'svelte'

  // components
  import NText from '../../components/text/text.svelte'

  import NIcon from '../../components/icon/icon.svelte'
  import NItem from '../../components/list-item/list-item.svelte'
  import NPoints from '../../components/points/points.svelte'
  import NInput from '../../components/input/input.svelte'

  // Modules
  import Tracker from '../../modules/tracker/tracker'
  import PositivityCondition from '../../modules/tracker/positivity-condition'

  // Stores
  import { TrackerStore } from '../../store/tracker-store'
  import { UserStore } from '../../store/user-store'
  import { Lang } from '../../store/lang'
  import { Interact } from '../../store/interact'

  import dayjs from 'dayjs'

  import { Ordinal } from '../../utils/ordinal/ordinal'
  import Button from '../../components/button/button.svelte'
  import SortableList from '../../components/sortable-list/sortable-list.svelte'
  import is from '../../utils/is/is'
  import Icon from '../../components/icon/icon.svelte'
  import ListItem from '../../components/list-item/list-item.svelte'
  import Card from '../../components/card/card.svelte'

  import Row from '../../components/row/row.svelte'
  import Divider from '../../components/divider/divider.svelte'
  import appConfig from '../../config/appConfig'

  // Prosp
  export let tracker = undefined
  export let className = ''

  // consts
  const dispatch = createEventDispatcher()

  // State
  let state = {
    showConditionForm: false,
    genesisCalc: new PositivityCondition(),
    selectedIndex: -1,
  }

  const getTrackerInput = async () => {
    const response = await Interact.trackerInput(tracker, {
      value: state.genesisCalc.v,
      allowSave: false,
    })
    if (response.value) {
      state.genesisCalc.v = response.value
    }
  }

  const scoreOptions = {
    value: 'Recorded Value',
    hour: 'Hour of Day (24)',
    month: 'Day of Month',
    total: "Today's Total",
  }

  const methods = {
    change() {
      dispatch('trackerChange', tracker)
    },
    newCondition() {
      state.showConditionForm = true
      methods.change()
    },
    getDays() {
      let days = []
      for (let i = 0; i < 31; i++) {
        days.push({
          value: Ordinal(i + 1),
          index: i + 1,
        })
      }
      return days
    },
    getHours() {
      let hours = []
      for (let i = 0; i < 24; i++) {
        let date = dayjs().startOf('day').add(i, 'hour')

        hours.push({
          value: $UserStore.meta.is24Hour
            ? date.format('HH:mm')
            : date.format('h:mm a'),
          index: i + 1,
        })
      }
      return hours
    },
    saveCondition() {
      tracker.score_calc = tracker.score_calc || []
      tracker.score_calc.push({ ...state.genesisCalc })
      state.showConditionForm = false
      methods.change()
    },
    async save() {
      await TrackerStore.saveTracker(tracker)
      dispatch('save', tracker)
      Interact.toast(`${tracker.label} saved`)
    },
    removeCondition(index) {
      tracker.score_calc = tracker.score_calc.filter((row, ind) => {
        return ind !== index
      })
      methods.change()
    },
  }
</script>

<style lang="postcss">
  .pos-label {
    font-size: 1.2em;
    display: none;
    min-width: 70px;
    text-transform: uppercase;
    text-align: center;
  }
</style>

{#if tracker}
  <div class={className}>
    <NInput
      listItem
      type="select"
      bind:value={tracker.score}
      on:change={methods.change}
      label={Lang.t('tracker.positivity', 'Positivity')}>
      <option value="0" selected>
        😐 {Lang.t('tracker.neutral', 'Neutral')}
      </option>
      <option value="1">😊 {Lang.t('tracker.positive', 'Positive')}</option>
      <option value="-1">😩 {Lang.t('tracker.negative', 'Negative')}</option>
      <option value="custom">🛠 {Lang.t('general.customize', 'Custom')}</option>
    </NInput>
    {#if tracker.score === 'custom'}
      <section class="points-editor">
        <Card className="m-2">
          <SortableList
            bind:items={tracker.score_calc}
            handle=".menu-handle"
            key="index"
            let:item
            let:index>
            <NItem className="">
              <button
                slot="left"
                class=""
                on:click={() => {
                  methods.removeCondition(index)
                }}>
                <NIcon name="remove" className="text-red-500" />
              </button>
              <!-- {index === 0 ? 'IF:' : 'ELSE:'} -->
              <NText size="sm">
                If
                <span class="px-1 bg-red-500 bg-opacity-50 rounded-md">
                  {scoreOptions[item.if] || 'Unknown'}
                </span>
                is
                <span class="px-1 bg-indigo-500 bg-opacity-50 rounded-md">
                  {item.is}
                </span>
                <span class="px-1 bg-opacity-50 rounded-md bg-primary-500">
                  {item.v}
                </span>
              </NText>

              <div slot="right">
                <NPoints points={item.sc} />
                <div class="px-1 ml-2 menu-handle">
                  <Icon name="menu" size={20} />
                </div>
              </div>

            </NItem>
          </SortableList>
          {#if state.showConditionForm}
            <div
              class="bg-white dark:bg-gray-900"
              style="padding:8px !important">

              <!-- If condition -->
              <NInput
                type="select"
                label="Trigger"
                bind:value={state.genesisCalc.if}>
                <option value="value">
                  {tracker.score_calc || [].length == 0 ? 'If' : 'Else'} Tracked
                  Value
                </option>
                <option value="hour">
                  {tracker.score_calc || [].length == 0 ? 'If' : 'Else'} Hour of
                  Day
                </option>
                <option value="month">
                  {tracker.score_calc || [].length == 0 ? 'If' : 'Else'} Day of
                  Month
                </option>
              </NInput>

              <!-- Greater / Less -->
              <div class="flex mt-1">
                <NInput
                  type="select"
                  label="Comparison"
                  style="width:60%"
                  className="mr-2"
                  bind:value={state.genesisCalc.is}>
                  <option value="gt">is Greater</option>
                  <option value="gte">is Greater or Equal</option>
                  <option value="lt">is Less</option>
                  <option value="lte">is Less or Equal</option>
                  <option value="eq">Equals</option>
                </NInput>

                <!-- Compare Value -->
                {#if state.genesisCalc.if == 'hour'}
                  <NInput
                    type="select"
                    placeholder="Than"
                    bind:value={state.genesisCalc.v}>
                    {#each methods.getHours() as hour}
                      <option value={hour.index}>{hour.value}</option>
                    {/each}
                  </NInput>
                {:else if state.genesisCalc.if == 'month'}
                  <NInput
                    type="select"
                    placeholder="Than"
                    bind:value={state.genesisCalc.v}>
                    {#each methods.getDays() as day}
                      <option value={day.index}>{day.value}</option>
                    {/each}
                  </NInput>
                {:else}
                  <NInput
                    pattern="[0-9]*"
                    style="width:40%;"
                    type="text"
                    label="Than"
                    bind:value={state.genesisCalc.v}>
                    <div slot="right">
                      <Button icon on:click={getTrackerInput}>
                        <NIcon name="addOutline" className="fill-inverse-2" />
                      </Button>
                    </div>
                  </NInput>
                {/if}

              </div>

              <!-- Set Score To -->
              <NInput
                type="select"
                placeholder="Select a Positivity"
                label="Set Score To:"
                bind:value={state.genesisCalc.sc}>
                <option disabled>{Lang.t('general.select', 'Select')}</option>
                {#each appConfig.positivity as score}
                  <option value={score.score}>
                    {score.emoji} {score.score}
                  </option>
                {/each}
              </NInput>

              <!-- Save Bar -->
              <Row className="pt-3 justify-between">
                <Button
                  text
                  size="sm"
                  className="left text-primary-bright"
                  color="transparent"
                  on:click={() => {
                    state.showConditionForm = false
                  }}>
                  {Lang.t('general.cancel', 'Cancel')}
                </Button>

                <Button
                  color="primary"
                  size="sm"
                  className="right stiff"
                  disabled={!is.truthy(state.genesisCalc.sc)}
                  on:click={methods.saveCondition}>
                  {Lang.t('tracker.add-condition', 'Add Rule')}
                </Button>

              </Row>

            </div>
          {/if}
        </Card>
        {#if !state.showConditionForm}
          <Divider center />
          <ListItem on:click={() => (state.showConditionForm = true)}>
            <div slot="left" class="p-0">
              <NIcon name="add" className="text-primary-500" />
            </div>
            <main>
              <div class="text-sm leading-tight text-primary-500">
                {Lang.t('tracker.create-condition', 'New Positivity Rule...')}
              </div>
              <div class="text-xs opacity-50">
                {Lang.t('positivity.sort-message', 'First true wins')}
              </div>
            </main>
          </ListItem>
        {/if}
      </section>
    {/if}
  </div>
{/if}
