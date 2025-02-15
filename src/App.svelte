<script lang="ts">
  import StatsModal from "./domains/stats/stats-modal.svelte";
  import { MessageStore } from "./domains/messages/MessageStore";
  import { StatsStore } from "./domains/stats/StatsStore";
  import UpdateAvailable from "./components/update-available/update-available.svelte";
  import { wait } from "./utils/tick/tick";
  // Svelte
  // import { Router, Route, navigate } from "svelte-routing";
  import Tailwindcss from "./style/Tailwind.svelte";
  import { onMount } from "svelte";
  import dayjs from "dayjs";

  // Vendors
  import Spinner from "./components/spinner/spinner.svelte";
  import { gestures } from "@composi/gestures";

  // Containers
  import Interactions from "./domains/interactions/interactions.svelte";
  import WhatsNewModal from "./domains/whats-new/whats-new-modal.svelte";

  // Utils
  import Logger from "./utils/log/log";

  import RouterView from "./routes/routes.svelte";

  // Stores
  import { UserStore } from "./store/user-store"; //  user auth and state
  import { Interact } from "./store/interact"; //  global alerts, popmenus, confirms, etc
  // import { BoardStore } from "./store/boards"; // board state  and methods
  import { Device } from "./store/device-store"; // board state  and methods
  import { TrackerStore } from "./store/tracker-store"; // tracker state and methods
  import { TrackerLibrary } from "./store/tracker-library";
  import { CommanderStore } from "./store/commander"; // commander - /?note=hi&lat=35&lng=-81.32

  import { PeopleStore } from "./store/people-store"; // Store for holding People
  import { ContextStore } from "./store/context-store"; // Store for holding Post Context (categories)
  import { DashboardStore } from "./store/dashboard-store"; // Store for holding Post Context (categories)

  import { Locations } from "./store/locations";
  import config from "./config/appConfig";
  import { OfflineQueue } from "./store/offline-queue-store";
  import { LastUsed } from "./store/last-used";
  import { SearchStore } from "./store/search-store";
  import tick from "./utils/tick/tick";
  import { LedgerStore } from "./domains/ledger/LedgerStore";
  import ProgressBar from "./components/progress-bar/progress-bar.svelte";
  import { ApiStore } from "./domains/api/api-store";

  import DynamicPage from "./DynamicPage.svelte";
  import Backdrop from "./components/backdrop/backdrop.svelte";
  import PersonModal from "./domains/people/person-modal.svelte";
  import Confetti from "./components/confetti/confetti.svelte";
  import StreakModal from "./domains/stats/steak/streak-modal.svelte";

  import {
    FirebaseStore,
    usePlanModal,
  } from "./domains/firebase/FirebaseStore";
  import FirebaseAccountModal from "./domains/firebase/firebase-account-modal.svelte";
  import FirebasePlanModal from "./domains/firebase/firebase-plan-modal.svelte";

  import "./style/main.css";
  // Set a better console
  const console = new Logger("APP");

  const [showPlanModal, dismissPlanModal] = usePlanModal();

  gestures();

  /**
   * Day / Time Change Monitoring
   * Fire off the MinuteChecker30 every 30 minutes
   * This will check if the day changed
   */
  let todayCheckPeriod = 1000 * 60 * 10;
  let todayCheckFormat = "YYYY-MM-DD";
  let todayKey = dayjs().format(todayCheckFormat);
  let newDay = false; // View reacts to this value

  // Check every X minutes
  setInterval(() => {
    // Get now key
    let checkKey = dayjs().format(todayCheckFormat);
    // Compare now key to today key
    if (todayKey !== checkKey) {
      // It's new - trigger some reactions
      newDay = true;
      // Show toast notification
      Interact.toast(`It's ${dayjs().format("dddd")}!`);
      // Set today key to check key
      todayKey = checkKey;
      LedgerStore.getToday();
      // Wait 500 ms
      setTimeout(() => {
        newDay = false;
      }, 500);
    }
    // Check if the theme has Changed
    methods.setDocParams({});
  }, todayCheckPeriod);

  // This should be reworked
  $: if (window && $TrackerStore && !window["$TrackerStore"]) {
    window["$TrackerStore"] = $TrackerStore;
  }

  // Offline monitor

  const methods = {
    hideSplashScreen() {
      document.querySelectorAll(".delete-on-app").forEach((d) => {
        d.classList.add("deleted");
        setTimeout(() => {
          d.remove();
        }, 500);
      });
    },
    setDocParams(options = {}) {
      let isDarkMode = window.matchMedia(
        "(prefers-color-scheme: dark)"
      ).matches;
      let theme = localStorage.getItem(config.theme_key) || "auto";
      document.documentElement.className = "";
      if (theme === "auto" && isDarkMode) {
        document.documentElement.classList.add("mode-dark");
      } else if (theme === "auto") {
        document.documentElement.classList.add("mode-light");
      } else {
        document.documentElement.classList.add(`mode-${theme}`);
      }
      tick(100, methods.hideSplashScreen);
    },
  };

  /**
   * App to Forground
   *
   * Document Change State Monitoring
   * In hopes of triggering events when the
   * state of the window changes be it from
   * the browser, or switching apps on a phone
   *
   * it kinda works.
   */
  // let hidden, visibilityChange, router;
  // if (typeof document.hidden !== "undefined") {
  //   hidden = "hidden";
  //   visibilityChange = "visibilitychange";
  // } else if (typeof document["msHidden"] !== "undefined") {
  //   hidden = "msHidden";
  //   visibilityChange = "msvisibilitychange";
  // } else if (typeof document["webkitHidden"] !== "undefined") {
  //   hidden = "webkitHidden";
  //   visibilityChange = "webkitvisibilitychange";
  // }

  window.addEventListener("load", () => {
    methods.setDocParams();
  });

  let ready = false;

  // Used to make sure that boards and trackers are loaded
  UserStore.onReady(async () => {
    console.log(
      `🌳🌳🌳🌳🌳 Welcome to NOMIE ${
        import.meta.env.PACKAGE_VERSION
      } 🌳🌳🌳🌳🌳`,
      "👋"
    );
    // Set the user if they're logged in
    ready = true;
    PeopleStore.init(); // Initialize the People Store
    Locations.init(); // Initialize Location Store
    ContextStore.init(); // check if this is a new version
    DashboardStore.init(); // Initilize Dashboards
    Device.init(); // Initialize Device
    LastUsed.init();
    SearchStore.init();

    await wait(500);
    MessageStore.loadMessages();
    CommanderStore.run();
    ApiStore.init();
  });

  // Initialize Offline Queue regardless if we're offline
  OfflineQueue.init();

  $: if (
    $UserStore.storageType === "firebase" &&
    $FirebaseStore &&
    $FirebaseStore.user &&
    !$FirebaseStore.plan &&
    !$FirebaseStore.showPlan
  ) {
    showPlanModal();
  } else if (
    $UserStore.storageType !== "firebase" &&
    $FirebaseStore &&
    $FirebaseStore.showPlan === true
  ) {
    dismissPlanModal();
  }

  onMount(() => {
    UserStore.initialize();
  });
