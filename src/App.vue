<template>
  <div id="app">
    <h1>
      Algorand network status
      <a href="#" @click.prevent="fetchStatus">
        <refresh-cw-icon size="0.5x"></refresh-cw-icon>
      </a>
      <a href="https://developer.algorand.org/docs/build-apps/connect/#check-node-status"
         target="_blank" class="link-info">
        <info-icon size="1.0x"></info-icon>
      </a>
    </h1>
    <vue-json-pretty
        :data="status">
    </vue-json-pretty>

    <hr>

    <h1>
      Algorand suggested parameters
      <a href="#" @click.prevent="fetchParams">
        <refresh-cw-icon size="0.5x"></refresh-cw-icon>
      </a>
      <a href="https://developer.algorand.org/docs/build-apps/connect/#check-suggested-transaction-parameters"
         target="_blank" class="link-info">
        <info-icon size="1.0x"></info-icon>
      </a>
    </h1>
    <vue-json-pretty
        :data="params">
    </vue-json-pretty>

    <hr>

    <h1 class="mb-0">
      Create an Account
      <button @click="createNewAccount(true)">Create Now</button>
      <a href="https://developer.algorand.org/docs/build-apps/hello_world/#create-an-account"
         target="_blank" class="link-info">
        <info-icon size="1.0x"></info-icon>
      </a>
    </h1>
    <a href="https://bank.testnet.algorand.network/" target="_blank" class="link">
      <external-link-icon class="link-icon" size="1.0x"></external-link-icon>
      Fund the account for testnet
    </a>
    <a href="#" @click.prevent="copyAddress" class="link">
      <copy-icon class="link-icon" size="1.0x"></copy-icon>
      {{ copyAddressText }}
    </a>
    <br>
    <h2>Address: <span>{{ account.address }}</span></h2>

    <hr>

    <h1>
      Account Info
      <a href="#" @click.prevent="fetchAccountInfo">
        <refresh-cw-icon size="0.5x"></refresh-cw-icon>
      </a>
      <a href="https://developer.algorand.org/docs/build-apps/hello_world/#check-your-balance"
         target="_blank" class="link-info">
        <info-icon size="1.0x"></info-icon>
      </a>
    </h1>
    <vue-json-pretty
        :data="accountInfo">
    </vue-json-pretty>

    <hr>

    <h1>
      Send a Pay Transaction To TestNet Faucet Address
      <a href="https://developer.algorand.org/docs/build-apps/hello_world/#construct-the-transaction"
         target="_blank" class="link-info">
        <info-icon size="1.0x"></info-icon>
      </a>
    </h1>
    <input type="number" min="0" v-model="amount" :disabled="sending">
    <button :disabled="!validAmount" @click="submitTxt">
      Send {{ this.amount > 0 ? this.amount : '' }} Algos
    </button>
    <br>
    <ol>
      <li v-for="(step, index) in txSteps" :key="index">{{ step }}</li>
    </ol>
    <a v-if="txId" :href="explorerLink" target="_blank" class="link link-left">
      <external-link-icon class="link-icon" size="1.0x"></external-link-icon>
      Show on Algorand BC Explorer for TestNet
    </a>

    <br>

    <hr>

    <h1>
      Read the last transaction from the blockchain
      <button @click="fetchTxnInfo" :disabled="!txId.length">Get Info</button>
      <a href="https://developer.algorand.org/docs/build-apps/hello_world/#read-the-transaction-from-the-blockchain"
         target="_blank" class="link-info">
        <info-icon size="1.0x"></info-icon>
      </a>
    </h1>
    <div v-if="txId">
      <vue-json-pretty
          :data="txnInfo">
      </vue-json-pretty>
    </div>
    <p v-else>
      Please generate a transaction before.
    </p>

    <br>

  </div>
</template>

<script>
import localforage from 'localforage'
import VueJsonPretty from 'vue-json-pretty'
import {RefreshCwIcon, CopyIcon, ExternalLinkIcon, InfoIcon} from 'vue-feather-icons'
import 'vue-json-pretty/lib/styles.css'

