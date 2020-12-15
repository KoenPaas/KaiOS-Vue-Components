<template>
    <div id="body">
        <div id="container">
            <label v-if="incorrect && !setup">Please try again. {{6 - count}} Tries left.</label>
            <label v-if="incorrect && setup">The pin code didn't match. Try again.</label>
            <label>Please enter your PIN</label>
            <div id="bullets">
                <div v-bind:class="{ bullet: true, fill: pin[count].length >= index }" v-for="index in 6" :key="index"></div>
            </div>
        </div>
        <input v-model="pin[count]" type="number" hidden>
    </div>
</template>

<script>
import socket from '@/js/socket.js'
import Vue from 'vue'

// account = account object, setup = bolean
export default {
  name: 'setup',
  props: ['account', 'setup'],
  data () {
    return {
      pin: ['', ''],
      count: 0,
      tries: 6,
      incorrect: false
    }
  },
  methods: {
    onKeyDown (event) {
      switch (event.key) {
        case 'Enter':
          break
        case 'SoftRight':
          Vue.set(this.pin, this.count, '')
          break
      }
      if (!isNaN(event.key)) {
        const push = this.pin[this.count] + event.key.toString()
        Vue.set(this.pin, this.count, push)
        if (this.pin[this.count].length === 6) this.next()
      }
    },
    next () {
      if (this.account !== undefined && this.setup) {
        if (this.count === 1) {
          if (this.pin[0] === this.pin[1]) {
            const address = this.account.account.address
            const secret = this.encrypt(this.account.account.familySeed, this.pin[0])
            socket.clear()
            localStorage.public = address
            localStorage.secret = secret
            this.$notify({
              group: 'foo',
              title: 'Created an Account!',
              type: 'success'
            })
            socket.init()
            setTimeout(() => this.$router.push('/'), 2000)
          } else {
            this.$notify({ group: 'foo', title: 'No match', type: 'error' })
            this.incorrect = true
            setTimeout(() => {
              this.count = 0
              Vue.set(this.pin, 0, '')
              Vue.set(this.pin, 1, '')
            }, 1500)
          }
          return null
        }
      } else {
        if (this.count === this.tries) this.deleteAccount()
      }
      setTimeout(() => this.count++, 500)
      // if setup
      // if (failed)
      // ifcount === 6 delete account else this.count ++
    },
    encrypt (input, secret) {
      // https://gist.github.com/WietseWind/fd8b0b9e888f58a747e18b31636ad423
      const salt = this.$CryptoJS.lib.WordArray.random(128 / 8)
      const iv = this.$CryptoJS.lib.WordArray.random(128 / 8)
      secret = this.$CryptoJS.enc.Utf8.parse(secret)
      var encrypted = this.$CryptoJS.AES.encrypt(input, this.$CryptoJS.PBKDF2(secret, salt, { keySize: 256 / 32, iterations: 100 }) /* key */, { iv: iv, padding: this.$CryptoJS.pad.Pkcs7, mode: this.$CryptoJS.mode.CBC })
      var output = salt.toString() + iv.toString() + encrypted.toString()
      return output
    },
    deleteAccount () {
      localStorage.removeItem('public')
      localStorage.removeItem('secret')
      socket.clear()
    }
  },
  mounted () {
    document.addEventListener('keydown', this.onKeyDown)
  },
  beforeDestroy () {
    document.removeEventListener('keydown', this.onKeyDown)
  }
}
</script>

<style scoped>
#body {
    display: flex;
    align-items: center;
}
#container {
    display: flex;
    flex-direction: column;
    align-items: center;
    margin: 0 auto;
}
#bullets {
    display: flex;
    flex-direction: row;
}

.bullet {
    border: 2px solid grey;
    border-radius: 20px;
    margin: 10px;
    height: 16px;
    width: 16px;
    display: flex;
    align-items: center;
}
.bullet.fill {
    background-color: grey;
}

.hole {
    border-radius: 20px;
    background-color: blue;
    width: 60%;
    height: 60%;
    margin: 0 auto;
}
</style>