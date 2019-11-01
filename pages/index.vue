<template>
  <section class="container">
    <div>
      <b-tabs content-class="mt-3">
        <b-tab title="QR Code Scan" active>
          <qrcode-stream @decode="onDecode" @init="onInit" />
          <span v-if="error" class="error">{{ error }}</span>
        </b-tab>
        <b-tab title="Phone/Email">
          <input id="input" v-model="phoneNumber" type="text" placeholder="Insert registered phone number">
          <button id="button" @click="verifyPhoneNumber">SCAN</button>
        </b-tab>
      </b-tabs>
    </div>
  </section>
</template>

<script>
import { QrcodeStream } from 'vue-qrcode-reader'
const reUrl = /(?:(?:https?|ftp):\/\/|\b(?:[a-z\d]+\.))(?:(?:[^\s()<>]+|\((?:[^\s()<>]+|(?:\([^\s()<>]+\)))?\))+(?:\((?:[^\s()<>]+|(?:\(?:[^\s()<>]+\)))?\)|[^\s`!()\[\]{};:'".,<>?«»“”‘’]))?/
const rePhoneNumber = /^01[3-9]\d{8}$/

function isUrl (data) {
  console.log(reUrl.test(data))
  console.log(rePhoneNumber.test(data))
  return reUrl.test(data) || rePhoneNumber.test(data)
}

export default {
  components: {
    QrcodeStream
  },
  data () {
    return {
      error: '',
      phoneNumber: null
    }
  },
  methods: {
    verifyPhoneNumber () {
      if (rePhoneNumber.test(this.phoneNumber)) {
        const url = `https://ng-bd.com/attendee/search?q=${this.phoneNumber}`
        this.process(url)
      } else {
        alert('Please input a correct registered phone number')
      }
    },
    onDecode (result) {
      if (reUrl.test(result)) {
        this.process(result)
      } else {
        alert('Please scan a correct qr code')
      }
    },
    process (result) {      
      this.$axios.$get(result)
        .then(result => {
          if (!!result.data.is_paid) {
            if (!!result.data.attend_at) {
              alert("You have allready confirmed your attendance")
            } else {
              return this.$axios.$put(result.approve_url)
                .then(() => {
                  alert('Your attendance is confirmed!')
                })
                .catch(() => {
                  alert('something went wrong, please go for manual process! :(')
                })
                .finally(() => {
                  this.phoneNumber = null
                })
            }
          } else {
            alert("You have not paid for the event yet")
          }
        })
        .catch(error => {
          if (error.message.includes('404')) alert('Sorry, you were not found registered!')
          if (error.message.includes('401')) alert('You have allready confirmed your attendance!')
        })
        .finally(() => {
          this.phoneNumber = null
        })
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

#input {
  border: 1px solid black;
}

#button {
  border: 1px solid;
  float: right;
}
</style>
