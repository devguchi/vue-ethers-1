<template>
  bytes32 <input v-model="b32" size="80" /><br />
  string <input v-model="str" size="80" style="margin-top: 10px" /><br />
  <button @click="toString" style="margin-top: 10px">toString</button>&nbsp;
  <button @click="toBytes32">toBytes32</button>
</template>

<script lang="ts">
import { BytesLike, ethers } from "ethers";
import { defineComponent } from "vue";

export default defineComponent({
  data: () => ({
    b32: null as BytesLike | null,
    str: null as string | null,
  }),
  methods: {
    toString() {
      if (this.b32 === null) return;
      this.str = ethers.utils.parseBytes32String(this.b32);
    },
    toBytes32() {
      if (this.str === null) return;
      this.b32 = ethers.utils.formatBytes32String(this.str);
    },
  },
});
</script>
