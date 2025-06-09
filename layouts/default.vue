<template>
  <div class="app-container">
    <div v-if="$config.SIDECHAIN_ID !== 'ssc-mainnet-hive'" class="bg-info">
      <div class="container-fluid text-center py-2">
        <fa-icon icon="exclamation-circle" /> The website is currently running on Hive-Engine Testnet!
      </div>
    </div>

    <div class="main-layout-wrapper">
      <div class="sidebar-container">
        <div class="sidebar-logo">
          <nuxt-link to="/">
            <img src="/logo.png" alt="Logo">
          </nuxt-link>
        </div>

        <div class="sidebar-nav">
          <nuxt-link v-if="$auth.loggedIn" class="sidebar-nav-item" :to="{name:'user-feed', params:{user: $auth.user.username}}">
            <fa-icon icon="rss" /> <span>Feed</span>
          </nuxt-link>

          <nuxt-link class="sidebar-nav-item" :to="{name:'sort', params:{sort:'trending'}}">
            <fa-icon icon="fire" /> <span>Explore</span>
          </nuxt-link>

          <nuxt-link v-if="$config.CURATED_FEED && $config.CURATED_FEED_ACCOUNT !== ''" class="sidebar-nav-item" :to="{name:'sort', params:{sort:'curated'}}">
            <fa-icon icon="star" /> <span>Curator's Pick</span>
          </nuxt-link>

          <nuxt-link v-if="$config.NFT_ENABLED" class="sidebar-nav-item" :to="{name:'nfts'}">
            <fa-icon icon="image" /> <span>NFTs</span>
          </nuxt-link>

          <nuxt-link v-if="$config.DTF_ENABLED" class="sidebar-nav-item" :to="{name:'proposals'}">
            <fa-icon icon="file-alt" /> <span>Proposals</span>
          </nuxt-link>

          <nuxt-link v-if="$config.POOL_ENABLED" class="sidebar-nav-item" :to="{name:'pool'}">
            <fa-icon icon="exchange-alt" /> <span>Pool</span>
          </nuxt-link>

          <nuxt-link v-if="$auth.loggedIn" class="sidebar-nav-item" :to="{ name:'publish' }">
            <fa-icon icon="pencil-alt" /> <span>Create Post</span>
          </nuxt-link>

          <nuxt-link v-if="$auth.loggedIn" class="sidebar-nav-item" :to="{ name:'user', params: {user: $auth.user.username} }">
            <fa-icon icon="user" /> <span>Profile</span>
          </nuxt-link>

          <nuxt-link v-if="$auth.loggedIn" class="sidebar-nav-item" :to="{ name:'user-wallet', params: {user: $auth.user.username} }">
            <fa-icon icon="wallet" /> <span>Wallet</span>
          </nuxt-link>

          <nuxt-link v-if="$auth.loggedIn && $auth.user.username === issuer" class="sidebar-nav-item" :to="{ name:'dashboard' }">
            <fa-icon icon="tachometer-alt" /> <span>Dashboard</span>
          </nuxt-link>

          <div class="mt-4">
            <a href="#" class="sidebar-nav-item" @click.prevent="changeColorMode">
              <template v-if="$colorMode.value === 'light'">
                <fa-icon :icon="['far', 'moon']" /> <span>Dark Mode</span>
              </template>
              <template v-else>
                <fa-icon :icon="['far', 'sun']" /> <span>Light Mode</span>
              </template>
            </a>
          </div>
        </div>
      </div>

      <div class="content-area-wrapper">
        <div class="main-header">
          <div class="search-container">
            <div class="search-input-wrapper">
              <fa-icon class="search-icon" icon="search" />
              <input 
                type="text" 
                class="search-input" 
                placeholder="Search posts, tags, users..."
                @keyup.enter="performSearch"
                v-model="searchQuery"
              >
            </div>
          </div>

          <div class="header-actions">
            <button v-if="$config.NFT_ENABLED && $route.name && ($route.name.startsWith('nfts') || ['user-collection', 'user-gallery', 'user-collection-series', 'user-gallery-series'].includes($route.name))" 
              class="header-action-btn" 
              @click.prevent="$bvModal.show('activityModal')">
              <fa-icon icon="shopping-basket" />
              <div v-if="cart.length > 0" class="cart-item-count badge badge-primary">
                {{ cart.length }}
              </div>
            </button>

            <template v-if="$auth.loggedIn">
              <button class="header-action-btn" @click.prevent="$router.push({ name: 'user-notifications', params: { user: $auth.user.username } })">
                <fa-icon icon="bell" />
              </button>

              <div class="dropdown">
                <img 
                  :src="`${$config.IMAGES_CDN}u/${$auth.user.username}/avatar`" 
                  class="user-avatar" 
                  @click="toggleUserMenu"
                >
                <div v-if="showUserMenu" class="dropdown-menu show" style="position: absolute; right: 0;">
                  <nuxt-link class="dropdown-item" :to="{ name:'user', params: {user: $auth.user.username} }">
                    Profile
                  </nuxt-link>
                  <nuxt-link class="dropdown-item" :to="{ name:'user-wallet', params: {user: $auth.user.username} }">
                    Wallet
                  </nuxt-link>
                  <nuxt-link class="dropdown-item" :to="{ name:'user-settings', params: {user: $auth.user.username} }">
                    Settings
                  </nuxt-link>
                  <div class="dropdown-divider"></div>
                  <a class="dropdown-item" href="#" @click.prevent="$auth.logout()">
                    Logout
                  </a>
                </div>
              </div>
            </template>

            <template v-else>
              <button class="btn btn-primary ml-2" @click.prevent="$bvModal.show('loginModal')">
                Login
              </button>
              <button v-if="$config.OUTPOST_ONBOARD" class="btn btn-outline-primary ml-2" @click.prevent="$bvModal.show('signupModal')">
                Sign up
              </button>
            </template>
          </div>
        </div>

        <div class="main-content-card">
          <Nuxt />
        </div>
      </div>
    </div>

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
import { mapActions, mapGetters } from 'vuex'
import BackToTop from 'vue-backtotop'
import Login from '@/components/modals/Login.vue'

