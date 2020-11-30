<template>
  <div id="app">
    <h1>
      Algorand network status
      <a href="#" @click.prevent="fetchStatus"> <refresh-cw-icon size="0.5x"></refresh-cw-icon></a>
    </h1>
    <vue-json-pretty
        :data="status">
    </vue-json-pretty>
  </div>
</template>

<script>
import VueJsonPretty from 'vue-json-pretty'
import { RefreshCwIcon } from 'vue-feather-icons'
import 'vue-json-pretty/lib/styles.css'
export default {
  name: 'App',
  data: () => {
    return {
      client: null,
      token: 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa',
      server: 'http://localhost',
      port: 4001,
      status: {}
    };
  },
  components: {
    VueJsonPretty,
    RefreshCwIcon
  },
  methods: {
    fetchStatus() {
      this.client.status()
          .do()
          .then(data => this.status = data);
    }
  },
  created() {
    this.client = new window.algosdk.Algodv2(this.token, this.server, this.port);

    this.fetchStatus();
  }
}
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  background-color: #2c3e50;
  color: azure;
  margin-top: 5px;
}
</style>
