<template>
  <!-- <q-footer bordered class="bg-white text-dark"> -->
  <q-tabs
    v-model="tab"
    dense
    indicator-color="transparent"
    active-color="primary"
    :class="{
      'bg-mydark text-white': $q.dark.mode,
      'text-dark': !$q.dark.mode,
    }"
  >
    <q-route-tab
      name="status"
      icon="las la-check"
      :label="$t('Status')"
      no-caps
      to="/home"
    />
    <q-route-tab
      name="deliveries"
      icon="las la-biking"
      :label="$t('Deliveries')"
      no-caps
      to="/home/deliveries"
      class="routertab_small"
    />
    <q-route-tab
      name="map"
      icon="las la-map-marked"
      :label="$t('Maps')"
      no-caps
      to="/home/maps"
      class="routertab_small"
    />
    <q-route-tab
      name="history"
      icon="las la-history"
      :label="$t('History')"
      no-caps
      to="/home/history"
      class="routertab_small"
    />
    <q-route-tab
      name="wallet"
      icon="las la-wallet"
      :label="$t('Wallet')"
      no-caps
      to="/home/wallet"
      class="routertab_small"
    />
  </q-tabs>
  <!-- </q-footer> -->
</template>

<script>
export default {
  name: "TabMenu",
  data() {
    return {
      tab: "status",
    };
  },
};
</script>
