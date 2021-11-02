<template lang="pug">
  v-dialog(:value="show" width="480" persistent )
    v-card
      div.pa-5(class="d-flex justify-end") 
        v-btn.fixed-button(icon @click="closeDialog")
          v-icon mdi-close

      div.pa-5(class="d-flex justify-center")
        v-icon(color="#BA8DBB" :size="125") {{ selectedService.serviceImage }}
        
      div(class="d-flex justify-center pb-5 pt-1")
        .dialog-service__title
          b {{ selectedService.serviceName }}

      div(class="pa-5")
        div(class="pa-2")
          .dialog-service__sub-title
            b Description
          .dialog-service__description
            div  {{ selectedService.serviceDescription }}

        div(class="pa-2")
          .dialog-service__sub-title
            b Expected Duration
          .dialog-service__description          
            div {{ selectedService.duration }} {{ selectedService.durationType}}
      
      v-row(class="pa-5")
        v-col(cols="3")
          v-icon.ml-2(v-if="!avatar" color="#BA8DBB" size="90") mdi-hospital
          v-img.ml-2(v-else :src="avatar" max-width="90" max-height="90")
        
        v-col(cols="6 mt-3")
          .dialog-service__sub-title
            b {{ selectedService.labName }}

          .dialog-service__description          
            span {{ selectedService.labAddress }}


        v-col(cols="3")
          v-row(class="ml-1 mt-1 mb-1")
            div(
              v-for="i in selectedService.labRate"
              :key="i")
              v-icon(color="primary" size="10") mdi-star

            div(
              v-for="i in ( 5 - selectedService.labRate )"
              :key="i")
              v-icon(color="primary" size="10" ) mdi-star-outline 
            span.ml-2( style="font-size: 10px;") ({{ selectedService.countRateLab }})

      .dialog-service__button
        Button(
          color="secondary" 
          width="48%"
          height="38" 
          style="font-size: 11px"
          outlined 
          @click="downloadFile"
        ) Download Sample Report

        Button(
          color="secondary" 
          width="48%"
          height="38" 
          style="font-size: 11px"
          @click="onSelect"
        ) Select This Service

</template>


<script>

import { mapState } from "vuex"
import Button from "@/common/components/Button"
import { downloadDecryptedFromIPFS } from "@/common/lib/ipfs"
import { hexToU8a } from "@polkadot/util"


export default {
  name: "ServiceDetailDialog",
  data: () => ({
    formatedDurationType: "",
    avatar: ""
  }),

  components: {
    Button
  },

  props: {
    show: Boolean
  },

  computed: {
    ...mapState({
      mnemonicData: (state) => state.substrate.mnemonicData,
      selectedService: (state) => state.testRequest.products
    })  
  },

  methods: {
    closeDialog() {
      this.$emit("close")
    },

    onSelect () {
      this.$router.push({ name: "customer-checkout"})
    },

    async downloadFile () {
      const publicKey = hexToU8a(this.mnemonicData.publicKey)
      const privateKey = hexToU8a(this.mnemonicData.privateKey)
      const baseUrl = "https://ipfs.io/ipfs/"
      const path = this.downloadPath.replace(baseUrl, "")
      await downloadDecryptedFromIPFS(
        path,
        privateKey,
        publicKey,
        this.serviceId + ".pdf",
        "application/pdf"
      )
    }
  }
}
</script>

<style lang="sass" scoped>
  @import "@/common/styles/mixins.sass"

  .dialog-service
    &__title
      display: flex
      align-items: center
      text-align: center
      letter-spacing: 0.0044rem
      @include h6

    &__sub-title
      @include body-text-2-opensans

    &__description
      @include body-text-3-opensans

    &__button
      margin-top: 15px
      display: flex
      align-items: center
      justify-content: space-between
      padding: 0 35px 40px 35px
      gap: 10px
    
    
  .fixed-button
    position: fixed
    width: 50px

      

</style>

