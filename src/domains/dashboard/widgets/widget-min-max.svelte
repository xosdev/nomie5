<script lang="ts">
  import type { Widget } from "../../../modules/dashboard/widget";
  import { formatValue } from "../dashboard-helpers";
  import Text from "../../../components/text/text.svelte";
  import { UserStore } from "../../../store/user-store";
  import dayjs from "dayjs";
  import Icon from "../../../components/icon/icon.svelte";
  export let widget: Widget;

  $: dateFormat = $UserStore.meta.is24Hour ? "ddd D MMM YYYY" : "ddd MMM D YYYY";
</script>

<div class="value min-max">
  <div class="max text-center">
    <Text size={formatValue(widget.stats.max.value, widget).toString().length > 5 ? 'lg' : 'xl'} bold>
      {formatValue(widget.stats.max.value, widget)}
    </Text>
    <Text size="xs" faded style="margin-left:-12px;">
      <Icon name="chevronUp" size={12} />
      {dayjs(widget.stats.max.date).format(dateFormat)}
    </Text>
  </div>
  <div class="min text-center">
    <!-- Max is set on purpose here.. for consistency -->
    <Text size={formatValue(widget.stats.max.value, widget).toString().length > 5 ? 'lg' : 'xl'} bold>
      {formatValue(widget.stats.min.value, widget)}
    </Text>
    <Text size="xs" faded style="margin-right:-12px;">
      {dayjs(widget.stats.min.date).format(dateFormat)}
      <Icon name="chevronDown" size={12} />
    </Text>
  </div>
</div>

<style global lang="postcss">
  .min-max {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
  .max {
    padding-bottom: 6px;
  }
  .widget-size-lg .min-max {
    font-size: 1.4em;
  }
  .widget-size-lg .min-max {
    flex-direction: row;
  }
  .widget-size-lg .min-max .min {
    padding-bottom: 6px;
    padding-left: 12px;
    border-left: solid 1px var(--color-solid-2);
    margin-left: 12px;
  }

  .widget-size-sm .n-text.xl {
    font-size: 1.4em;
  }
  .widget-size-md .min-max {
    flex-direction: row;
  }
  .widget-size-md .min-max .min {
    padding-bottom: 6px;
    padding-left: 12px;
    border-left: solid 1px var(--color-solid-2);
    margin-left: 12px;
  }
</style>
