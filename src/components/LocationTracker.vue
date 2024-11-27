<template>
  <div class="hidden">
    =>{{ LocationStore.has_watcher }}
    <pre>{{ LocationStore.watchers }}</pre>
    <pre>{{ LocationStore.coordinates }}</pre>
  </div>
</template>

<script>
import { ref, onMounted } from "vue";
import { useScheduleStore } from "stores/ScheduleStore";
import { useLocationStore } from "stores/LocationStore";
import APIinterface from "src/api/APIinterface";
import AppLocation from "src/api/AppLocation";
import { useRouter, useRoute } from "vue-router";
import { Geolocation } from "@capacitor/geolocation";
import { useQuasar } from "quasar";
// import { Http } from "@capacitor-community/http";
import { CapacitorHttp } from "@capacitor/core";
import { registerPlugin } from "@capacitor/core";
import config from "src/api/config";
import auth from "src/api/auth";
const BackgroundGeolocation = registerPlugin("BackgroundGeolocation");
import { firebaseDb, firebaseCollectionEnum } from "src/boot/firebase";
import {
  doc,
  setDoc,
  addDoc,
  collection,
  onSnapshot,
} from "firebase/firestore";
import { DateTime } from "luxon";

//https://edupala.com/ionic-capacitor-geolocation-for-getting-location-data/

export default {
  name: "LocationTracker",
  setup() {
    const router = useRouter();
    const route = useRoute();
    const $q = useQuasar();

    const Schedule = useScheduleStore();
    const LocationStore = useLocationStore();
    const watchId = ref(undefined);
    const driverInfo = ref(undefined);

    onMounted(() => {
      driverInfo.value = auth.getUser();
      if ($q.capacitor) {
        checkLocationPermission();
      }
    });

    function checkLocationPermission() {
      AppLocation.islocationEnabled()
        .then((data) => {
          // if ($q.capacitor) {
          //   if (!LocationStore.has_watcher) {
          //     initBackground();
          //   }
          // } else {
          //   watchLocation();
          // }
          // getRealtimeLocation();
          if ($q.capacitor) {
            if (!LocationStore.has_watcher) {
              initBackground();
            }
            watchLocation();
          }
        })
        .catch((error) => {
          const $skip = APIinterface.getSession("skip_location");
          if ($skip != 1) {
            router.push({
              path: "/location/permission",
              query: { page: "/home" },
            });
          }
        });
    }

    const initBackground = () => {
      BackgroundGeolocation.addWatcher(
        {
          backgroundMessage: "Location tracking updates.",
          backgroundTitle: "Tracking...",
          requestPermissions: true,
          stale: false,
          distanceFilter: 2,
        },
        (location, error) => {
          if (error) {
            APIinterface.notify("red-5", error, "error_outline", $q);
          }
          LocationStore.coordinates = {
            lat: location.latitude,
            lng: location.longitude,
            accuracy: location.accuracy,
            altitude: location.altitude,
            altitudeAccuracy: location.altitudeAccuracy,
            speed: location.speed,
            bearing: location.bearing,
            time: location.time,
            simulated: location.simulated,
            created_at: APIinterface.getDateTimeNow(),
            driver_id: driverInfo.value.driver_id,
          };
          httpLocation();
          setFirebaseLocation();
          //createFirebaseLocationLogs();
        }
      ).then((watcher_id) => {
        LocationStore.has_watcher = true;
        LocationStore.watchers.push(watcher_id);
      });
    };

    async function watchLocation() {
      console.log("watchLocation");
      let position = await Geolocation.getCurrentPosition();
      if (position) {
        console.debug(position);
        LocationStore.coordinates = {
          lat: position.coords.latitude,
          lng: position.coords.longitude,
          accuracy: position.coords.accuracy,
          altitude: position.coords.altitude,
          altitudeAccuracy: position.coords.altitudeAccuracy,
          speed: position.coords.speed,
          bearing: "",
          time: position.timestamp,
          simulated: false,
          created_at: APIinterface.getDateTimeNow(),
          driver_id: driverInfo.value.driver_id,
        };
        httpLocation();
        setFirebaseLocation();
      }
    }

    const clearWatch = () => {
      Geolocation.clearWatch({ id: watchId.value })
        .then((result) => {
          console.log("result of clear is", result);
        })
        .catch((error) => {
          console.log("clearWatch error", error);
        });
    };

    const stopWacthers = () => {
      if (Object.keys(LocationStore.watchers).length > 0) {
        Object.entries(LocationStore.watchers).forEach(([key, items]) => {
          BackgroundGeolocation.removeWatcher({
            id: items,
          });
          LocationStore.watchers = [];
          LocationStore.has_watcher = false;
        });
      }
    };

    const httpLocation = async () => {
      try {
        const options = {
          url: config.api_base_url + "/updateLocation",
          headers: { Authorization: "token " + auth.getToken() },
          params: {
            latitude: String(LocationStore.coordinates.lat),
            longitude: String(LocationStore.coordinates.lng),
            accuracy: String(LocationStore.coordinates.accuracy),
            altitude: String(LocationStore.coordinates.altitude),
            altitudeAccuracy: String(
              LocationStore.coordinates.altitudeAccuracy
            ),
            speed: String(LocationStore.coordinates.speed),
            bearing: String(LocationStore.coordinates.bearing),
            time: String(LocationStore.coordinates.time),
            simulated: String(LocationStore.coordinates.simulated),
          },
        };
        let HttpResponse = await CapacitorHttp.get(options);
      } catch (err) {
        console.log(err);
      }
    };

    const setFirebaseLocation = async () => {
      console.debug("setFirebaseLocation");
      console.debug(driverInfo.value);
      const docRef = doc(
        firebaseDb,
        firebaseCollectionEnum.driver,
        driverInfo.value.driver_uuid
      );
      await setDoc(docRef, LocationStore.coordinates);
    };

    const createFirebaseLocationLogs = async () => {
      console.debug("createFirebaseLocationLogs");
      let $date_now = APIinterface.getDateNow();
      const docRef = await addDoc(
        collection(
          firebaseDb,
          firebaseCollectionEnum.driver_logs,
          driverInfo.value.driver_uuid,
          $date_now
        ),
        LocationStore.coordinates
      );
    };

    const getRealtimeLocation = async () => {
      console.log("getRealtimeLocation");
    };

    return { clearWatch, stopWacthers, LocationStore };
  },
};
</script>
