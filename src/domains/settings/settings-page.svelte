<script lang="ts">
  import { navigate } from "svelte-routing";
  import IonIcon from "../../components/icon/ion-icon.svelte";
  import SettingsTweakList from "./settings-tweak-list.svelte";
  import Container from "../../components/container/container.svelte";
  import ToolbarGrid from "../../components/toolbar/toolbar-grid.svelte";

  import NotificationSolid from "ionicons/dist/svg/notifications.svg?component";

  //Vendors

  import { onMount } from "svelte";

  import SocialShare from "../../modules/share/share";
  import Storage from "../../modules/storage/storage";
  import { MessageStore } from "../messages/MessageStore";

  // Components
  import ListItem from "../../components/list-item/list-item.svelte";
  import NIcon from "../../components/icon/icon.svelte";
  import NButtonGroup from "../../components/button-group/button-group.svelte";
  import BlockstackOptions from "../../components/storage/blockstack.svelte";
  import LocalstorageOptions from "../../components/storage/localstorage.svelte";
  import PouchDBOptions from "../../components/storage/pouchdb.svelte";
  import Spacer from "../../components/spacer/spacer.svelte";
  import List from "../../components/list/list.svelte";

  import Divider from "../../components/divider/divider.svelte";

  // Containers
  import ImporterModal from "../../domains/importer/importer.svelte";

  import NLayout from "../../domains/layout/layout.svelte";

  // Vendors
  import dayjs from "dayjs";
  import localforage from "localforage";

  // Stores
  import { UserStore } from "../../store/user-store";
  import { LedgerStore } from "../ledger/LedgerStore";
  import { Interact } from "../../store/interact";
  import { TrackerStore } from "../../store/tracker-store";
  import { Lang } from "../../store/lang";
  import { Device } from "../../store/device-store";

  // Config
  import config from "../../config/appConfig";
  // Comtainers

  // Components
  import Text from "../../components/text/text.svelte";
  import Button from "../../components/button/button.svelte";
  import Icon from "../../components/icon/icon.svelte";
  import appConfig from "../../config/appConfig";

  import tick from "../../utils/tick/tick";

  import { LastUsed } from "../../store/last-used";
  import { AppStore } from "../../store/app-store";

  import ToggleSwitch from "../../components/toggle-switch/toggle-switch.svelte";
  import { ApiStore } from "../api/api-store";

  import SettingsDataList from "./settings-data-list.svelte";
  import SettingsAboutList from "./settings-about-list.svelte";
  import SettingsFeaturesList from "./settings-features-list.svelte";

  export const location = undefined;
  export const style = undefined;

  // consts
  // const Export = new Exporter();

  let data = {
    signedIn: false,
    files: [],
    showMassEditor: false,
  };

  $: alwaysLocate = $UserStore.alwaysLocate;

  let ledger = null;
  let trackers = null;
  // let user = null;
  let showImporter = false;

  let st = 0;
  async function specialTap() {
    st = st + 1;
    if (st > 9) {
      methods.unlockFeatures();
    }
  }

  let methods = {
    sign_out() {
      UserStore.signout();
    },
    share() {
      SocialShare(
        `I track my life with LeLo! It's free, private, and you get to design what you track. @LeLoApp`,
        "https://LeLo.nbk.net"
      );
    },
    async unlockFeatures() {
      UserStore.unlockHiddenFeatures();
      Interact.confetti({
        show: true,
        title: "🎁  Patron Only Features Unlocked",
        message: "Hey you! Thank you for your continued support. 💘",
        timeout: 5000,
      });
    },
    async tryPatronPin() {
      let pin = await Interact.inputPin("Patron Key", true);

      if (pin === appConfig.patron_pin) {
        methods.unlockFeatures();
      } else {
        Interact.alert(
          `Invalid Patron Pin`,
          `Please check the code and try again`
        );
      }
    },
    locations() {
      Interact.pickLocation();
    },
    sign_in() {
      UserStore.redirectToSignIn();
    },

    bookAge(date) {
      return dayjs(`${date}-01`).fromNow();
    },
    faq() {
      navigate("/faq");
    },
    shop() {
      navigate("/shop");
    },

    settingChange() {
      UserStore.saveMeta();
    },
  };
  let view = Storage.local.get("settings/view") || "settings";

  const changeView = (v: any) => {
    view = v;
    Storage.local.put("settings/view", v);
    Device.scrollToTop();
  };
  // const setTimeout = setTimeout;
  onMount(() => {
    Device.scrollToTop();
  });
</script>

