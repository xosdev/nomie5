<script lang="ts">
  import { createEventDispatcher } from 'svelte'
  import Icon from '../../components/icon/icon.svelte'

  import ListItem from '../../components/list-item/list-item.svelte'
  import Text from '../../components/text/text.svelte'

  import type { ITracker } from '../../modules/tracker/tracker'
  import { LastUsed } from '../../store/last-used'
  import { LedgerStore } from '../ledger/LedgerStore'

  import ShortcutButton from '../../components/shortcut-button/shortcut-button.svelte'
  import Counter from '../../components/counter/counter.svelte'
  import NomieUOM from '../../utils/nomie-uom/nomie-uom'
  import math from '../../utils/math/math'
  import time from '../../utils/time/time'
  import { UserStore } from '../../store/user-store'
  import TrackerButton from '../../components/classic-button/classic-button.svelte'
  import ScoreTracker from '../../modules/scoring/score-tracker'
  import { Lang } from '../../store/lang'
  import Button from '../../components/button/button.svelte'
  import Avatar from '../../components/avatar/avatar.svelte'
  import is from '../../utils/is/is'

  import { ActiveLogStore } from '../../store/active-log'
  import { TrackerStore } from '../../store/tracker-store'
  import { Interact } from '../../store/interact'

  const dispatch = createEventDispatcher()

  export let trackers: Array<ITracker>
  export let hideMore: boolean = false
  export let hideAdd: boolean = false
  export let view: 'list' | 'detail' | 'button' | string =
    localStorage.getItem('board-view') || 'detail'

  function getLastUsed(tracker) {
    if ($LastUsed[tracker.tag]) {
      return time.fromNow($LastUsed[tracker.tag].date)
    } else {
      return undefined
    }
  }

  function getTotalTaps(tracker): number {
    let values = $LedgerStore.today[tracker.tag]
      ? $LedgerStore.today[tracker.tag].values
      : []
    return values.length
  }

  function getHoursUsed(tracker) {
    if ($LedgerStore.today.hasOwnProperty(tracker.tag)) {
      return $LedgerStore.today[tracker.tag].hours
    } else {
      return []
    }
  }

  function getPositivity(tracker) {
    let value = getTodaysValue(tracker, false)
    value = value || 0
    return ScoreTracker(value, tracker)
  }

  function getTodaysValue(tracker, format: boolean = true): string | number {
    let value = null
    // Does this tracker exist in today's map?
    if ($LedgerStore.today.hasOwnProperty(tracker.tag)) {
      // What type of Math should we do?
      if (tracker.math === 'sum') {
        // Sum it up!
        value = math.round(math.sum($LedgerStore.today[tracker.tag].values))
      } else {
        // Round things!
        value = math.round(math.average($LedgerStore.today[tracker.tag].values))
      }
    }
    // If the value is 0 set it to a string
    // so that svelte reacts the the changes
    if (value === 0) {
      value = `0`
    }
    // Return it processed
    if (format && is.truthy(value)) {
      return value ? NomieUOM.format(value, tracker.uom) : null
    } else {
      return value
    }
  }

  /**
   * Long Press a Tracker
   * If it's a timer, it will toggle the running state.
   * otherwise, it will auto save a log using the trackers default value
   * @param tracker
   */
  async function longPress(tracker) {
    let note: string
    // If it's a timer - then toggle running state
    if (tracker.type == 'timer') {
      if (tracker.started) {
        const age = (new Date().getTime() - tracker.started) / 1000
        note = tracker.toNoteString(age)
        TrackerStore.stopTimer(tracker)
      } else {
        TrackerStore.startTimer(tracker)
      }
      // If itx anything other than a note-type get the value
    } else if (tracker.type !== 'note' && tracker.type !== 'picker') {
      note = tracker.toNoteString()
    }

    // If we have a note
    if (note && note.length) {
      // Add element to Capture Log
      ActiveLogStore.addElement(note)
      // Save it
      await LedgerStore.saveLog(ActiveLogStore.asLog())
      // Clear active log
      await ActiveLogStore.clear()
    } else if (tracker.type !== 'timer') {
      // If it's a note - just open it like normal
      Interact.trackerTap(tracker, $TrackerStore.trackers)
    }
    return false
  }
</script>

<style lang="postcss" global>
  .tracker-list-item.in-note .tracker-label {
    font-weight: bold;
    color: var(--tracker-color);
  }
  .tracker-list-item .highlight {
    position: absolute;
    left: 3px;
    top: 6px;
    bottom: 6px;
    width: 3px;
    border-radius: 2px;
  }
  .tracker-list-item.no-value {
    background-color: transparent;
  }
  .tracker-list-item.no-value .highlight {
    display: none;
  }
</style>

