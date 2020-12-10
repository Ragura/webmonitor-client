<template>
  <img alt="Vue logo" src="./assets/ps5.jpeg" />
  <h1>Playstation 5 webmonitor</h1>
  <h2 v-if="connected">Connected to server</h2>
  <h2 v-else>Not connected to server</h2>
  <ul v-show="connected">
    <li v-for="site of sites" :key="site.url">
      <a :href="site.url">{{ site.title }}</a> :
      {{ site.changed ? "CHANGED!!!" : "UNCHANGED" }}
    </li>
  </ul>
</template>

<script>
import io from "socket.io-client";
import { ref } from "vue";

export default {
  name: "App",
  components: {},
  setup() {
    const socket = io("http://localhost:3000");
    const connected = ref(false);
    const sites = ref([]);
    const alertsound = new Audio("/alert.mp3");
    // const alarmsound = new Audio("/alarm.mp3");

    socket.on("connect", () => {
      connected.value = socket.connected;
    });

    socket.on("disconnect", () => {
      connected.value = false;
      // alarmsound.play();
    });

    socket.on("list", data => {
      sites.value = data;
      console.log("Initial list:", data);
    });

    socket.on("update", data => {
      if (data.changed) {
        const index = sites.value.findIndex(site => site.url === data.url);
        if (index >= 0) {
          sites.value[index].changed = data.changed;
        }
      }
      alertsound.play();
    });

    return {
      connected,
      sites,
      alertsound
    };
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