{#if showImporter}
  <ImporterModal
    visible={showImporter}
    on:dismiss={() => (showImporter = false)}
  />
{/if}

<NLayout pageTitle="Settings">
  <div slot="header">
    <Container>
      <ToolbarGrid>
        <Button
          slot="left"
          type="clear"
          title={Lang.t("general.messages", "Messages")}
          className="relative pl-2 text-primary-600"
          on:click={() => {
            navigate("/messages");
          }}
        >
          <IonIcon
            icon={NotificationSolid}
            className={$MessageStore.unseen
              ? "text-red-500 animate-pulse"
              : "text-gray-500"}
          />
        </Button>
        <h1 class="ntitle">
          {Lang.t("settings.settings", "Settings")}
        </h1>

        <button
          slot="right"
          title="Frequently asked Questions"
          class="pr-4 text-primary-600"
          on:click={methods.faq}
        >
          {Lang.t("general.faq", "FAQ")}
        </button>
      </ToolbarGrid>
    </Container>

    <nav class="px-2 pb-1 n-toolbar">
      <NButtonGroup className="mx-auto" style="max-width:400px;">
        <Button
          className={view == "settings" ? "active" : ""}
          on:click={() => {
            changeView("settings");
          }}
        >
          {Lang.t("general.settings", "Settings")}
        </Button>

        <Button
          className={view == "data" ? "active" : ""}
          on:click={() => {
            changeView("data");
          }}
        >
          {Lang.t("settings.tab-data", "Data")}
          {#if $ApiStore.items.length}
            <div class="notify" />
          {/if}
        </Button>
        <Button
          className={view == "about" ? "active" : ""}
          on:click={() => {
            changeView("about");
          }}
        >
          {Lang.t("settings.tab-about", "About")}
        </Button>
      </NButtonGroup>
    </nav>
  </div>

  <div slot="content" class="pt-2">
    <Container>
      {#if $UserStore.meta}
        <div class="page page-settings">
          <div class="p-0 ">
            {#if $UserStore.meta.hiddenFeatures}
              <!-- <ListItem
              className="mb-3"
              href={appConfig.patreonHome}
              detail
              compact>
              <div slot="left">
                <Icon name="cake" className="fill-primary-bright" size={28} />
              </div>
              <Text bold>
                {Lang.t('settings.patron-official', 'Official Nomie Patron')}
              </Text>
              <div slot="right">
                <Text size="sm">{Lang.t('general.latest', 'Latest')}</Text>
              </div>
            </ListItem> -->
            {:else}
              <!-- <ListItem compact className="mb-3">
              <div slot="left" style="font-size:28px">🎁</div>
              <Text bold>{Lang.t('settings.become-a-patron', 'Become a Patron')}</Text>
              <Text size="sm" leading2 faded>
                {Lang.t('settings.patron-description', 'Starting at $2/mon for early features, exclusive content, and support.')}
              </Text>
              <Row insetLeft>
                <Button
                  areaLabel="Unlock"
                  text
                  size="sm"
                  on:click={() => {
                    methods.tryPatronPin(true);
                  }}
                >
                  {Lang.t('settings.patron-unlock', 'Unlock')}
                </Button>
                <Button
                  ariaLabel="Join Now"
                  text
                  size="sm"
                  click={() => {
                    Device.open(appConfig.patreon);
                  }}
                >
                  {Lang.t('settings.patron-join-now', 'Join Now')}
                </Button>
                <Spacer />
              </Row>
              <div slot="right" />
            </ListItem> -->
            {/if}

            {#if view == "settings"}
              <SettingsFeaturesList />
              <SettingsTweakList />
            {:else if view == "data"}
              <SettingsDataList
                on:showImporter={() => {
                  showImporter = true;
                }}
              />
              <!--
              *******************************************
              DATA VIEW
              *******************************************
            -->

              <!-- <div class="my-2 n-list solo">
              <Text bold className="my-3 mx-3">{Lang.t('general.type', 'Finding old data')}</Text>
              <ListItem bottomLine title="Find Context" on:click={ContextStore.searchForContext}>
                <span slot="left">💬</span>
              </ListItem>
              <ListItem title="Find People" on:click={PeopleStore.searchForPeople}>
                <span slot="left">👨‍👨‍👧‍👧</span>
              </ListItem>
            </div> -->
            {:else if view == "about"}
              <!--
              *******************************************
              ABOUT VIEW 
              *******************************************
            -->
              <SettingsAboutList />
            {/if}
            <!-- END Views -->

            <List className="mt-3">
              <ListItem
                title={Lang.t("general.questions", "Questions")}
                className="mb-2"
                detail
              >
                <span slot="left">🆘</span>
                <div slot="right">
                  <Text size="sm">
                    <a
                      class="nbtn nbtn-xs nbtn-rounded nbtn-dark"
                      href={`mailto:${config.support_email}?subject=LeLo ${
                        import.meta.env.PACKAGE_VERSION
                      } `}
                    >
                      Email
                    </a>
                  </Text>
                </div>
              </ListItem>
            </List>

            <Spacer gap={1} />

            <button
              class="w-full px-4 mt-4 mb-3 text-sm text-center "
              on:click={specialTap}
            >
              <div class="text-sm text-black dark:text-white">
                Version v{import.meta.env.PACKAGE_VERSION}
              </div>
              <div class="text-xs text-gray-800 dark:text-gray-500">
                <strong>NBK NET</strong>, LLC &copy; Copyright 2021 - {dayjs().format(
                  "YYYY"
                )}
              </div>
            </button>
          </div>
          <!-- end container -->
        </div>
      {/if}
    </Container>
  </div>
  <!-- end content slot-->
</NLayout>
