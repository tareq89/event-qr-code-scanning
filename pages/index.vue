<template>
  <section class="container">
    <div>
      <qrcode-stream @decode="onDecode" @init="onInit" />
      <span v-if="error" class="error">{{ error }}</span>
    </div>
  </section>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'

function isUrl (data) {
  return true
}

function callMatchUrl (url) {
  switch (url) {
    case 1:
      return Promise.resolve({
        isPaid: 1,
        isAttended: 0,
        approve_url: 'https://laravel.com.bd'
      })
    case 1:
      return Promise.resolve({
        isPaid: 1,
        isAttended: 1
      })
    default:
      return Promise.resolve({
        isPaid: 0,
        isAttended: 0
      })
  }
}

function callApproveUrl (url) {
  return Promise.resolve({})
}

export default {
  components: {
    QrcodeStream
  },
  data () {
    return {
      result: '',
      error: '',
      info: '',
      showConfirmAttendance: false
    }
  },
  methods: {
    onDecode (result) {
      this.result = result
      if (isUrl(result)) {
        callMatchUrl(result)
          .then(result => {
            if (result.isPaid) {
              if (result.isAttended) {
                this.info = "You have allready confirmed your attendance"
              } else {
                this.showConfirmAttendance = true
              }
            } else {
              this.info = "You have not paid for the event yet"
            }
          })
      } 
    },
    confirmAttendance () {
      callApproveUrl(this.result)
        .then(() => {
          alert('Your attendance has been confirmed')
        })
        .catch(() => {
          alert('CONFIRMATION FAILED!')
        })
        .finally(() => {
          this.resetState()
        })
    },
    resetState () {
      this.info = null
      this.showConfirmAttendance = false
      this.result = null
    },
    async onInit (promise) {
      try {
        await promise
      } catch (error) {
        if (error.name === 'NotAllowedError') {
          this.error = "ERROR: you need to grant camera access permisson"
        } else if (error.name === 'NotFoundError') {
          this.error = "ERROR: no camera on this device"
        } else if (error.name === 'NotSupportedError') {
          this.error = "ERROR: secure context required (HTTPS, localhost)"
        } else if (error.name === 'NotReadableError') {
          this.error = "ERROR: is the camera already in use?"
        } else if (error.name === 'OverconstrainedError') {
          this.error = "ERROR: installed cameras are not suitable"
        } else if (error.name === 'StreamApiNotSupportedError') {
          this.error = "ERROR: Stream API is not supported in this browser"
        }
      }
    }
  }
}
</script>

<style>
.container
{
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.title
{
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif; /* 1 */
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}
.subtitle
{
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}
.links
{
  padding-top: 15px;
}
.error {
  background-color: red;
  color: white;
  font-size: 1em;
}
</style>
