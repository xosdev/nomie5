<script lang="ts">
  import TrackerSmallBlock from "../../components/tracker-small-block/tracker-small-block.svelte";
  import Text from "../../components/text/text.svelte";
  import type { Widget } from "../../modules/dashboard/widget";
  import { createEventDispatcher } from "svelte";
  import Icon from "../../components/icon/icon.svelte";
  import Button from "../../components/button/button.svelte";
  import nid from "../../modules/nid/nid";

  // Widgets
  import WidgetWhatTime from "./widgets/widget-what-time.svelte";
  import WidgetLastUsed from "./widgets/widget-last-used.svelte";
  import WidgetBarChart from "./widgets/widget-bar-chart.svelte";
  import WidgetValue from "./widgets/widget-value-display.svelte";
  import WidgetNote from "./widgets/widget-note.svelte";
  import WidgetMinMax from "./widgets/widget-min-max.svelte";
  import WidgetPositivityPie from "./widgets/widget-positivity-pie.svelte";
  import WidgetMap from "./widgets/widget-map.svelte";
  import WidgetStreak from "./widgets/widget-streak.svelte";

  import { Interact } from "../../store/interact";
  import { DashboardStore } from "../../store/dashboard-store";
  import Spinner from "../../components/spinner/spinner.svelte";
  import { Lang } from "../../store/lang";
  import TrackerInputer from "../../modules/tracker/tracker-inputer";
  import { TrackerStore } from "../../store/tracker-store";
  import { ActiveLogStore } from "../../store/active-log";
  import NLog from "../../modules/nomie-log/nomie-log";

  import type TrackableElement from "../../modules/trackable-element/trackable-element";

  const dispatch = createEventDispatcher();
  const id = nid();
  export let widget: Widget;

  function getLabel(element: TrackableElement) {
    if (element.type == "person") {
      return element.obj ? element.obj.username : element.id;
    } else if (element.type == "tracker") {
      return element.obj ? element.obj.label : element.id;
    } else {
      return element.id;
    }
  }

  function widgetActions() {
    const addOn = {
      icon: "tracker",
      title: ` ${
        widget.element.type == "person"
          ? Lang.t("people.check-in", `Check-in`)
          : Lang.t("general.track-value", `Track Value`)
      }`,
      click: async () => {
        if (widget.element.type == "person") {
          Interact.person(widget.element.id);
        } else if (widget.element.type == "tracker") {
          const input = new TrackerInputer(
            widget.element.obj,
            $TrackerStore.trackers
          );
          let note = await input.getNoteString();
          ActiveLogStore.journal(new NLog({ note }));
        }
        // Interact.openStats(widget.element.toSearchTerm());
      },
    };
    const buttons: Array<any> = Interact.getElementOptionButons(widget.element);
    buttons[0].divider = "true";
    buttons.unshift(addOn);
    Interact.popmenu({ title: `${getLabel(widget.element)} options`, buttons });
  }

  function getClass(widget: Widget): string {
    let classes = [`type-${widget.type}`];
    let value;
    if (widget.stats) {
      if (widget.type == "last-used") {
        value = widget.stats.daysPast;
      } else {
        value = widget.math !== "sum" ? widget.stats.avg : widget.stats.sum;
      }
    }
    value = value || 0;

    if (widget.compareValue) {
      if (value > widget.compareValue) {
        classes.push(`over widget-${widget.compareOverColor}`);
      } else if (value < widget.compareValue) {
        classes.push(`over widget-${widget.compareUnderColor}`);
      }
    }
    classes.push(`widget-size-${widget.size}`);

    return classes.join(" ");
  }

  function widgetOptionSelected(type: string, widget: Widget) {
    dispatch("action", { type, widget });
  }
</script>

