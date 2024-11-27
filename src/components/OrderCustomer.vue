<template>
  <div
    class="bg-whitex q-pb-md"
    :class="{
      'bg-mydark ': $q.dark.mode,
      'bg-white ': !$q.dark.mode,
    }"
  >
    <q-card class="no-shadow">
      <q-card-section>
        <!-- <p class="font11 no-margin">Customer Details</p> -->
        <div class="row justify-between">
          <div>{{ $t("Customer Details") }}</div>
          <div>#{{ data.order_id }}</div>
        </div>

        <div class="row items-center">
          <div class="col">
            <p class="text-weight-medium no-margin">{{ data.full_name }}</p>
            <p class="font11 no-margin">
              <template v-if="order_meta[data.order_id]">
                <template v-if="order_meta[data.order_id].address1">{{
                  order_meta[data.order_id].address1
                }}</template>
              </template>
              {{ data.address }}
            </p>
            <p v-if="order_meta[data.order_id]" class="text-weight-medium">
              {{ order_meta[data.order_id].address_label }}
            </p>
          </div>
          <div class="col-4 text-right">
            <div class="flex items-center justify-end">
              <q-btn
                round
                color="primary"
                icon="las la-map"
                size="sm"
                unelevated
                flat
                class="q-mr-sm"
                to="/home/maps"
              />
              <template v-if="order_meta[data.order_id]">
                <q-btn
                  :href="'tel:' + order_meta[data.order_id].contact_number"
                  round
                  color="mygreen"
                  icon="las la-phone-volume"
                  size="sm"
                  unelevated
                  flat
                />
              </template>
            </div>
          </div>
        </div>
        <!-- row -->
      </q-card-section>
    </q-card>

    <q-card
      class="no-shadow card-borderedx no-border-radius"
      :class="{
        '': $q.dark.mode,
        'card-bordered': !$q.dark.mode,
      }"
    >
      <q-card-section>
        <p class="font11 no-margin">{{ $t("Payment") }}</p>
        <div class="row items-center">
          <div class="col">{{ paymentLabel }}</div>
          <div v-if="isCash" class="col text-right text-weight-bold">
            {{ data.total }}
          </div>
        </div>
      </q-card-section>
    </q-card>

    <q-card class="no-shadow no-border-radius">
      <q-card-section v-if="order_meta[data.order_id]">
        <p class="font11 no-margin">{{ $t("Drop-off instructions") }}</p>
        <p class="no-margin">
          <span class="text-weight-bold"
            >{{ $t("Aparment, suite or floor") }} :</span
          >
          {{ order_meta[data.order_id].location_name }}
        </p>
        <p class="no-margin">
          <span class="text-weight-bold">{{ $t("Delivery options") }} :</span>
          {{ order_meta[data.order_id].delivery_options }}
        </p>
        <p class="no-margin">
          <span class="text-weight-bold"
            >{{ $t("Delivery instructions") }} :</span
          >
          {{ order_meta[data.order_id].delivery_instructions }}
        </p>
      </q-card-section>
    </q-card>

    <q-card
      class="no-shadow card-borderedx no-border-radius"
      :class="{
        '': $q.dark.mode,
        'card-bordered': !$q.dark.mode,
      }"
    >
      <q-card-section class="text-center">
        <p class="font12">{{ $t("Estimated Arrival") }}</p>
        <!-- map -->
        <div class="map medium bg-grey-1 relative-position">
          <div
            class="q-mr-sm absolute-bottom-right q-ma-md"
            style="z-index: 99"
          >
            <launchNavigation :location="center"> </launchNavigation>
          </div>
          <MapComponents
            class="maps"
            :keys="Activity.maps_config.key"
            :provider="Activity.maps_config.provider"
            :zoom="Activity.maps_config.zoom"
            :center="center"
            :markers="markers"
          >
          </MapComponents>
        </div>
        <!-- map -->

        <q-space class="q-pa-sm"></q-space>

        <q-list bordered separator class="radius8">
          <q-slide-item
            @left="changeStatus"
            left-color="light-green"
            class="radius8"
          >
            <template v-slot:left>
              <q-spinner color="primary" size="2em" />
            </template>
            <q-item
              class="text-white text-weight-bold btn-11"
              :style="`background-color:${data.delivery_steps.status_data.bg_color} !important;color:${data.delivery_steps.status_data.font_color} !important;`"
            >
              <q-item-section class="text-center font17">{{
                data.delivery_steps.label
              }}</q-item-section>
              <q-item-section avatar>
                <q-avatar
                  text-color="white"
                  :style="`color:${data.delivery_steps.status_data.font_color} !important;`"
                  icon="las la-angle-double-right"
                />
              </q-item-section>
            </q-item>
          </q-slide-item>
        </q-list>
      </q-card-section>
    </q-card>
  </div>
  <!-- white -->
