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

    <h1 class="mb-0">
      Create an Account
      <button @click="createNewAccount(true)">Create Now</button>
    </h1>
    <a href="https://bank.testnet.algorand.network/" target="_blank">
      <external-link-icon size="1.0x"></external-link-icon>
      Fund the account for testnet
    </a>
    <a href="#" @click.prevent="copyAddress">
      <copy-icon size="1.0x"></copy-icon> {{ copyAddressText }}
    </a>
    <br>
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
import { RefreshCwIcon, CopyIcon, ExternalLinkIcon } from 'vue-feather-icons'
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
      copyAddressText: 'Copy address'
    };
  },
  components: {
    VueJsonPretty,
    RefreshCwIcon,
    CopyIcon,
    ExternalLinkIcon
  },
  computed: {
    accountAddress() {
      return 'address' in this.account ? this.account.address : '';
    }
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
      };

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
    },
    async copyAddress() {
      if (!navigator.clipboard) {
        alert('Copy to clipboard not supported!')
      }

      await navigator.clipboard.writeText(this.accountAddress);

      this.copyAddressText = 'Copied!';

      setTimeout(() => this.copyAddressText = 'Copy Address', 1000);
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

  .mb-0 {
    margin-bottom: 0;
  }

  a {
    color: wheat;
    margin-bottom: 1rem;
    float: right;
    font-size: small;

    svg { margin-left: 2rem; }
  }

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
