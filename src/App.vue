<template>
  <div id="app">
    <h1>
      Algorand network status
      <a href="#" @click.prevent="fetchStatus"> <refresh-cw-icon size="0.5x"></refresh-cw-icon></a>
    </h1>
    <vue-json-pretty
        :data="status">
    </vue-json-pretty>

    <hr>

    <h1>
      Algorand suggested parameters
      <a href="#" @click.prevent="fetchParams"> <refresh-cw-icon size="0.5x"></refresh-cw-icon></a>
    </h1>
    <vue-json-pretty
        :data="params">
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
    },
    fetchParams() {
      this.client.getTransactionParams()
          .do()
          .then(data => this.params = data);
    }
  },
  created() {
    this.client = new window.algosdk.Algodv2(this.token, this.server, this.port);

    this.fetchStatus();

    this.fetchParams();
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
