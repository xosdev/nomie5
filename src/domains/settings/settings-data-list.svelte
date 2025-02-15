<script lang="ts">
  import NIcon from "@/components/icon/icon.svelte";
  import { switchStorage, useStorageSelectMenu } from "./settings-functions";
  import { LastUsed } from "../../store/last-used";
  import ToggleSwitch from "../../components/toggle-switch/toggle-switch.svelte";

  import { UserStore } from "../../store/user-store";
  import Text from "@/components/text/text.svelte";
  import ListItem from "@/components/list-item/list-item.svelte";
  import { Lang } from "../../store/lang";
  import LocalstorageOptions from "@/components/storage/localstorage.svelte";
  import PouchDBOptions from "@/components/storage/pouchdb.svelte";

  import List from "@/components/list/list.svelte";
  import { ApiStore } from "../api/api-store";
  import { navigate } from "svelte-routing";
  import { ChevronRight } from "svelte-hero-icons";
  import { deleteEverything } from "./settings-functions";
  import { createEventDispatcher } from "svelte";

  import FirebaseSettings from "../firebase/firebase-settings.svelte";

  // let fileInput
  // let showImporter = false

  const dispatch = createEventDispatcher();

  const showStorageOptions = () => {
    useStorageSelectMenu({
      current: $UserStore.storageType,
      onSelect(storage: any) {
        switchStorage(storage);
      },
    });
  };
</script>

<List
  className="mb-3"
  title={Lang.t("settings.storage-location", "Storage Location")}
  outside
>
  <ListItem on:click={showStorageOptions} detail>
    <span slot="left">☁️</span>
    <Text>
      {#if $UserStore.storageType === "local"}
        This device only
      {:else if $UserStore.storageType === "pouchdb"}
        {Lang.t("storage.pouchdb", "Local + CouchDB")}
      {:else if $UserStore.storageType === "firebase"}
        Nomie Cloud
      {:else if $UserStore.storageType === "blockstack"}
        {Lang.t("storage.blockstack", "Blockstack")}
      {/if}
    </Text>
    <div slot="right" class="text-sm text-primary-500">
      {Lang.t("general.switch", "Switch")}
    </div>
  </ListItem>
</List>

{#if $UserStore.storageType === "blockstack"}
  Blockstack is no longer available
  <!-- <BlockstackOptions /> -->
{/if}
{#if $UserStore.storageType === "local"}
  <LocalstorageOptions />
{/if}
{#if $UserStore.storageType === "firebase"}
  <FirebaseSettings />
{/if}
{#if $UserStore.storageType === "pouchdb"}
  <PouchDBOptions />
{/if}

<List
  className="mb-3"
  outside
  title={Lang.t("settings.import-data", "Import Data")}
>
  <ListItem
    clickable
    title={Lang.t("settings.nomie-api", "Nomie API")}
    on:click={() => navigate("/api")}
  >
    <span slot="left">🚚</span>
    <span slot="right">
      {#if $ApiStore.items.length}
        <div class="nbtn nbtn-xs nbtn-rounded nbtn-danger">
          {$ApiStore.items.length}
        </div>
      {/if}
      <NIcon name={ChevronRight} className="text-gray-500 text-opacity-80" />
    </span>
  </ListItem>
  <ListItem
    clickable
    title={`${Lang.t("settings.import-from-backup", "Import from Backup")}`}
    on:click={() => {
      dispatch("showImporter");
    }}
  >
    <span slot="left">📦</span>
    <div slot="right">
      <!-- <input
        class="hidden"
        type="file"
        bind:this={fileInput}
        on:change={() => {}} /> -->
      <NIcon name="chevronRight" className="fill-faded-2" />
    </div>
  </ListItem>
</List>
<List className="mb-3" outside>
  <ListItem
    clickable
    title={`${Lang.t("settings.import-from-csv", "Import from CSV ")}`}
    to="/settings/import/csv"
  >
    <span slot="left">📄</span>
    <span slot="right" class="flex">
      <div
        class="bg-red-500 text-white rounded-lg text-sm flex items-center px-2"
      >
        Beta
      </div>
      <NIcon name="chevronRight" className="fill-faded-2" />
    </span>
  </ListItem>
</List>

<List
  className="mb-3"
  outside
  title={Lang.t("settings.export-data", "Export Data")}
>
  <ListItem
    detail
    title={Lang.t("settings.generate-backup", "Generate Backup")}
    to="/settings/export/backup"
  >
    <span slot="left">📦</span>
  </ListItem>
  <ListItem
    detail
    title={Lang.t("settings.generate-csv", "Generate CSV")}
    to="/settings/export/csv"
  >
    <span slot="left">📃</span>
  </ListItem>
</List>

<List
  className="mb-3"
  outside
  title={Lang.t("settings.miscellaneous", "Miscellaneous")}
>
  <ListItem
    clickable
    title={Lang.t("settings.update-last-used-date", "Update Last-Used Dates")}
    on:click={LastUsed.updateAll}
  >
    <span slot="left">🕰</span>
  </ListItem>

  <ListItem
    title={Lang.t("settings.hide-backup-reminder", "Hide backup reminder")}
  >
    <span slot="left">📕</span>
    <div slot="right">
      <ToggleSwitch
        bind:value={$UserStore.meta.hideBackup}
        on:change={() => {
          alert("Make this work");
        }}
      />
    </div>
  </ListItem>

  <ListItem
    detail
    title={Lang.t("general.browse-files", "Browse My Data Files...")}
    on:click={() => {
      navigate("/files");
    }}
  >
    <span slot="left">📂</span>
  </ListItem>
</List>

<List
  className="mb-3"
  outside
  title={Lang.t("settings.danger-zone", "Danger Zone")}
>
  <ListItem detail on:click={deleteEverything} clickable>
    <div slot="left">⚠️</div>
    <Text className="text-red">
      {Lang.t("settings.destroy-all-data", "Destroy all Data")}
    </Text>
  </ListItem>
</List>
