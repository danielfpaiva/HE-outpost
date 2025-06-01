<template>
  <div class="app-container">
    <div v-if="$config.SIDECHAIN_ID !== 'ssc-mainnet-hive'" class="bg-info">
      <div class="container-fluid text-center py-2">
        <fa-icon icon="exclamation-circle" /> The website is currently running on Hive-Engine Testnet!
      </div>
    </div>

    <Header />

    <main>
      <Nuxt />
    </main>

    <Login />
    <SignUp v-if="$config.OUTPOST_ONBOARD" />

    <notifications :duration="15000" position="top right" />

    <client-only>
      <back-to-top bottom="50px" right="50px">
        <b-button variant="primary" class="btn-to-top">
          <fa-icon icon="chevron-up" />
        </b-button>
      </back-to-top>
    </client-only>
  </div>
</template>

<script>
import { mapActions } from 'vuex'
import BackToTop from 'vue-backtotop'
import Header from '@/components/Header.vue'
import Login from '@/components/modals/Login.vue'

export default {
  name: 'MainLayout',

  components: {
    BackToTop,
    Header,
    Login,
    SignUp: () => import(/* webpackChunkName: "SignUpModal" */ '@/components/modals/SignUp.vue')
  },

  async fetch () {
    await this.fetchTrendingTags()
  },

  async mounted () {
    if (this.$auth.loggedIn) {
      await Promise.all([this.fetchFollowers(), this.fetchFollowing()])
    }

    this.$auth.$storage.watchState('loggedIn', (loggedIn) => {
      if (!loggedIn) {
        this.$cookies.remove('nftm_access_token')
        this.$cookies.remove('nftm_refresh_token')
      }
    })
    
    // Add Google Fonts
    const link = document.createElement('link')
    link.rel = 'stylesheet'
    link.href = 'https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap'
    document.head.appendChild(link)
  },

  methods: {
    ...mapActions('user', ['fetchAccountScotData', 'fetchFollowers', 'fetchFollowing']),
    ...mapActions('scot', ['fetchTrendingTags'])
  },

  timers: {
    fetchAccountScotData: { time: 3 * 60 * 1000, autostart: true, immediate: true, repeat: true }
  }
}
</script>

<style>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

main {
  flex: 1;
}

.btn-to-top {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  font-size: 25px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  transition: all 0.2s ease;
}

.btn-to-top:hover {
  transform: translateY(-3px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.15);
}
</style>