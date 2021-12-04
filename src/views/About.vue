<template>
  <div class="about">
    <h1><a href="https://docs.ethers.io/">ethers.js</a> Test</h1>
    <input v-model="address" placehodler="Ethereum Address" />
    <button @click="getBalance">balance</button>
    <dl>
      <dt>Block Number</dt>
      <dd>{{blockNumber}}</dd>
    </dl>
    <dl>
      <dt>Balance</dt>
      <dd>{{balance}}</dd>
    </dl>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import {ethers} from "ethers";
import {markRaw} from "vue";

declare global {
  interface Window {
    ethereum: any;
  }
}

export default defineComponent({
  data: () => ({
    provider: null as any,
    blockNumber: null as number | null,
    address: null as string | null,
    balance: 0.0 as number
  }),
  async created() {
    await window.ethereum.enable();
    const provider = new ethers.providers.Web3Provider(window.ethereum);
    this.provider = markRaw(provider);
    this.getBlockNumber();
  },
  methods: {
    async getBlockNumber() {
      this.blockNumber = await this.provider.getBlockNumber();
    },
    async getBalance() {
      this.balance = 0.0
      const balance = await this.provider.getBalance(this.address);
      this.balance = parseFloat(ethers.utils.formatEther(balance));
    }
  }
})
</script>
