<template>
  <header>
    <!-- Mobile menu toggle button - only visible on small screens -->
    <button 
      class="mobile-menu-toggle d-md-none" 
      @click="toggleMobileMenu"
    >
      <fa-icon icon="bars" />
    </button>
    
    <!-- User dropdown for mobile - only visible on small screens when logged in -->
    <div v-if="$auth.loggedIn" class="mobile-user-menu d-md-none">
      <img 
        :src="`${$config.IMAGES_CDN}u/${$auth.user.username}/avatar`" 
        class="user-avatar" 
        @click="toggleUserMenu"
      >
      <div v-if="showUserMenu" class="dropdown-menu show">
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
  </header>
</template>

<script>
import { mapGetters } from 'vuex'

export default {
  name: 'Header',
  
  data() {
    return {
      showUserMenu: false
    }
  },

  computed: {
    ...mapGetters(['issuer']),
    ...mapGetters('nftmarketplace', ['cart'])
  },

  mounted() {
    document.addEventListener('click', this.closeUserMenuOnClickOutside)
  },
  
  beforeDestroy() {
    document.removeEventListener('click', this.closeUserMenuOnClickOutside)
  },

  methods: {
    toggleMobileMenu() {
      // Emit an event to toggle the mobile sidebar
      this.$eventBus.$emit('toggle-mobile-sidebar')
    },
    
    toggleUserMenu(event) {
      event.stopPropagation()
      this.showUserMenu = !this.showUserMenu
    },
    
    closeUserMenuOnClickOutside(event) {
      if (this.showUserMenu && !event.target.closest('.mobile-user-menu')) {
        this.showUserMenu = false
      }
    }
  }
}
</script>

<style lang="scss">
.mobile-menu-toggle {
  position: fixed;
  top: 1rem;
  left: 1rem;
  z-index: 1100;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #ffffff;
  border: none;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  color: #6c757d;
  
  &:focus {
    outline: none;
  }
}

.mobile-user-menu {
  position: fixed;
  top: 1rem;
  right: 1rem;
  z-index: 1100;
  
  .dropdown-menu {
    position: absolute;
    right: 0;
    left: auto;
    top: 45px;
  }
}

@media (min-width: 768px) {
  .mobile-menu-toggle,
  .mobile-user-menu {
    display: none !important;
  }
}
</style>