<script setup>
import { ref } from "vue";
import { pausableWatch, useBluetooth } from "@vueuse/core";

const { isSupported, isConnected, device, requestDevice, server } =
  useBluetooth({
    acceptAllDevices: true,
    optionalServices: ["battery_service"],
  });

const batteryPercent = ref();

const isGettingBatteryLevels = ref(false);

const getBatteryLevels = async () => {
  isGettingBatteryLevels.value = true;

  // Get the battery service:
  const batteryService = await server.getPrimaryService("battery_service");

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

  // Convert received buffer to number:
  const batteryLevel = await batteryLevelCharacteristic.readValue();

  batteryPercent.value = await batteryLevel.getUint8(0);
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
</template>




<style scoped>
.read-the-docs {
  color: #888;
}
</style>
