<template>
  <div class="relative mx-auto lg:w-1/2 border-2 border-red-900 bg-gray-50 shadow-xl rounded-xl p-8">
    <p class="text-3xl font-bold">Your Staking Account is: {{ parseFarmerState(farmerAcc) }}</p>
    <div class="mb-2 w300:block hidden">Your identity: {{ farmerAcc?.identity.toBase58() }}</div>
    <div class="mb-2">Associated vault: {{ farmerAcc?.vault.toBase58() }}</div>
    <div class="mb-2">Items staked: {{ farmerAcc?.gemsStaked }}</div>

    <div class="flex mb-5">
      <div class="flex-1 mr-5">
      <FarmerRewardDisplay
        :key="farmerAcc?.rewardA"
        :farmReward="farmAcc?.rewardA"
        :reward="farmerAcc?.rewardA"
	      :allGems="farmAcc?.gemsStaked"
        :yourGems="farmerAcc?.gemsStaked"
          title="$DUST Rewards"
        />
      </div>
    </div>
    <button class="bg-rb-mickeyred text-white rounded-lg py-2 px-3" @click="refreshFarmer">
      Refresh rewards
    </button>
    <div class="absolute w-32 -bottom-6 right-4">
      <img src="/converse.png" alt="shoey" />
    </div>
    <div class="absolute w-32 -bottom-6 right-32">
      <img src="/air-shoey.png" alt="shoey" />
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, watch } from 'vue';
import FarmerRewardDisplay from '@/components/gem-farm/FarmerRewardDisplay.vue';
import useWallet from '@/composables/wallet';
import useCluster from '@/composables/cluster';
import { initGemFarm } from '@/common/gem-farm';
import { PublicKey } from '@solana/web3.js';
import { parseDate } from '@/common/util';

export default defineComponent({
  components: { FarmerRewardDisplay },
  props: {
    farm: String,
    farmAcc: Object,
    farmer: String,
    farmerAcc: Object,
  },
  emits: ['refresh-farmer'],
  setup(props, ctx) {
    const { wallet, getWallet } = useWallet();
    const { cluster, getConnection } = useCluster();

    let gf: any;
    watch([wallet, cluster], async () => {
      gf = await initGemFarm(getConnection(), getWallet()!);
    });

    //need an onmounted hook because this component isn't yet mounted when wallet/cluster are set
    onMounted(async () => {
      if (getWallet() && getConnection()) {
        gf = await initGemFarm(getConnection(), getWallet()!);
      }
    });

    // --------------------------------------- display farmer
    // todo ideally should be using one from client, but n/a during render
    const parseFarmerState = (farmer: any): string => {
      return Object.keys(farmer.state)[0];
    };

    const refreshFarmer = async () => {
      await gf.refreshFarmerWallet(
        new PublicKey(props.farm!),
        new PublicKey(props.farmer!)
      );
      ctx.emit('refresh-farmer');
    };

    return {
      refreshFarmer,
      parseFarmerState,
      parseDate,
    };
  },
});
</script>

<style scoped></style>
