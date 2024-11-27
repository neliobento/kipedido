<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent } from "vue";
import { useActivityStore } from "stores/ActivityStore";

export default defineComponent({
  name: "NotopfooterLayout",
  setup() {
    const Activity = useActivityStore();
    return { Activity };
  },
  created() {
    this.$q.dark.set(this.Activity.dark_mode);
  },
});
</script>
