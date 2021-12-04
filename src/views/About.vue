<template>
  <div class="about">
    <h1><a href="https://docs.ethers.io/">ethers.js</a> Test</h1>
    <div v-if="connected_wallet">
      <h2>Network</h2>
      <dl>
        <dt>chainId</dt>
        <dd>{{ network.chainId }}</dd>
        <dt>name</dt>
        <dd>{{ network.name }}</dd>
      </dl>
      <dl>
        <dt>Block Number</dt>
        <dd>{{ blockNumber }}</dd>
      </dl>
      <input
        v-model="addressForGetBalance"
        placeholder="Address For Get Balance"
      />
      <button @click="getBalance">balance</button>
      <dl>
        <dt>Balance</dt>
        <dd>{{ balance }}</dd>
      </dl>
      <dl>
        <dt>My Balance</dt>
        <dd>{{ myBalance }}</dd>
      </dl>
      <div style="padding: 10px; background: #e2e2e2">
        <input
          v-model="addressForSend"
          placeHolder="Address For Send ETH"
          style="margin-bottom: 10px"
        /><br />
        <input
          v-model="sendAmount"
          placeHolder="amount (ETH)"
          style="margin-bottom: 10px"
        /><br />
        <button @click="sendEth" v-if="!isSending">SEND!</button>
        <p v-else>SENDING...</p>
        <p v-if="sendError" style="color: red">{{ sendError }}</p>
        <p v-if="txResponseHash" style="color: blue">
          {{ txResponseHash }}
        </p>
      </div>
    </div>
    <p v-else>please connect wallet</p>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import { ethers } from "ethers";
import { markRaw } from "vue";

declare global {
  interface Window {
    ethereum: any;
  }
}

type Network = {
  chainId: number | null;
  name: string | null;
  ensAddress: string | null;
};
const network = {
  chainId: null,
  name: null,
  ensAddress: null,
};

export default defineComponent({
  data: () => ({
    provider: null as any,
    blockNumber: null as number | null,
    addressForGetBalance: null as string | null,
    addressForSend: null as string | null,
    balance: 0.0 as number,
    myAccountAddress: null as string | null,
    myBalance: 0.0 as number,
    signer: null as any,
    network: network as Network,
    sendAmount: null as number | null,
    isSending: false as boolean,
    sendError: null as null | string,
    txResponseHash: null as null | string,
  }),
  async created() {
    await window.ethereum.enable();
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    this.provider = markRaw(provider);
    this.getBlockNumber();
    this.getNetwork();
    this.getSigner();
  },
  computed: {
    connected_wallet: function () {
      return window.ethereum.enable();
    },
  },
  methods: {
    async getBlockNumber() {
      this.blockNumber = await this.provider.getBlockNumber();
    },
    async getNetwork() {
      this.network = await this.provider.getNetwork();
    },
    async getSigner() {
      this.signer = this.provider.getSigner();
      this.myAccountAddress = await this.signer.getAddress();
      const balance = await this.signer.getBalance();
      this.myBalance = parseFloat(ethers.utils.formatEther(balance));
    },
    async getBalance() {
      this.balance = 0.0;
      const balance = await this.provider.getBalance(this.addressForGetBalance);
      this.balance = parseFloat(ethers.utils.formatEther(balance));
    },
    async sendEth() {
      this.isSending = true;
      this.sendError = null;
      this.txResponseHash = null;
      if (this.sendAmount === null || this.sendAmount <= 0) {
        alert("please input AMOUNT");
        this.isSending = false;
        return;
      }
      const eth = this.sendAmount.toString();
      const wei = ethers.utils.parseEther(eth);
      console.log("wei", wei);
      const tx = await this.signer
        .sendTransaction({
          to: this.addressForSend,
          from: this.myAccountAddress,
          value: wei,
        })
        .catch((err: any) => (this.sendError = err.toString()));
      console.log("transactionResponse", tx);
      this.txResponseHash = tx.hash;
      this.isSending = false;
    },
  },
});
</script>
