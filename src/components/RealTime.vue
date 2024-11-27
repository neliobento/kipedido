<template>
  <div style="display: none"></div>
</template>

<script>
import { loadScript } from "vue-plugin-load-script";
import auth from "src/api/auth";
import { useActivityStore } from "stores/ActivityStore";
import APIinterface from "src/api/APIinterface";

export default {
  name: "RealTime",
  setup() {
    const Activity = useActivityStore();
    return { Activity };
  },
  data() {
    return {
      data: [],
      pusher: undefined,
      channel: undefined,
    };
  },
  watch: {
    Activity: {
      immediate: true,
      deep: true,
      handler(newValue, oldValue) {
        if (Object.keys(newValue.realtime_data).length > 0) {
          if (auth.authenticated()) {
            this.data = newValue.realtime_data;
            if (this.data.realtime_app_enabled == 1) {
              this.initProvider();
            }
          }
        }
      },
    },
  },
  methods: {
    initProvider() {
      switch (this.data.realtime_provider) {
        case "pusher":
          loadScript("https://js.pusher.com/7.0/pusher.min.js")
            .then(() => {
              this.initPusher();
            })
            .catch(() => {
              console.debug("failed loading realtime script");
            });
          break;
      }
    },
    initPusher() {
      console.log("initPusher");
      if (APIinterface.empty(this.pusher)) {
        const userData = auth.getUser();
        Pusher.logToConsole = false;
        this.pusher = new Pusher(this.data.pusher_key, {
          cluster: this.data.pusher_cluster,
        });
        this.channel = this.pusher.subscribe(userData.driver_uuid);
        this.channel.bind(this.data.event, (data) => {
          this.$emit("afterReceive", data);
        });
      }
    },
  },
};
</script>
