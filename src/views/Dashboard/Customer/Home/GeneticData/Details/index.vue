<template lang="pug">
  .ga-order-detail
    .ga-order-detail__title Test Summary

    .ga-order-detail__cards
      ServiceCard
      StepperStatusCard(
        @cancel="showingDialog"
        @reject="showReasonOfRejection"
      )

    ConfirmationDialog(
      :show="showCancelDialog"
      :loading="isLoading"
      :txWeight="Number(txWeight).toFixed(4)"
      title="Cancel"
      btnMessage="Cancel Order"
      message="By cancel this service, your file might not be able to be analyzed, and your balance will be refunded"
      @close="showCancelDialog = false"
      @click="cancelOrder"
    )

    ReasonOfRejection(
      :show="reasonOfRejection"
      @close="reasonOfRejection = false"
    )
  
</template>


<script>
import { mapState } from "vuex"
import ServiceCard from "./ServiceCard"
import StepperStatusCard from "./StepperStatusCard"
import ReasonOfRejection from "./ReasonOfRejection"
import ConfirmationDialog from "@/views/Dashboard/Customer/Home/MyTest/ConfirmationDialog"
import { cancelGeneticAnalysisOrder } from "@debionetwork/polkadot-provider"
import { getCancelGeneticAnalysisOrderFee } from "@/common/lib/polkadot-provider/command/genetic-analysis-orders"

export default {
  name: "GAOrderDetail",

  components: {
    ServiceCard,
    StepperStatusCard,
    ConfirmationDialog,
    ReasonOfRejection
  },

  computed: {
    ...mapState({
      api: (state) => state.substrate.api,
      lastEventData: (state) => state.substrate.lastEventData,
      wallet: (state) => state.substrate.wallet,
      web3: (state) => state.metamask.web3
    })
  },

  watch: {
    lastEventData(e) {
      if (e !== null) {
        const dataEvent = JSON.parse(e.data.toString())
        if (e.method === "GeneticAnalysisOrderCancelled") {
          if (dataEvent[0].customerId === this.wallet.address) {
            this.isLoading = false
            this.showCancelDialog = false
          }
        }
      }
       
    }
  },

  data: () => ({
    showCancelDialog: false,
    orderId: null,
    isLoading: false,
    txWeight: "Calculating...",
    reasonOfRejection: false
  }),

  async mounted() {
    this.orderId = null

    if(this.$route.params.id) {
      this.orderId = this.$route.params.id
    }

    await this.getCancelFee()

  },

  methods: {
    showingDialog() {
      this.showCancelDialog = true
    },

    async getCancelFee() {
      const txWeight = await getCancelGeneticAnalysisOrderFee(this.api, this.wallet, this.orderId)
      this.txWeight = this.web3.utils.fromWei(String(txWeight.partialFee), "ether")
    },

    async cancelOrder() {
      this.isLoading = true
      await cancelGeneticAnalysisOrder(this.api, this.wallet, this.orderId)
    },

    showReasonOfRejection() {
      this.reasonOfRejection = true
    }
  }

}
</script>

<style lang="sass" scoped>
  @import "@/common/styles/mixins.sass"
  @import "@/common/styles/functions.sass"


  .ga-order-detail
    &__title
      display: flex
      justify-content: center
      align-items: center
      letter-spacing: 0.0044em
      margin-top: toRem(52px)
      @include h6-opensans

    &__cards
      display: flex
      padding: toRem(52px)
      gap: toRem(14px)

      

</style>