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
      <input v-model="address" placehodler="Ethereum Address" />
      <button @click="getBalance">balance</button>
      <dl>
        <dt>Balance</dt>
        <dd>{{ balance }}</dd>
      </dl>
      <dl>
        <dt>My Account</dt>
        <dd>{{ myAccountAddress }}</dd>
      </dl>
      <dl>
        <dt>My Balance</dt>
        <dd>{{ myBalance }}</dd>
      </dl>
    </div>
    <p v-else>connect wallet</p>
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
    address: null as string | null,
    balance: 0.0 as number,
    myAccountAddress: null as string | null,
    myBalance: 0.0 as number,
    signer: null as any,
    network: network as Network,
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
      const balance = await this.provider.getBalance(this.address);
      this.balance = parseFloat(ethers.utils.formatEther(balance));
    },
  },
});
</script>