{#if widget && widget.type !== "text"}
  <div class="dashboard-widget {getClass(widget)}" {id}>
    <div class="flex widget-header">
      <TrackerSmallBlock
        xs
        truncate
        novalue
        element={widget.element}
        on:click={() => {
          widgetActions();
        }}
      />
      <div class="filler" />
      <Button
        size="xs"
        color="clear"
        className="p-1"
        on:click={() => {
          DashboardStore.widgetOptions(widget, widgetOptionSelected);
          dispatch("click");
        }}
      >
        <Icon name="settings" style="fill: var(--color-inverse-2)" size={16} />
      </Button>
    </div>
    <div class="widget-main">
      {#if widget.stats}
        {#if ["barchart", "linechart"].indexOf(widget.type) > -1 && widget.stats && widget.stats.chart}
          <WidgetBarChart {widget} />
        {:else if widget.type == "value" && widget.stats}
          <WidgetValue {widget} />
        {:else if widget.type == "note" && widget.stats}
          <WidgetNote {widget} />
        {:else if widget.type == "what-time"}
          <WidgetWhatTime {widget} />
        {:else if widget.type == "last-used"}
          <WidgetLastUsed {widget} />
        {:else if widget.type == "positivity" && widget.stats}
          <WidgetPositivityPie {widget} />
        {:else if widget.type == "min-max" && widget.stats}
          <WidgetMinMax {widget} />
        {:else if widget.type == "map" && widget.stats}
          <WidgetMap {widget} />
        {:else if widget.type == "streak" && widget.stats}
          <WidgetStreak {widget} />
        {:else}
          <div class="value">Unknown {widget.type}</div>
        {/if}
      {:else}
        <div class="flex value">
          <Spinner size={24} />
        </div>
      {/if}
    </div>
    <div class="flex widget-footer">
      {#if widget.timeRange}
        <Text
          size="xs"
          className="text-center flex-grow text-uppercase font-weight-bold"
        >
          {widget.getLabel()}
        </Text>
      {/if}
    </div>
  </div>
{:else}
  <div
    {id}
    class="dashboard-text type-text text-center widget-size-{widget.size}"
    on:click={() => {
      dispatch("click");
    }}
  >
    {widget.description}
  </div>
{/if}

<style lang="postcss" global>
  .dashboard-text {
    padding: 4px 16px;
    min-width: 100% !important;
    flex-grow: 1;
    display: flex;
    width: 100%;
    font-size: 0.9em;
    color: var(--color-inverse-2);
    align-items: center;
    text-align: center !important;
    justify-content: center;
    max-width: 100% !important;
  }
  .dashboard-text.widget-size-md {
    font-size: 1em;
    font-weight: 500;
    padding: 8px 16px;
    line-height: 1.1em;
  }
  .dashboard-text.widget-size-lg {
    padding: 8px 16px;
    font-size: 1.6em;
    line-height: 1.7em;
    font-weight: 700;
    padding: 16px;
  }
  .dashboard-text + .dashboard-text {
    padding-top: 0px !important;
    padding-bottom: 16px;
  }
  .dashboard-widget {
    max-height: 300px;
    position: relative;
  }
  .dashboard-widget:after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 4px;
  }
  .dashboard-widget.over .widget-footer .n-text {
    color: #fff !important;
  }
  .dashboard-widget.under .widget-footer .n-text {
    color: #fff !important;
  }
  .dashboard-widget.widget-red .widget-footer .n-text {
    color: var(--color-red) !important;
  }
  .dashboard-widget.widget-blue .widget-footer .n-text {
    color: var(--color-blue) !important;
  }
  .dashboard-widget.widget-green .widget-footer .n-text {
    color: var(--color-green) !important;
  }
  .dashboard-widget.widget-orange .widget-footer .n-text {
    color: var(--color-orange) !important;
  }
  .dashboard-widget.widget-red:after {
    background-color: var(--color-red) !important;
  }
  .dashboard-widget.widget-blue:after {
    background-color: var(--color-blue) !important;
  }
  .dashboard-widget.widget-green:after {
    background-color: var(--color-green) !important;
  }
  .dashboard-widget.widget-orange:after {
    background-color: var(--color-orange) !important;
  }
  .dashboard-widget {
    display: flex;
    flex-direction: column;
    @apply bg-white dark:bg-black;
    @apply text-gray-900 dark:text-gray-100;
    @apply shadow-lg;
    @apply rounded-2xl;
    @apply flex-grow flex-shrink;
    @apply overflow-hidden;
  }
  .dashboard-widget .widget-header,
  .dashboard-widget .widget-footer {
    padding: 6px 8px;
    flex-grow: 0;
    flex-shrink: 0;
    min-height: 32px;
  }
  .dashboard-widget .widget-footer {
    margin-top: 8px;
  }
  .dashboard-widget .widget-main {
    height: 90px;
    flex-grow: 1;
  }
  .widget-size-lg {
    @apply col-span-4;
    @apply row-span-2;
  }
  .widget-size-md {
    @apply col-span-4;
    @apply row-span-1;
  }
  .widget-size-sm {
    @apply col-span-2;
    @apply row-span-1;
  }
  .dashboard-widget .value {
    display: flex;
    flex-grow: 1;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    color: var(--color-inverse);
    height: 100%;
    padding: 2px 8px;
  }
  .dashboard-widget .value .current {
    font-size: 2em;
    line-height: 100%;
    text-align: center;
  }
  .dashboard-widget .value.value-sm .current {
    font-size: 1.4em;
  }
  .dashboard-widget .value.value-sm .avg {
    opacity: 0.6;
  }
  .dashboard-widget .value.last-used .current {
    font-size: 1.5em;
  }
</style>
