<script setup>
import { ref, watch } from "vue";
import { pausableWatch, useBluetooth } from "@vueuse/core";

const { isSupported, isConnected, device, requestDevice } = useBluetooth({
  acceptAllDevices: true,
  optionalServices: ["battery_service"],
});

// Get the battery service:
const batteryService = ref();
const batteryLevelCharacteristic = ref();
const batteryLevel = ref();
const batteryPercent = ref();
const deviceName = ref();

let server = async function () {
  console.log("setting up server");

  let server = await device.value.gatt.connect();
  console.log(server);
  deviceName.value = server.device.gatt.device.name;
  batteryService.value = await server.getPrimaryService("battery_service");
  batteryLevelCharacteristic.value =
    await batteryService.value.getCharacteristic("battery_level");
  batteryLevel.value = await batteryLevelCharacteristic.value.readValue();
  // Convert received buffer to number:
  batteryPercent.value = await batteryLevel.value.getUint8(0);
  return server;
};

watch(device, server);

const isGettingBatteryLevels = ref(false);

const getBatteryLevels = async () => {
  isGettingBatteryLevels.value = true;

  // Get the current battery level
  const batteryLevelCharacteristic = await batteryService.getCharacteristic(
    "battery_level"
  );

  // Listen to when characteristic value changes on `characteristicvaluechanged` event:
  batteryLevelCharacteristic.addEventListener(
    "characteristicvaluechanged",
    (event) => {
      batteryPercent.value = event.target.value.getUint8(0);
    }
  );
};

const { stop } = pausableWatch(isConnected, (newIsConnected) => {
  if (!newIsConnected || !server.value || isGettingBatteryLevels.value) return;
  // Attempt to get the battery levels of the device:
  getBatteryLevels();
  // We only want to run this on the initial connection, as we will use a event listener to handle updates:
  stop();
});
</script>


  <template>
  <button @click="requestDevice()">Request Bluetooth Device</button>
  <h1>{{ deviceName }}</h1>
  {{ batteryPercent }}%
  <progress id="file" :value="batteryPercent" max="100"> {{ batteryPercent }}% </progress>
</template>




<style scoped>
.read-the-docs {
  color: #888;
}
</style>
