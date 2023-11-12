<script lang="ts">
  import { useRegisterSW } from "virtual:pwa-register/svelte";
  import { getToastStore } from "@skeletonlabs/skeleton";
  import type { ToastSettings } from "@skeletonlabs/skeleton";

  // replaced dynamically
  const buildDate = __DATE__;

  const { offlineReady, needRefresh, updateServiceWorker } = useRegisterSW({
    onRegistered(r) {
      if (__RELOAD_SW__) {
        r &&
          setInterval(() => {
            console.log("Checking for sw update");
            r.update();
          }, 20000 /* 20s for testing purposes */);
      } else {
        console.log(`SW Registered: ${r}`);
      }
    },
    onRegisterError(error) {
      console.log("SW registration error", error);
    },
  });
  const close = () => {
    offlineReady.set(false);
    needRefresh.set(false);
  };
  const toastStore = getToastStore();

  $: toast = $offlineReady || $needRefresh;
  $: if (toast) {
    if ($offlineReady) {
      const t: ToastSettings = {
        message: "App ready to work offline",
      };
      toastStore.trigger(t);
    } else {
      const t: ToastSettings = {
        message: "New content available, click on reload button to update.",
        action: {
          label: "Reload",
          response: () => updateServiceWorker(true),
        },
      };
      toastStore.trigger(t);
    }
  }
</script>

<div class="pwa-date">
  {buildDate}
</div>

<style>
  .pwa-date {
    visibility: hidden;
  }
</style>