</script>

<Tailwindcss />

{#if !ready && $UserStore.signedIn === false}
  <DynamicPage route="setup" />
{/if}

<!-- {#if ready} -->
{#if $UserStore.signedIn === true && !newDay}
  <RouterView />
  <WhatsNewModal />
{:else if $UserStore.signedIn == undefined || newDay}
  <div class="empty-notice h-screen flex items-center justify-center">
    <Spinner />
  </div>
{/if}

<!-- Global Modals, alerts, menus, etc-->
{#if ready && $StatsStore.trackables.length > 0}
  <!-- <DynamicPage container="stats/stats-modal" /> -->
  <StatsModal />
{/if}

<!-- {#if ready && $TrackerLibrary.show}
  <DynamicPage container="library/library" />
{/if} -->

{#if $UserStore.storageType === "firebase" && $FirebaseStore && $FirebaseStore.showLogin}
  <FirebaseAccountModal />
{/if}

{#if $UserStore.storageType === "firebase" && $FirebaseStore && $FirebaseStore.showPlan}
  <FirebasePlanModal />
{/if}

{#if ready && $Interact.people.active}
  <!-- <DynamicPage container="people/person-modal" /> -->
  <PersonModal />
{/if}

<Backdrop id="blocker" visible={$Interact.blocker.show}>
  {#if $Interact.blocker.show}
    <div style="min-width:200px;">
      <div class="my-3 ml-2 text-center text-white">
        {#if !$Interact.blocker.percent}
          <Spinner size={32} />
        {/if}
        {$Interact.blocker.message}
      </div>
      {#if $Interact.blocker.percent}
        <ProgressBar percentage={$Interact.blocker.percent} />
      {/if}
    </div>
  {/if}
</Backdrop>

<Interactions />

{#if $Interact.streak.show}
  <StreakModal />
{/if}

{#if $Interact.onThisDay}
  <DynamicPage container="on-this-day/on-this-day" />
{/if}

{#if $Interact.pin.show}
  <DynamicPage container="pin-lock/pin-lock" />
{/if}

{#if $SearchStore.show}
  <DynamicPage container="search/search" />
{/if}

<UpdateAvailable />

{#if $Interact.focusedEditor}
  <DynamicPage component="capture-log/focused" />
{/if}

{#if $Interact.confetti.show}
  <Confetti />
{/if}

<div id="photo-holder" class="hidden" style="display:none">
  <img id="photo-holder-image " alt="avatar-holder" />
</div>

<canvas id="confetti" />