export default {
  name: 'App',
  data: () => {
    return {
      TESTNET_FAUCET_ADDRESS: 'GD64YIY3TWGDMCNPP553DZPPR6LDUSFQOIJVFDPPXWEG3FVOJCCDBBHU5A',
      TXN_FEE: 1000,
      TXN_NOTE: 'Generated from git@github.com:crx4/algorand-sandbox-examples.git',
      amount: '0',
      txId: '',
      txSteps: [],
      client: null,
      token: 'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa',
      server: 'http://localhost',
      port: 4001,
      status: {},
      params: {},
      account: {},
      accountInfo: {},
      txnInfo: {},
      copyAddressText: 'Copy address',
      sending: false
    };
  },
  components: {
    VueJsonPretty,
    RefreshCwIcon,
    CopyIcon,
    ExternalLinkIcon,
    InfoIcon
  },
  computed: {
    accountAddress() {
      return 'address' in this.account ? this.account.address : '';
    },
    validAmount() {
      return parseInt(this.amount) > 0 &&
          parseInt(this.amount + '000000') <= (parseInt(this.accountInfo.amount) - this.TXN_FEE) &&
          !this.sending;
    },
    explorerLink() {
      return 'https://testnet.algoexplorer.io/tx/' + this.txId;
    }
  },
  methods: {
    fetchStatus() {
      this.client.status()
          .do()
          .then(data => this.status = data);
    },
    fetchParams() {
      return this.client.getTransactionParams()
          .do()
          .then(data => this.params = data);
    },
    createNewAccount(recreate = false) {
      let account = window.algosdk.generateAccount();

      this.account = {
        address: account.addr,
        detail: {
          sk: account.sk,
          passphrase: window.algosdk.secretKeyToMnemonic(account.sk)
        }
      };

      if (recreate) {
        localforage.setItem('algo-account', this.account);

        this.fetchAccountInfo();
      }
    },
    fetchAccount() {
      localforage.getItem('algo-account').then(value => {
        if (value) {
          this.account = value;

          this.fetchAccountInfo();

          return;
        }

        this.createNewAccount();

        localforage.setItem('algo-account', this.account);

        this.fetchAccountInfo();
      });
    },
    fetchAccountInfo() {
      this.client.accountInformation(this.account.address)
          .do()
          .then(info => this.accountInfo = info);
    },
    async copyAddress() {
      if (!navigator.clipboard) {
        alert('Copy to clipboard not supported!')
      }

      await navigator.clipboard.writeText(this.accountAddress);

      this.copyAddressText = 'Copied!';

      setTimeout(() => this.copyAddressText = 'Copy Address', 1000);
    },
    async submitTxt() {
      this.txSteps = [];
      this.sending = true;

      this.txSteps.push('Fetching actual network parameters.');
      await this.fetchParams();

      this.params.fee = this.TXN_FEE;
      this.params.flatFee = true;

      let txn = window.algosdk.makePaymentTxnWithSuggestedParams(
          this.account.address,
          this.TESTNET_FAUCET_ADDRESS,
          parseInt(this.amount + '000000'),
          undefined,
          window.algosdk.encodeObj(this.TXN_NOTE),
          this.params
      );

      this.txSteps.push('Signing transaction.');
      let signedTxn = txn.signTxn(this.account.detail.sk);
      this.txId = txn.txID().toString();

      this.txSteps.push('Submitting transaction to the TestNet. TxID: ' + this.txId);
      await this.client.sendRawTransaction(signedTxn).do();
      this.txSteps.push('Transaction submitted!');

      await this.waitForConfirmation();

      this.fetchAccountInfo();

      this.sending = false;
    },
    async waitForConfirmation() {
      let status = (await this.client.status().do());
      let lastRound = status["last-round"];
      while (this.sending) {
        const pendingInfo = await this.client.pendingTransactionInformation(this.txId).do();
        if (pendingInfo["confirmed-round"] !== null && pendingInfo["confirmed-round"] > 0) {
          this.txSteps.push("Transaction " + this.txId + " confirmed in round " + pendingInfo["confirmed-round"]);
          break;
        }
        this.txSteps.push('Waiting for confirmation.');
        lastRound++;
        await this.client.statusAfterBlock(lastRound).do();
      }
    },
    async fetchTxnInfo() {
      this.txnInfo = await this.client.pendingTransactionInformation(this.txId).do();
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
  background-color: #2c3e50;
  color: azure;
  padding: 0 1rem;
  margin-top: 5px;

  .mb-0 {
    margin-bottom: 0;
  }

  a {
    color: wheat;
  }

  ol {
    color: #13ce66;
  }

  h2 span {
    color: #13ce66;
  }

  .link {
    margin-bottom: 1rem;
    float: right;
    font-size: small;

    .link-icon {
      margin-left: 2rem;
    }
  }

  .link-left {
    float: left !important;
  }

  .link-info {
    float: right !important;
    margin-top: 1rem;
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
