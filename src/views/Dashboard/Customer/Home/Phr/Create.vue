<template lang="pug">
  .customer-create-phr
    ui-debio-error-dialog(
      :show="!!error"
      :title="error ? error.title : ''"
      :message="error ? error.message : ''"
      @close="error = null"
    )

    ui-debio-modal(
      :show="showModal"
      :title="isEdit ? 'Edit Health Record File' : 'Add Health Record File'"
      cta-title="Submit"
      :cta-action="handleNewFile"
      :cta-outlined="false"
      @onClose="onCloseModalDocument"
    )
      ui-debio-input(
        :error="isDirty.document && isDirty.document.title"
        :rules="$options.rules.document.title"
        variant="small"
        label="Document Title"
        placeholder="Medical Receipt"
        v-model="document.title"
        outlined
        block
        validate-on-blur
        @isError="handleError"
      )
      ui-debio-textarea(
        :error="isDirty.document && isDirty.document.description"
        :rules="$options.rules.document.description"
        variant="small"
        label="Description"
        placeholder="This is my first file in PDF Format. Lorem ipsum dolor sit amet,"
        v-model="document.description"
        validate-on-blur
        outlined
        block
        @isError="handleError"
      )
      ui-debio-file(
        v-model="document.file"
        :error="isDirty.document && isDirty.document.file"
        :rules="$options.rules.document.file"
        variant="small"
        accept=".pdf"
        label-rules="(.pdf - Maximum file size is 2 MB)"
        label="File input"
        :clearFile="!isEdit && clearFile"
        @isError="handleError"
        validate-on-blur
      )

    .customer-create-phr__wrapper
      .customer-create-phr__title.mb-13 Upload Health Records
      .customer-create-phr__forms
        ui-debio-input(
          :rules="$options.rules.phr.title"
          variant="small"
          label="Health Record Title"
          :error="isDirty.phr && isDirty.phr.title"
          placeholder="Add Title"
          v-model="phr.title"
          outlined
          block
          validate-on-blur
          @isError="handleError"
        )

        ui-debio-dropdown(
          :items="categories"
          :error="isDirty.phr && isDirty.phr.category"
          :rules="$options.rules.phr.category"
          variant="small"
          label="Health Record Category"
          placeholder="Select Category"
          v-model="phr.category"
          item-text="category"
          item-value="category"
          outlined
          close-on-select
          validate-on-blur
          block
          @isError="handleError"
        )

        ui-debio-button.secondary--text(
          color="secondary"
          height="2.5rem"
          block
          outlined
          @click="handleAddFile"
        ) Add file

        .customer-create-phr__files
          .customer-create-phr__files-title
            p {{ computeFiles.length ? "Uploaded Files" : "File Information" }}
            p.mb-0(v-if="!computeFiles.length") Before uploading the document, please ensure that all personal data is removed or redacted.
          .customer-create-phr__files-items
            .customer-create-phr__file-item.customer-create-phr__file-item--no-file.d-flex.align-center(
              :class="{ 'customer-create-phr__file-item--error': fileEmpty }"
              v-if="!computeFiles.length"
              @click="showModal = true"
            )
              .customer-create-phr__file-details.mt-0
                .customer-create-phr__file-details--left
                  ui-debio-icon.customer-create-phr__file-icon(
                    :icon="fileTextIcon"
                    size="28"
                    color="#D3C9D1"
                    fill
                  )
                  .customer-create-phr__file-name No File uploaded, Please add file to upload

            template(v-else)
              .customer-create-phr__file-item(v-for="(item, idx) in computeFiles" :key="item.createdAt")
                ui-debio-modal.modal-confirm(
                  :show="showModalConfirm === item.createdAt"
                  title="Do you want to delete ?"
                  @onClose="showModalConfirm = null"
                )
                  span.modal-confirm__title By deleting this file, your file will not be uploaded
                  .modal-confirm__cta.d-flex.justify-space-between(slot="cta")
                    ui-debio-button(
                      outlined
                      width="100"
                      color="secondary"
                      @click="showModalConfirm = null"
                    ) No
                    ui-debio-button(
                      width="100"
                      color="secondary"
                      @click="onDelete(item.createdAt)"
                    ) Yes

                .customer-create-phr__file-title(:title="`Title: ${item.title}`") {{ item.title }}
                .customer-create-phr__file-description(:title="`Description: ${item.description}`") {{ item.description }}
                .customer-create-phr__file-details
                  .customer-create-phr__file-details--left
                    ui-debio-icon.customer-create-phr__file-icon(
                      :icon="fileTextIcon"
                      size="28"
                      color="#D3C9D1"
                      fill
                    )
                    .customer-create-phr__file-name(v-if="item.file" :title="`File: ${item.file.name}`") {{ item.file.name }}

                  .customer-create-phr__file-details--right
                    ui-debio-icon.customer-create-phr__file-edit(
                      :icon="pencilIcon"
                      size="15"
                      color="#989898"
                      stroke
                      @click="onEdit(item)"
                    )
                    ui-debio-icon.customer-create-phr__file-delete(
                      :icon="trashIcon"
                      size="15"
                      color="#989898"
                      fill
                      @click="showModalConfirm = item.createdAt"
                    )

        .d-flex.flex-column
          p.transaction-weight__info.d-flex.justify-space-between(
            @mouseleave="handleShowTooltip"
          )
            span.transaction-weight__text.mr-6.d-flex.align-center
              | Estimated transaction weight
              ui-debio-icon.ml-1(
                :icon="alertIcon"
                size="14"
                stroke
                @mouseenter="handleShowTooltip"
              )
              span.transaction-weight__tooltip(
                :class="{ 'transaction-weight__tooltip--show': showTooltip }"
              ) Total fee paid in DBIO to execute this transaction.
            span {{ txWeight }}
          ui-debio-button.white--text(
            color="secondary"
            :loading="isLoading"
            :disabled="disabledButton"
            height="2.5rem"
            @click="handleSubmit"
            block
          ) Submit
