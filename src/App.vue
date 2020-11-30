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

    <hr>

    <h1>
      Create an Account
      <button @click="createNewAccount(true)">Create Now</button>
    </h1>
    <vue-json-pretty
        :data="account">
    </vue-json-pretty>

    <hr>

    <h1>
      Wallet Info
      <a href="#" @click.prevent="fetchWalletInfo"> <refresh-cw-icon size="0.5x"></refresh-cw-icon></a>
    </h1>
    <vue-json-pretty
        :data="wallet">
    </vue-json-pretty>

  </div>
</template>

<script>
import localforage from 'localforage'
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
      status: {},
      params: {},
      account: {},
      wallet: {},
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
    },
    createNewAccount(recreate = false) {
      let account = window.algosdk.generateAccount();
      this.account = {
        address: account.addr,
        passphrase: window.algosdk.secretKeyToMnemonic(account.sk)
      }

      if (recreate) {
        localforage.setItem('algo-account', this.account);

        this.fetchWalletInfo();
      }
    },
    fetchAccount() {
      localforage.getItem('algo-account').then(value => {
        if (value) {
          this.account = value;

          this.fetchWalletInfo();

          return;
        }

        this.createNewAccount();

        localforage.setItem('algo-account', this.account);

        this.fetchWalletInfo();
      });
    },
    fetchWalletInfo() {
      this.client.accountInformation(this.account.address)
          .do()
          .then(info => this.wallet = info);
    }
  },
  created() {
    this.client = new window.algosdk.Algodv2(this.token, this.server, this.port);

    this.fetchStatus();

    this.fetchParams();

    this.fetchAccount();
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

  h1 {
    position: relative;

    button {
      position: absolute;
      top: .75rem;
      margin-left: 1rem;
      margin-bottom: 10px;
    }
  }
}
</style>
