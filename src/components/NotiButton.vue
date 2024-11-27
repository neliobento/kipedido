<template>
  <q-btn
    to="/account/notifications"
    flat
    round
    dense
    icon="las la-bell"
    color="amber-5"
  >
    <transition
      v-if="hasData"
      appear
      enter-active-class="animated zoomIn"
      leave-active-class="animated zoomOut"
    >
      <q-badge floating color="red" rounded style="top: 2px" />
    </transition>
  </q-btn>

  <Realtime
    ref="realtime"
    getevent="notifications"
    @after-receive="afterReceive"
  />
</template>

<script>
import { defineAsyncComponent } from "vue";

export default {
  name: "NotiButton",
  components: {
    Realtime: defineAsyncComponent(() => import("src/components/RealTime.vue")),
  },
  data() {
    return {
      data: [],
    };
  },
  computed: {
    hasData() {
      if (Object.keys(this.data).length > 0) {
        return true;
      }
      return false;
    },
  },
  methods: {
    afterReceive(data) {
      console.debug("afterReceivex");
      this.data = data;
      console.debug(this.data);
      const $notification_type = this.data.notification_type;
      if ($notification_type == "assign_order") {
        const $message = JSON.parse(this.data.message);
        console.debug($message.order_uuid);
        if ($message.order_uuid) {
          this.$router.push({
            path: "/order/new",
            query: { order_uuid: $message.order_uuid },
          });
        }
      } else if ($notification_type == "order_update") {
        if (this.data.meta_data) {
          if (this.data.meta_data.order_uuid) {
            if (this.data.meta_data.status == "new") {
              this.$router.push({
                path: "/order/new",
                query: { order_uuid: this.data.meta_data.order_uuid },
              });
            }
          }
        }
      }
    },
  },
};
</script>