{#if view == 'list'}
  {#each trackers as tracker}
    <ListItem
      id="tracker-{tracker.tag}"
      clickable
      style="--tracker-color:{tracker.color}"
      className="tracker-board-button tracker-{tracker.tag} py-2
      tracker-list-item flex-shrink-off {is.truthy(getTodaysValue(tracker)) ? 'has-value' : 'no-value'}"
      compact={$UserStore.localSettings.compactButtons}
      on:longtap={(evt) => {
        longPress(tracker)
      }}
      on:click={(evt) => {
        if (['svg'].indexOf(evt.detail.target.tagName) == -1) {
          dispatch('tap', tracker)
        }
      }}>
      <div class="highlight" style="background-color:{tracker.color}" />
      <div slot="left" class="flex pr-0 justify-content-center">
        <Avatar
          emoji={tracker.emoji}
          label={tracker.label}
          size={$UserStore.localSettings.compactButtons ? 30 : 40} />
      </div>
      <main>
        <h3 class="font-bold text-black dark:text-white line-clamp-1">
          {tracker.label}
        </h3>
        {#if getLastUsed(tracker)}
          <p class="text-xs text-gray-500">
            {getLastUsed(tracker) || undefined}
          </p>
        {/if}
      </main>
      <span slot="right" class="flex items-center mr-2 space-x-4">
        <Text>{getTodaysValue(tracker) || ''}</Text>
        {#if tracker.started}
          <Counter
            started={tracker.started}
            className="text-lg"
            color={tracker.color} />
        {/if}
        {#if !hideMore}
          <Button
            shape="round"
            className="bg-gray-100 dark:bg-black text-primary-500"
            size="sm"
            style="z-index:1000"
            on:click={(evt) => {
              dispatch('more', tracker)
            }}>
            <Icon name="more" size={18} className="prevent fill-inverse-1" />
          </Button>
        {/if}
      </span>
    </ListItem>
  {/each}
  {#if !hideAdd}
    <ListItem
      compact={$UserStore.localSettings.compactButtons}
      clickable
      title={Lang.t('tracker.add-tracker', 'Add Tracker')}
      on:click={() => dispatch('add')}
      className="tracker-add py-3 tracker-list-item flex-shrink-off no-value">
      <div slot="left">
        <Text
          style="width:40px;"
          center
          size={$UserStore.localSettings.compactButtons ? 'xl' : 'xxl'}>
          ➕
        </Text>
      </div>
    </ListItem>
  {/if}
{:else if view === 'detail'}
  <!-- Short Cut Button Style -->

  <div class="item-grid">
    {#each trackers as tracker}
      <ShortcutButton
        id="tracker-{tracker.tag}"
        compact={$UserStore.localSettings.compactButtons}
        title={tracker.label}
        hoursUsed={getHoursUsed(tracker)}
        subtitle={!tracker.started ? getLastUsed(tracker) : null}
        emoji={tracker.emoji}
        value={getTodaysValue(tracker)}
        oneTap={tracker.one_tap}
        color={tracker.color}
        on:longpress={() => {
          longPress(tracker)
        }}
        className="tracker-{tracker.tag} tracker-board-button"
        {hideMore}
        on:click={() => {
          dispatch('tap', tracker)
        }}
        on:more={() => {
          dispatch('more', tracker)
        }}>
        <div slot="subtitle">
          {#if tracker.started}
            <Counter started={tracker.started} color={tracker.color} />
          {/if}
        </div>
      </ShortcutButton>
    {/each}
    {#if !hideAdd}
      <ShortcutButton
        compact={$UserStore.localSettings.compactButtons}
        title={Lang.t('tracker.add-tracker', 'Add Tracker')}
        emoji="➕"
        className="tracker-add"
        hideMore={true}
        on:click={() => {
          dispatch('add')
        }} />
    {/if}
  </div>
{:else if view == 'button'}
  <!-- Class Button Style -->
  <div class="trackers n-grid">
    {#each trackers as tracker}
      <TrackerButton
        {tracker}
        className="tracker-board-button"
        id="tracker-{tracker.tag}"
        value={getTodaysValue(tracker)}
        hoursUsed={getHoursUsed(tracker)}
        positivity={getPositivity(tracker)}
        on:longpress={(evt) => {
          longPress(tracker)
        }}
        on:click={(evt) => {
          dispatch('tap', tracker)
        }}
        on:more={() => {
          dispatch('more', tracker)
        }} />
    {/each}
    {#if !hideAdd}
      <TrackerButton
        hideMore={true}
        tracker={{ tag: 'add', label: `${Lang.t('tracker.add-tracker', 'Add Tracker')}`, emoji: '➕' }}
        on:click={(evt) => {
          dispatch('add', evt)
        }} />
    {/if}
  </div>
{/if}

<!--    className={`${state.addedTrackers.indexOf(tracker.tag) > -1 ? 'added pulse' : ''} ${state.savingTrackers.indexOf(tracker.tag) > -1 ? 'wiggle saving' : ''}`} -->
