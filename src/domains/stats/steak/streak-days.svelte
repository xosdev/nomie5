<script lang="ts">
  import dayjs from 'dayjs'
  import Icon from '../../../components/icon/icon.svelte'

  import Text from '../../../components/text/text.svelte'
  import appConfig from '../../../config/appConfig'
  import type NLog from '../../../modules/nomie-log/nomie-log'

  export let logs: Array<NLog>
  export let date = dayjs()
  export let days: number = 7
  export let color: string = appConfig.primary_color
  export let size: number = 24

  let loopOver: Array<any> = []

  $: if (days || date) {
    loopOver = Array(days)
      .fill(0)
      .map((d, i) => {
        let loopDate = date.subtract(i, 'day')
        return {
          date: loopDate,
          used: logs.find(
            (nlog: NLog) =>
              nlog.endDayjs().format('YYYY-MM-DD') ===
              loopDate.format('YYYY-MM-DD'),
          )
            ? true
            : false,
        }
      })
      .reverse()
  }
</script>

<style>
  .streak-days .date {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
  .streak-days .date .ball {
    border-radius: 50px;
    margin-bottom: 2px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
</style>

<div class="streak-days">
  <div class="flex">
    {#each loopOver as loopDate}
      <div class="date {loopDate.used ? 'used' : 'not-used'}">
        <div
          class="ball"
          style="{loopDate.used ? `background-color:${color};` : `border: solid 1px ${color};`}
          height:{size}px; width:{size}px; ">
          {#if loopDate.used}
            <Icon name="checkmark" className="fill-white" size={size * 0.6} />
          {/if}
        </div>
        <Text size="xxs" faded={!loopDate.used}>
          {loopDate.date.format('MM/DD')}
        </Text>
      </div>
    {/each}
  </div>
</div>
