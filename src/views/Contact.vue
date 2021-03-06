<template>
    <div v-if="contactobj" class="container">
        <fieldset>
            <label>Name:</label>
            <input v-model="contactobj.name" placeholder="Contact Name" ref="input0">
        </fieldset>
        <hr>
        <fieldset>
            <label>Address:</label>
            <input v-model="contactobj.public" class="prevent" placeholder="R Address" ref="input1">
            <input v-model="contactobj.tag" class="prevent" type="number" placeholder="Destination Tag" ref="input2">
        </fieldset>
        <hr>
        <div id="buttons">
            <input type="button" class="prevent" value="Delete" ref="input3">
            <input type="button" value="Save" ref="input4">
            <input type="button" value="Send" ref="input5">
        </div>
    </div>
</template>

<script>
import store from '@/js/store'
import Vue from 'vue'
import { Encode } from 'xrpl-tagged-address-codec'
import dataStore from '@/js/dataStore.worker.js'

export default {
  props: ['contact', 'add'],
  data () {
    return {
      focusIndex: 0,
      contactobj: {
        name: null,
        public: null,
        tag: null
      }
    }
  },
  methods: {
    async deleteContact () {
      try {
        await dataStore.deleteContact(this.contactobj.public)
        this.$router.push('/Contacts')
      } catch (e) {
        this.$notify({ group: 'foo', title: 'Could not delete contact', type: 'error' })
        console.error(e)
      }
    },
    async setContact () {
      try {
        if (this.contact) {
          await dataStore.setContact(this.contactobj.public, this.contactobj.name, this.contactobj.tag, true)
          Vue.notify({ group: 'foo', type: 'success', title: 'Updated a Contact' })
        } else {
          await dataStore.setContact(this.contactobj.public, this.contactobj.name, this.contactobj.tag, false)
          Vue.notify({ group: 'foo', type: 'success', title: 'Added a Contact' })
        }
        this.$router.push('/Contacts')
      } catch (e) {
        this.$notify({ group: 'foo', title: e, type: 'error' })
      }
    },
    onKeyDown (event) {
      switch (event.key) {
        case 'ArrowDown':
          this.focusIndex++
          break
        case 'ArrowUp':
          this.focusIndex--
          break
      }
      this.focusInput(Object.keys(this.$refs).length)
    },
    focusInput (length) {
      if (this.focusIndex >= length) {
        this.focusIndex = length - 1
      } else if (this.focusIndex < 0) {
        this.focusIndex = 0
      }
      const input = 'input' + this.focusIndex
      this.$refs[input].focus()
    }
  },
  created () {
    store.keys.left = {
      string: 'Back',
      fn: () => this.$router.go(-1)
    }
    store.keys.center.fn = () => {
      if (this.focusIndex >= 3) {
        try {
          Encode({
            account: this.contactobj.public,
            tag: this.contactobj.tag,
            test: false
          })
        } catch (e) {
          return this.$notify({ group: 'foo', title: 'This is not an XRP Account', type: 'error' })
        }
      }

      switch (this.focusIndex) {
        case 3:
          this.deleteContact()
          break
        case 4:
          this.setContact()
          break
        case 5:
          this.$router.push({ name: 'Send', params: { account: this.contactobj.public, tag: this.contactobj.tag } })
      }
    }
    store.keys.right = {
      string: 'del',
      fn: () => {
        if (this.focusIndex >= 3) return null
        var newValue
        if (this.focusIndex === 0) {
          newValue = this.contactobj.name.slice(0, -1)
          return Vue.set(this.contactobj, 'name', newValue)
        }
        if (this.focusIndex === 1) {
          newValue = this.contactobj.public.slice(0, -1)
          return Vue.set(this.contactobj, 'account', newValue)
        }
        if (this.focusIndex === 2) {
          newValue = this.contactobj.tag.toString().slice(0, -1)
          Vue.set(this.contactobj, 'tag', parseInt(newValue))
        }
      }
    }
    if (!this.contact && !this.add) this.$router.push({ name: 'Contacts' })
    else if (this.contact) this.contactobj = this.contact
  },
  mounted () {
    document.addEventListener('keydown', this.onKeyDown)
    document.getElementsByClassName('prevent').forEach(element => {
      element.addEventListener('keydown', e => {
        if (e.which === 38 || e.which === 40) {
          e.preventDefault()
        }
      })
    })
    if (this.$refs.input0) this.$refs.input0.focus()
  },
  beforeDestroy () {
    document.removeEventListener('keydown', this.onKeyDown)
  }
}
</script>

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
    width: 100%;
    overflow-y: hidden;
    overflow-x: hidden;
  }
fieldset {
    display: flex;
    flex-direction: column;
    border: none;
    padding: 0;
}
fieldset label {
    margin: 6px;
}
fieldset input {
    margin: 6px;
    color: blue;
}
hr {
  color: rgba(77, 79, 102);
  width: 95%;
}
#buttons {
    display: flex;
    flex-direction: row;
    align-self: center;
}
#buttons input {
    margin: 10px;
}
#buttons input:focus {
    color: white;
    background-color: blue;
}
</style>