export default {
  name: 'MainLayout',

  components: {
    BackToTop,
    Login,
    SignUp: () => import(/* webpackChunkName: "SignUpModal" */ '@/components/modals/SignUp.vue')
  },

  data() {
    return {
      searchQuery: '',
      showUserMenu: false
    }
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
    
    // Close user menu when clicking outside
    document.addEventListener('click', this.closeUserMenuOnClickOutside)
  },
  
  beforeDestroy() {
    document.removeEventListener('click', this.closeUserMenuOnClickOutside)
  },

  computed: {
    ...mapGetters(['issuer']),
    ...mapGetters('nftmarketplace', ['cart'])
  },

  methods: {
    ...mapActions('user', ['fetchAccountScotData', 'fetchFollowers', 'fetchFollowing']),
    ...mapActions('scot', ['fetchTrendingTags']),
    
    changeColorMode () {
      if (this.$colorMode.value === 'dark') {
        this.$colorMode.preference = 'light'
      } else {
        this.$colorMode.preference = 'dark'
      }
    },
    
    performSearch() {
      if (this.searchQuery.trim().length > 0) {
        this.$router.push({ name: 'nfts-search', query: { q: this.searchQuery.trim() } })
      }
    },
    
    toggleUserMenu(event) {
      event.stopPropagation()
      this.showUserMenu = !this.showUserMenu
    },
    
    closeUserMenuOnClickOutside(event) {
      if (this.showUserMenu && !event.target.closest('.dropdown')) {
        this.showUserMenu = false
      }
    }
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

.dropdown-menu {
  min-width: 200px;
  padding: 0.5rem 0;
  margin: 0.5rem 0 0;
  border: none;
  border-radius: 0.5rem;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15);
}

.dropdown-item {
  padding: 0.5rem 1.5rem;
  font-weight: 500;
}

.dropdown-item:hover {
  background-color: #f8f9fa;
}

.cart-item-count {
  position: absolute;
  top: -5px;
  right: -5px;
  font-size: 0.7rem;
  min-width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
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