</template>

<script>
import { defineAsyncComponent } from "vue";
import jwtDecode from "jwt-decode";
import APIinterface from "src/api/APIinterface";
import { useLocationStore } from "stores/LocationStore";
import { useActivityStore } from "stores/ActivityStore";

export default {
  name: "OrderCustomer",
  props: ["order_uuid", "merchants", "data", "order_meta", "payment_list"],
  components: {
    MapComponents: defineAsyncComponent(() =>
      import("components/MapComponents.vue")
    ),
    launchNavigation: defineAsyncComponent(() =>
      import("components/launchNavigation.vue")
    ),
  },
  setup() {
    const LocationStore = useLocationStore();
    const Activity = useActivityStore();
    return { LocationStore, Activity };
  },
  data() {
    return {
      loading: false,
      center: [],
      markers: {},
      reset: undefined,
    };
  },
  watch: {
    data(newdata, oldata) {
      this.getCoordinates();
    },
  },
  created() {
    this.getCoordinates();
  },
  computed: {
    paymentLabel() {
      const payment_type = this.payment_list[this.data.payment_code]
        ? this.payment_list[this.data.payment_code]
        : 0;
      return payment_type == 1
        ? this.$t("Paid online")
        : this.$t("Collect Cash");
    },
    isCash() {
      const payment_type = this.payment_list[this.data.payment_code]
        ? this.payment_list[this.data.payment_code]
        : 0;
      return payment_type == 1 ? false : true;
    },
  },
  methods: {
    getCoordinates() {
      console.debug("getCoordinates");
      const $location = this.order_meta[this.data.order_id]
        ? this.order_meta[this.data.order_id]
        : false;

      let $icon = "";
      try {
        const $storage = APIinterface.getStorage("driver_maps_config");
        const $decoded = jwtDecode($storage);
        $icon = $decoded.icon;
      } catch (err) {
        console.debug(err);
      }

      if ($location) {
        this.center = {
          lat: parseFloat($location.latitude),
          lng: parseFloat($location.longitude),
        };
        this.markers = [
          {
            lat: parseFloat($location.latitude),
            lng: parseFloat($location.longitude),
            label: APIinterface.getIcon("customer"),
            icon: "marker_icon_destination",
          },
        ];
        if (this.LocationStore.hadData()) {
          this.markers.push({
            lat: parseFloat(this.LocationStore.coordinates.lat),
            lng: parseFloat(this.LocationStore.coordinates.lng),
            label: APIinterface.getIcon("driver"),
            icon: "marker_icon_rider",
          });
        }
      }
    },
    changeStatus(reset) {
      this.reset = reset;
      this.changeOrderStatus(this.data.delivery_steps.methods);
    },
    changeOrderStatus(methods) {
      this.loading = true;
      APIinterface.showLoadingBox("", this.$q);
      APIinterface.fetchDataByToken(methods, {
        order_uuid: this.order_uuid,
      })
        .then((result) => {
          this.reset.reset();
          this.$emit("afterChangestatus", result.details);
        })
        .catch((error) => {
          console.debug(error);
          this.reset.reset();
          APIinterface.notify("red-5", error, "error_outline", this.$q);
        })
        .then((data) => {
          APIinterface.hideLoadingBox(this.$q);
          this.loading = false;
        });
    },
  },
};
</script>
