<template lang="pug">
  .payment-history-details
    .payment-history-details__wrapper
      ui-debio-card(block centered-content)
        h2.payment-history-details__title {{ computeDetailsTitle }}
        .payment-history-details__content
          .payment-details__wrapper

            .payment-details__product
              ui-debio-avatar.product__image(src="https://picsum.photos/150" size="150" rounded)
              .product__details
                .product__name {{ $route.params.service_info.name }}
                .product__lab
                  .product__lab-name {{ $route.params.lab_info.name }}
                  ui-debio-rating.product__lab-rating(:rating="4" :total-reviews="800" size="15")
                .product__lab-address
                  span.address__title Address
                  p.address__text.mb-0 {{ $route.params.lab_info.address }}, {{ $route.params.lab_info.city }}

            .payment-details__status
              .payment-details__field
                .test__title Test status
                .test__status Rejected
              .payment-details__field
                .payment__title Payment Status
                .payment__status {{ $route.params.status }}
              .payment-details__field
                .speciment__title Specimen number
                .speciment__status {{ $route.params.dna_sample_tracking_id }}

            .payment-details__instruction(v-if="$route.params.status === 'Paid'")
              ui-debio-icon.payment-details__instruction-icon(:icon="alertIcon" size="15" color="#52C41B" stroke)
              p.payment-details__instruction-text.mb-0
                | Please proceed to send sample, see instruction
                | <router-link to="/" class="payment-details__instruction-link">here!</router-link> 

            .payment-details__service
              .service__table
                .service__field
                  .service__field-title Service Price
                  .service__field-colon :
                  .service__field-value
                    | {{ $route.params.service_info.prices_by_currency[0].total_price }}
                    | {{ $route.params.service_info.prices_by_currency[0].currency }}
                .service__field(v-if="$route.params.service_info.prices_by_currency[0].additional_prices.length")
                  .service__field-title Quality Control Price
                  .service__field-colon :
                  .service__field-value
                    | {{ $route.params.service_info.prices_by_currency[0].additional_prices[0].value }}
                    | {{ $route.params.service_info.prices_by_currency[0].currency }}
                .service__field(v-if="$route.params.status === 'Refunded'")
                  .service__field-title Refund amount
                  .service__refund -
                .service__field
                  .service__field-title.d-flex.align-center
                    | Reward
                    .reward(@mouseleave="handleShowPopup('leave')")
                      ui-debio-icon.reward__icon.ml-1(
                        :icon="alertIcon"
                        size="10"
                        color="#6F6F6F"
                        stroke
                        @mouseenter="handleShowPopup('enter')"
                      )
                      .reward__popup(v-if="rewardPopup")
                        .reward__popup-text you will get the reward after your test status is result ready and payment status fulfilled
                        Button.reward__popup-button.mt-6(color="secondary" height="30" width="100") Learn more
                  .service__field-colon :
                  .service__field-value - DBIO

</template>

<script>
import { alertIcon } from "@/common/icons"
import Button from "@/common/components/Button"

export default {
  name: "CustomerPaymentDetails",

  components: { Button },

  data: () => ({ alertIcon, rewardPopup: false }),

  computed: {
    computeDetailsTitle() {
      return this.$route.params?.status === "Paid"
        ? "Thank you for your order"
        : `${this.$route.params?.status} Order`
    }
  },

  beforeMount() {
    if (!Object.keys(this.$route.params).length) this.$router.push({ name: "customer-payment-history" })
  },

  methods: {
    handleShowPopup(val) {
      if (val === "enter") this.rewardPopup = true
      else this.rewardPopup = false
    }
  }
}
</script>

<style lang="sass" scoped>
  @import "@/common/styles/mixins.sass"

  .payment-history-details
    &__title
      margin-top: 35px

    &__content
      min-width: 575px
      max-width: 700px
      padding: 30px
      margin-top: 60px
      border: 1px solid #E9E9E9
      border-radius: 4px

    &::v-deep
      .ui-debio-card__body
        display: flex
        flex-direction: column
        align-items: center
        padding-bottom: 100px

  .payment-details
    &__product
      display: flex
      gap: 25px

    &__status
      display: flex
      gap: 50px
      margin: 40px 0 25px

    &__instruction
      background: #E7FFDC
      display: flex
      align-items: center
      justify-content: center
      margin-bottom: 25px
      gap: 15px
      padding: 12px
      color: #52C41B

    &__instruction-link
      color: #A568FF

  .product
    &__details
      width: calc(100% - 175px)
      display: flex
      flex-direction: column
      gap: 15px

    &__name
      @include h3

    &__lab
      width: 100%
      display: flex
      align-items: center
      justify-content: space-between
      gap: 80px
      
    &__lab-name
      @include body-text-2

  .address
    &__title
      @include body-text-medium-4

    &__text
      overflow: hidden
      display: -webkit-box
      -webkit-line-clamp: 2
      -webkit-box-orient: vertical
      color: #757274
      @include body-text-2

  .test__title,
  .payment__title,
  .speciment__title
    color: #595959
    @include body-text-4

  .test__status,
  .payment__status,
  .speciment__status
    @include body-text-medium-1

  .service
    &__table
      display: flex
      flex-direction: column

    &__field
      display: flex
      justify-content: space-between
      padding: 10px 40px
      background: #FCFCFC
      @include body-text-2

      &:nth-child(odd)
        background: #F6F7FB

    &__field-title
      flex: 1

    &__field-value
      flex: 1
      text-align: right
  .reward
    position: relative
    text-align: right

    &__popup
      width: 290px
      padding: 15px
      position: absolute
      font-size: 12px
      top: 23px
      left: -100px
      background: #FFFFFF
      border: 1px solid #E9E9E9

      &::before
        top: -22px
        content: ""
        display: block
        height: 20px
        left: 98px
        position: absolute
        border-color: transparent transparent #E9E9E9 transparent
        border-style: solid
        border-width: 11px

      &::after
        border-left: solid transparent 10px
        border-right: solid transparent 10px
        border-bottom: solid #FFFFFF 10px
        top: -10px
        content: " "
        height: 0
        left: 99px
        position: absolute
        width: 0

    &__popup-text
      text-align: left

    &__popup-button
      text-transform: capitalize
      font-size: 12px
</style>