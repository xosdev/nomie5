<script lang="ts">
  import ApiArchives from "./../domains/api/api-archive.svelte";
  import ApiItems from "./../domains/api/api-items.svelte";
  import ApiSettings from "./../domains/api/api-settings.svelte";

  import { onMount } from "svelte";
  import ButtonGroup from "../components/button-group/button-group.svelte";
  import Button from "../components/button/button.svelte";
  import Text from "../components/text/text.svelte";
  import ToolbarGrid from "../components/toolbar/toolbar-grid.svelte";
  import Toolbar from "../components/toolbar/toolbar.svelte";
  import { ApiStore } from "../domains/api/api-store";
  import Layout from "../domains/layout/layout.svelte";
  import { Lang } from "../store/lang";
  import Icon from "../components/icon/icon.svelte";
  import Api from "../domains/api/api.svelte";

  export const location: any = undefined;

  type ViewType = "settings" | "items" | "archives";
  let view: string | ViewType = "settings";
  let title: string = "";

  $: if (view) {
    switch (view) {
      case "settings":
        title = "API Settings";
        break;
      case "items":
        title = "Item Queue";
        break;
      case "archives":
        title = "API Archives";
        break;
    }
  }

  function setView(_view: ViewType) {
    view = _view;
    localStorage.setItem("api-view", view);
    // UserStore.hyperStorage.setItem('api-view', view);
  }

  async function main() {
    view = localStorage.getItem("api-view") || "settings";
  }

  onMount(main);
</script>

<Layout pageTitle={title}>
  <div slot="header">
    <ToolbarGrid>
      <h1 class="ntitle">{title}</h1>
      <div slot="right">
        {#if $ApiStore.registered && !$ApiStore.deviceDisabled}
          <Button icon on:click={ApiStore.getLogs}>
            <Icon name="checkmark" className="text-primary-500" />
          </Button>
        {/if}
      </div>
    </ToolbarGrid>
    {#if !$ApiStore.deviceDisabled}
      <Toolbar>
        <ButtonGroup
          size="sm"
          buttons={[
            {
              active: view == "settings",
              label: Lang.t("general.settings", "Settings"),
              click() {
                setView("settings");
              },
            },
            {
              active: view == "items",
              label: Lang.t("general.new", `New (${$ApiStore.items.length})`),
              click() {
                setView("items");
              },
            },
            {
              active: view == "archives",
              label: Lang.t("general.archives", "Archives"),
              click() {
                setView("archives");
              },
            },
          ]}
        />
      </Toolbar>
    {/if}
  </div>

  <div class="">
    {#if view == "settings"}
      <ApiSettings />
    {:else if view == "items"}
      <ApiItems />
    {:else if view == "archives"}
      <ApiArchives />
    {/if}
  </div>
</Layout>
