<template>
  <main>
    <div class="hero">
      <h1 v-if="pending">...</h1>
      <h1 v-else>{{ toFixed(gwei, 4) }} gwei</h1>
      <div v-if="error">{{ error }}</div>
      <p>updated @ {{ updatedAt ? formatISO(updatedAt) : "" }}</p>
      <p>update in {{ countdown }}s</p>
    </div>
    <h2>Ethereum Gas Tracker</h2>
    <p>
      Unlike
      <a
        href="https://etherscan.io/gastracker"
        target="_blank"
        rel="noopener noreferrer"
        >Etherscan</a
      >, this Ethereum Gas Tracker directly retrieves gas prices through
      <a
        herf="https://ethereum.github.io/execution-apis/api-documentation/"
        target="_blank"
        rel="noopener noreferrer"
        >Ethereum's JSON-RPC interface</a
      >, provided by
      <a
        href="https://developers.cloudflare.com/web3/ethereum-gateway/"
        target="_blank"
        rel="noopener noreferrer"
        >Cloudflare</a
      >. It doesn't require an access token and is a static website that can be
      deployed anywhere. The gas price will be updated in the webpage's title
      bar every {{ interval }} seconds, allowing users to keep it open in the
      background as an indicator.
    </p>
  </main>
</template>

<script setup lang="ts">
import { useInterval, useTitle } from "@vueuse/core";
import { formatISO, secondsToMilliseconds } from "date-fns";
import { Decimal } from "decimal.js";
import { computed, onMounted, ref, watch } from "vue";

const pending = ref(false);
const error = ref<null | unknown>(null);
const wei = ref<null | Decimal>(null);
const gwei = computed(() => (wei.value ? wei.value.div(10 ** 9) : null));

const updatedAt = ref<null | Date>();
const title = computed(
  () => `${toFixed(gwei.value, 2)} | Ethereum gas tracker`,
);
useTitle(title);

function toFixed(n: null | Decimal, m: number) {
  return n ? n.toFixed(m) : null;
}

const endpoint = "https://cloudflare-eth.com";
async function refreshGasPrice() {
  try {
    error.value = null;
    pending.value = true;
    const body = {
      jsonrpc: "2.0",
      method: "eth_gasPrice",
      params: [],
      id: 0,
    };
    const json = await fetch(endpoint, {
      method: "POST",
      body: JSON.stringify(body),
    }).then((r) => r.json());
    wei.value = new Decimal(json.result);
    updatedAt.value = new Date();
  } catch (err) {
    console.error(err);
    error.value = err;
  } finally {
    pending.value = false;
  }
}

const interval = 15;
const counter = useInterval(secondsToMilliseconds(1));
watch(
  () => counter.value,
  () => {
    if (counter.value % interval !== 0) return;
    refreshGasPrice();
  },
);
const countdown = computed(() => interval - (counter.value % interval));

onMounted(() => refreshGasPrice());
</script>

<style scoped>
.hero {
  text-align: center;
}
</style>
