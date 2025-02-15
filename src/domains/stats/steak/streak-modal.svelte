<script lang="ts">
  import NModal from '../../../components/modal/modal.svelte'
  import NIcon from '../../../components/icon/icon.svelte'
  import NCalendar from '../../../components/calendar/calendar.svelte'
  import NToolbarGrid from '../../../components/toolbar/toolbar-grid.svelte'

  // Modules and Utils
  import math from '../../../utils/math/math'
  import dayjs from 'dayjs'
  import extractor from '../../../utils/extract/extract'

  // Stores
  import { UserStore } from '../../../store/user-store'
  import { Interact } from '../../../store/interact'
  import { LedgerStore } from '../../ledger/LedgerStore'
  import { TrackerStore } from '../../../store/tracker-store'
  import NextPrevCal from '../../../components/next-prev-cal/next-prev-cal.svelte'
  import Text from '../../../components/text/text.svelte'
  import Button from '../../../components/button/button.svelte'

  const timeFormat = $UserStore.meta.is24Hour ? 'HH:mm' : 'h:mm a'
  const dateFormat = $UserStore.meta.is24Hour ? 'MM/DD/YYYY' : 'MMM D YYYY'
  let _elCalendar

  const state = {
    date: dayjs().startOf('month'),
    logs: [],
    percentage: 0,
    daysHit: 0,
    daysTotal: 0,
    thisMonth: true,
  }

  function next() {
    state.date = state.date.add(1, 'month')
  }

  function prev() {
    state.date = state.date.subtract(1, 'month')
  }

  let lastStreakShow = null
  let trackableElement = null
  let tracker = null

  $: if ($Interact.streak.show && $Interact.streak.show !== lastStreakShow) {
    lastStreakShow = $Interact.streak.show
    trackableElement = extractor.toElement($Interact.streak.show)
    tracker = TrackerStore.getByTag(trackableElement.id)
  }

  function getPercentage(rows) {
    let start = dayjs(state.date).startOf('month')
    let end = dayjs(start).endOf('month')
    if (state.thisMonth) {
      end = dayjs().endOf('day')
    }
    let diff = end.diff(start, 'day') + 1
    let final = []
    for (var i = 0; i < diff; i++) {
      let date = dayjs(start).add(i, 'day')
      let hasEvent = rows.find(
        (row) =>
          new Date(row.end).toDateString() === date.toDate().toDateString(),
      )
      final.push(hasEvent)
    }
    let found = final.filter((r) => r).length
    let total = final.length

    state.daysTotal = total
    state.daysHit = found

    return found / total
  }

  async function main() {
    let payload = {
      start: state.date.startOf('month'),
      end: state.date.endOf('month'),
    }
    let type = extractor.toElement($Interact.streak.show)

    let logs = await LedgerStore.query({
      search: type.toSearchTerm(),
      start: payload.start,
      end: payload.end,
    })

    logs = logs.map((row: any) => {
      row.start = new Date(row.start)
      row.end = new Date(row.end)
      row.repeat = 'never'
      return row
    })
    state.logs = logs
    state.percentage = getPercentage(logs) * 100
  }

  let lastDate
  $: if ($Interact.streak.show && state.date.format('YYYY-MM') !== lastDate) {
    lastDate = state.date.format('YYYY-MM')
    state.thisMonth = lastDate == dayjs().format('YYYY-MM')
    main()
  }
  $: if (!$Interact.streak.show) {
    state.date = dayjs()
    lastDate = null
  }
</script>

<style lang="postcss">
  /* .spinner-container {
    width: 100px;
    height: 100px;
    padding: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: var(--color-solid);
    box-shadow: var(--box-shadow-neu);
    border-radius: 50px;
  } */
</style>

{#if tracker}
  <NModal show={$Interact.streak.show} type="bottom-slideup" ariaLabel="Stats">
    <div slot="header" class="w-100">
      <NToolbarGrid>
        <div slot="left">
          <Button icon className="tap-icon" on:click={Interact.closeStreak}>
            <NIcon name="close" />
          </Button>
        </div>
        <main slot="main">{$Interact.streak.show}</main>
      </NToolbarGrid>
      <div class="flex px-3 n-toolbar">
        <Text bold className="filler">{state.date.format('MMM YYYY')}</Text>
        <NextPrevCal
          on:next={next}
          on:previous={prev}
          hideCal={true}
          style="max-width:80px;" />
      </div>
    </div>
    <div class="p-3">

      <NCalendar
        bind:this={_elCalendar}
        color={tracker.color}
        {tracker}
        showHeader={false}
        on:dayClick={(event) => {
          state.date = dayjs(event.detail)
          main()
        }}
        initialDate={state.date}
        events={state.logs} />

      <div class="n-panel center-all">

        <div class="n-panel w-50 center-all vertical">
          <h1 class="text-inverse">
            {state.daysHit}
            <span class="text-inverse-3">of</span>
            {state.daysTotal}
          </h1>
          <small class="text-inverse-2">
            {math.round(state.percentage, 0)}% of the days
          </small>
        </div>
        <div class="py-2 n-panel w-50 center-all">
          <!-- <div class="spinner-container">
            // TODO: Add circle chart
          </div> -->
        </div>
      </div>
    </div>
  </NModal>
{/if}