</template>

<script>
import { mapState } from "vuex"

import Kilt from "@kiltprotocol/sdk-js"
import CryptoJS from "crypto-js"
import cryptWorker from "@/common/lib/ipfs/crypt-worker"
import { getEMRCategories } from "@/common/lib/api"
import {
  registerElectronicMedicalRecord,
  registerElectronicMedicalRecordFee
} from "@debionetwork/polkadot-provider"
import { u8aToHex } from "@polkadot/util"
import { validateForms } from "@/common/lib/validate"
import { errorHandler } from "@/common/lib/error-handler"
import { generalDebounce } from "@/common/lib/utils"
import errorMessage from "@/common/constants/error-messages"
import { uploadFile, getFileUrl } from "@/common/lib/pinata-proxy"
import { fileTextIcon, alertIcon, pencilIcon, trashIcon, eyeOffIcon, eyeIcon } from "@debionetwork/ui-icons"
import Web3 from "web3"

const englishAlphabet = val => (val && /^[A-Za-z0-9!@#$%^&*\\(\\)\-_=+:;"',.\\/? ]+$/.test(val)) || errorMessage.INPUT_CHARACTER("English alphabet")

export default {
  name: "CustomerPHRCreate",

  mixins: [validateForms],

  data: () => ({
    errorMessage,
    fileTextIcon,
    pencilIcon,
    trashIcon,
    eyeOffIcon,
    alertIcon,
    eyeIcon,

    isEdit: false,
    showModal: false,
    showPassword: false,
    showModalConfirm: null,
    error: null,
    isLoading: false,
    disabledButton: false,
    fileEmpty: false,
    clearFile: false,
    showTooltip: false,
    password: "",
    publicKey: null,
    secretKey: null,
    txWeight: null,
    phr: {
      title: "",
      category: "",
      files: []
    },
    document: {
      title: "",
      description: "",
      file: null
    },
    categories: []
  }),

  computed: {
    ...mapState({
      api: (state) => state.substrate.api,
      wallet: (state) => state.substrate.wallet,
      lastEventData: (state) => state.substrate.lastEventData,
      mnemonicData: (state) => state.substrate.mnemonicData,
      walletBalance: (state) => state.substrate.walletBalance
    }),

    computeFiles() {
      return this.phr.files.map(file => ({ ...file, percent: 0 })).reverse()
    },

    disabledDocumentForm() {
      return this.document.title === "" || this.document.description === "" || this.document.file === null
    },

    disabledFinish() {
      return this.computeFiles?.some(file => file.percent !== 100)
    }
  },

  watch: {
    lastEventData(event) {
      if (event !== null) {
        const dataEvent = JSON.parse(event.data.toString())
        if (event.method === "ElectronicMedicalRecordAdded") {
          if (dataEvent[1] === this.wallet.address) {
            this.resetState()
            this.isLoading = false
            this.$router.push({ name: "customer-phr" })
          }
        }
      }
    },

    mnemonicData(val) {
      if (val) this.initialDataKey()
    },

    phr: {
      deep: true,
      immediate: true,
      handler: generalDebounce(async function (val) {
        this.txWeight = "Calculating..."

        const txWeight = await registerElectronicMedicalRecordFee(this.api, this.wallet, val)
        this.txWeight = `${Number(Web3.utils.fromWei(String(txWeight.partialFee), "ether")).toFixed(4)} DBIO`
      }, 500)
    }
  },

  rules: {
    password: [val => !!val || errorMessage.PASSWORD(8)],
    phr: {
      title: [
        val => !!val || errorMessage.REQUIRED,
        val => val && val.length < 50 || errorMessage.MAX_CHARACTER(50),
        englishAlphabet
      ],
      category: [val => !!val || errorMessage.REQUIRED]
    },
    document: {
      title: [
        val => !!val || errorMessage.REQUIRED,
        val => val && val.length < 50 || errorMessage.MAX_CHARACTER(50),
        englishAlphabet
      ],
      description: [
        val => !!val || errorMessage.REQUIRED,
        val => val && val.length < 255 || errorMessage.MAX_CHARACTER(255),
        englishAlphabet
      ],
      file: [
        val => !!val || errorMessage.REQUIRED,
        val => (val && val.size < 2000000) || errorMessage.FILE_SIZE(2),
        val => (val && val.type === "application/pdf") || errorMessage.FILE_FORMAT("PDF")
      ]
    }
  },

  async created() {
    this.fetchCategories()
    if (this.mnemonicData) this.initialDataKey()
  },

  methods: {
    initialDataKey() {
      const cred = Kilt.Identity.buildFromMnemonic(this.mnemonicData.toString(CryptoJS.enc.Utf8))

      this.publicKey = u8aToHex(cred.boxKeyPair.publicKey)
      this.secretKey = u8aToHex(cred.boxKeyPair.secretKey)
    },

    async fetchCategories() {
      this.categories = await getEMRCategories()
    },

    resetState() {
      Object.assign(this.phr, { title: "", category: "", files: [] })
      Object.assign(this.document, { title: "", description: "", file: null })

      this.password = ""
    },

    handleNewFile() {
      if (Number(this.walletBalance) < Number(this.txWeight.split(" ")[0])) {
        this.error = {
          title: "Insufficient Balance",
          message: "Your transaction cannot go through because your account balance is too low or doesn't meet the minimum deposit needed. Please check your balance."
        }
        return
      }

      this._touchForms("document")
      const { title: docTitle, description: docDescription, file: docFile } = this.isDirty?.document
      if (docTitle || docDescription || docFile) return

      const context = this
      const fr = new FileReader()
      const { createdAt, title, description, file } = this.document

      fr.onload = async function () {
        try {
          const encrypted = await context.encrypt({
            text: fr.result,
            fileType: file.type,
            fileSize: file.size,
            fileName: file.name
          })

          const { chunks, fileName, fileType, fileSize } = encrypted

          const dataFile = {
            title,
            description,
            file,
            chunks,
            fileName,
            fileSize,
            fileType,
            createdAt: new Date().getTime()
          }

          if (context.isEdit) {
            const index = context.phr.files.findIndex(phrFile => phrFile.createdAt === createdAt)

            context.phr.files[index] = dataFile

            context.phr.files = context.phr.files.map(phrFile => phrFile)
            context.isEdit = false
          } else {
            context.phr.files.push(dataFile)
          }

        } catch (e) {
          this.error = e.message
        }
      }

      fr.readAsArrayBuffer(file)

      this.onCloseModalDocument()
    },

    onEdit(item) {
      Object.assign(this.document, { ...item })
      this.showModal = true
      this.isEdit = true
      this.clearFile = false
    },

    handleCancelUpload() {
      this.showLoadingFiles = false
      this.resetState()
    },

    onCloseModalDocument() {
      setTimeout(() => {
        this.isEdit = false
      }, 350)
      this.showModal = false
      Object.assign(this.document, { title: "", description: "", file: null })
      this.clearFile = true
      this._resetForms("document")
    },

    onDelete(id) {
      this.showModalConfirm = null
      this.phr.files = this.phr.files.filter(file => file.createdAt !== id)
    },

    handleAddFile() {
      this.showModal = true
      this.isEdit = false
      this.clearFile = false
    },

    async handleSubmit() {
      if (Number(this.walletBalance) < Number(this.txWeight.split(" ")[0])) {
        this.error = {
          title: "Insufficient Balance",
          message: "Your transaction cannot go through because your account balance is too low or doesn't meet the minimum deposit needed. Please check your balance."
        }
        return
      }

      this._touchForms("phr")
      const isPHRValid = Object.values(this.isDirty?.phr).every(v => v !== null && v === false)
      const isDocumentValid = Object.values(this.isDirty?.document).every(v => v !== null && v === false)

      if (!isPHRValid || (!isDocumentValid && this.phr.files.length < 1)) {
        if (!this.phr.files.length) this.fileEmpty = true

        return
      }

      this.fileEmpty = false
      this.clearFile = true

      await this.finalSubmit()
    },

    handleShowPassword() {
      this.showPassword = !this.showPassword
    },

    async finalSubmit() {
      this.isLoading = true
      this.disabledButton = true

      try {
        if (this.phr.files.length === 0) return

        for await (let [index, value] of this.phr.files.entries()) {
          const dataFile = await this.setupFileReader({ value })
          await this.upload({
            encryptedFileChunks: dataFile.chunks,
            fileName: dataFile.fileName,
            fileSize: dataFile.fileSize,
            index: index,
            fileType: dataFile.fileType
          })
        }

        await registerElectronicMedicalRecord(this.api, this.wallet, this.phr)
      } catch (e) {
        const error = await errorHandler(e.message)
        this.error = error
        this.isLoading = false
        this.disabledButton = false
      }
    },

    setupFileReader({ value }) {
      return new Promise((resolve, reject) => {
        const file = value.file
        const fr = new FileReader()

        fr.onload = async function () {
          resolve(value)
        }

        fr.onerror = reject

        fr.readAsArrayBuffer(file)
      })
    },

    async encrypt({ text, fileType, fileName, fileSize }) {
      const context = this
      const arrChunks = []
      let chunksAmount
      const pair = {
        secretKey: this.secretKey,
        publicKey: this.publicKey
      }

      const data = await new Promise((resolve, reject) => {
        try {
          cryptWorker.workerEncryptFile.postMessage({ pair, text, fileType }) // Access this object in e.data in worker
          cryptWorker.workerEncryptFile.onmessage = async (event) => {
            if (event.data.chunksAmount) {
              chunksAmount = event.data.chunksAmount
              return
            }

            arrChunks.push(event.data)
            context.encryptProgress = (arrChunks.length / chunksAmount) * 100

            if (arrChunks.length === chunksAmount) {
              resolve({
                fileName,
                fileSize,
                fileType,
                chunks: arrChunks
              })
            }
          }

          this.fileEmpty = false
        } catch (err) {
          reject(new Error(err.message))
        }
      })

      return data
    },

    async upload({ encryptedFileChunks, index, fileType, fileName, fileSize }) {
      const data = JSON.stringify(encryptedFileChunks)
      const blob = new Blob([data], { type: fileType })

      const result = await uploadFile({
        title: fileName,
        type: fileType,
        size: fileSize,
        file: blob
      })

      const link = getFileUrl(result.IpfsHash)

      this.phr.files[index].recordLink = link
    },

    setPercent(file) {
      const selected = this.computeFiles.find(v => v.file === file)

      setInterval(() => {
        if (selected && selected.percent < 100) selected.percent += 1
      }, 2000)

      return selected?.percent
    },

    handleShowTooltip(e) {
      if (e.type === "mouseenter") {
        this.showTooltip = true
      } else {
        this.showTooltip = false
      }
    },

    onRetry() {
      // TODO: Add script later
    },

    onCancel() {
      // TODO: Add script later
    }
  }
}
</script>

<style lang="sass" scoped>
  @import "@/common/styles/mixins.sass"

  .customer-create-phr
    width: 100%
    height: 100%

    &__wrapper
      background: #ffffff
      padding: 50px 0

    &__title
      @include h6
      text-align: center

    &__forms
      display: flex
      flex-direction: column
      gap: 20px
      margin: 0 auto
      max-width: 450px

    &__files
      margin: 10px 0 20px

    &__files-title
      margin-bottom: 24px

      p:first-of-type
        @include button-2

      p
        @include body-text-3-opensans

    &__files-items
      display: flex
      flex-direction: column
      gap: 20px
      padding-right: .35rem
      max-height: calc(116px * 3)
      overflow-y: auto

      &::-webkit-scrollbar-track
        background-color: #f2f2ff

      &::-webkit-scrollbar
        width: 0.25rem

      &::-webkit-scrollbar-thumb
        border-radius: 0.625rem
        background: #a1a1ff

    &__files-loading
      &::v-deep
        .ui-debio-modal__card
          max-width: 600px

    &__file-item
      padding: 12px 20px
      border-radius: 4px
      border-style: dashed 
      border-color: #8AC1FF
      background: #F9F9FF
      transition: all cubic-bezier(.7, -0.04, .61, 1.14) .3s

      &:hover
        background: #f2f2ff
        border-color: #a1a1ff

      &--no-file
        height: 64px

      &--error
        border-color: #c400a5

        &::v-deep
          .customer-create-phr__file-icon > svg
            color: #c400a5 !important

        .customer-create-phr__file-name
          color: #c400a5

    &__file-title
      font-weight: 600
      overflow: hidden
      white-space: nowrap
      text-overflow: ellipsis
      @include body-text-1

    &__file-name
      max-width: 320px
      white-space: nowrap
      overflow: hidden
      text-overflow: ellipsis
      @include body-text-5

    &__file-description
      margin-top: 10px
      display: -webkit-box
      -webkit-line-clamp: 2
      -webkit-box-orient: vertical
      overflow: hidden
      color: #595959
      @include body-text-4

    &__file-details
      margin-top: 12px
      display: flex
      justify-content: space-between

      &--left,
      &--right
        display: flex
        align-items: center
        gap: 10px

    &__file-edit,
    &__file-delete
      cursor: pointer

    &::v-deep

      .ui-debio-modal__card-title
        @include h2
        font-weight: 700

  .modal-confirm
    &__title
      width: 212px
      text-align: center

    &__cta
      width: 250px
      margin: 0 auto

  .modal-password
    &__cta
      gap: 20px

  .transaction-weight
    &__text
      position: relative

    &__tooltip
      max-width: 143px
      padding: 10px
      position: absolute
      top: 35px
      z-index: 10
      background: #fff
      border: 1px solid #D3C9D1
      right: -120px
      transition: all .3s ease-in-out
      visibility: hidden
      opacity: 0
      @include tiny-reg

      &::after
        position: absolute
        content: ""
        display: block
        top: -20px
        height: 20px
        border-color: transparent transparent #FFF transparent
        border-style: solid
        border-width: 8px

      &::before
        position: absolute
        content: ""
        display: block
        top: -21px
        height: 20px
        border-color: transparent transparent #D3C9D1 transparent
        border-style: solid
        border-width: 8px

      &--show
        opacity: 1
        visibility: visible

  .modal-files-upload
    &__files
      display: flex
      flex-direction: column
      gap: 35px

    &__file-details
      display: flex
      align-items: center
      gap: 115px
      justify-content: space-between

    &__file-name
      max-width: 360px
      overflow: hidden
      text-overflow: ellipsis
      white-space: nowrap
      @include body-text-medium-2

    &__progress
      display: flex
      align-items: center
      justify-content: space-between
      gap: 15px
      margin-top: 8px

    &__status
      display: flex
      align-items: center
      gap: 8px

    &__file-percent,
    &__error-message
      @include body-text-5

    &__error-message
      color: #F5222D

    &__file-percent
      color: #8C8C8C

    &__progress-check
      width: 12px
      height: 12px
      border-radius: 50%

      &--success
        background: linear-gradient(225deg, #76DA92 0%, #61FFF6 100%)

      &--cancel
        background: #757274

      &--failed
        background: #FF525B

    &__progressbar
      width: 100%
      height: 6px
      background: #E7E7E7
      border-radius: 3px

      &--thumb
        width: 0%
        height: 100%
        background: #6FE4AF
        border-radius: inherit
</style>
