<template>
  <div class="login">
    <b-modal id="loginModal" title="Login" hide-footer centered>
      <template #default>
        <div class="pt-md-3 pb-md-3 pr-md-5 pl-md-5">
          <div class="form-group">
            <b-form-input v-model.trim="username" placeholder="Hive username" :state="$v.username.$dirty ? !$v.username.$error : null" @keyup.enter="logMeIn" />
          </div>

          <div class="text-center">
            <b-button variant="primary" block @click="logMeIn">
              Login with Keychain
            </b-button>

            <template v-if="!isKeychain">
              <p class="small mt-3 mb-0">
                Download Hive Keychain for
              </p>

              <ul class="list-inline">
                <li class="list-inline-item">
                  <a href="https://chrome.google.com/webstore/detail/hive-keychain/jcacnejopjdphbnjgfaaobbfafkihpep" target="_blank">Chrome/Opera/Brave</a>
                </li>
                <li class="list-inline-item">
                  <a href="https://addons.mozilla.org/en-GB/firefox/addon/hive-keychain/" target="_blank">Firefox</a>
                </li>
                <li class="list-inline-item">
                  <a href="https://play.google.com/store/apps/details?id=com.mobilekeychain" target="_blank">Android</a>
                </li>
                <li class="list-inline-item">
                  <a href="https://apps.apple.com/us/app/hive-keychain/id1552190010" target="_blank">iOS</a>
                </li>
              </ul>
            </template>
            <hr>

            <b-button variant="outline-secondary" block @click.prevent="$bvModal.show('smartLock')">
              SmartLock
            </b-button>
          </div>
        </div>
      </template>
    </b-modal>

    <smart-lock :callback="smartLockLogin" :key-types="['posting','active']" />
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import { required, minLength, maxLength } from 'vuelidate/lib/validators'
import SmartLock from '@/components/modals/SmartLock.vue'

export default {
  name: 'LoginModal',

  components: {
    SmartLock
  },

  data () {
    return {
      username: ''
    }
  },

  computed: {
    isKeychain () {
      return !!window.hive_keychain
    }
  },

  async beforeMount () {
    if (!this.$auth.loggedIn) {
      const self = this
      this.username = localStorage.getItem('username')

      if (this.username) {
        const wif = localStorage.getItem(`smartlock-${this.username}`)

        if (wif) {
          await this.smartLockLogin(this.username, wif)
        } else if (!window.hive_keychain) {
          new Promise(resolve => setTimeout(resolve, 500)).then(() => {
            if (!window.hive_keychain) {
              new Promise(resolve => setTimeout(resolve, 1000)).then(() => {
                if (!window.hive_keychain) {
                  return
                }

                self.logMeIn()
              })
            } else {
              self.logMeIn()
            }
          })
        } else {
          self.logMeIn()
        }
      }
    }
  },

  mounted () {
    this.$root.$on('smartlock-loggedin', () => {
      this.$root.$bvModal.hide('loginModal')
    })
  },

  methods: {
    ...mapActions('user', ['login', 'loginWithKey']),

    async logMeIn () {
      this.$v.$touch()

      if (window.hive_keychain && !this.$v.$invalid) {
        await this.login({ username: this.username })

        this.$bvModal.hide('loginModal')
      }
    },

    async smartLockLogin (username, wif) {
      const data = {
        username,
        wif
      }

      await this.loginWithKey(data)
    }
  },

  validations: {
    username: {
      required,
      minLength: minLength(3),
      maxLength: maxLength(16)
    }
  }
}
</script>

<style>
</style>
