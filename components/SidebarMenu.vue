<template>
  <b-sidebar
    id="sidebar-menu"
    right
    shadow
    bg-variant="primary"
    text-variant="light"
    class="sidebar-menu"
    backdrop
  >
    <div class="p-3">
      <div class="d-flex align-items-center justify-content-between mb-4">
        <h5 class="m-0">Menu</h5>
        <b-button variant="link" class="p-0 text-light" @click="$root.$bvToggle.toggle('sidebar-menu')">
          <fa-icon icon="times" />
        </b-button>
      </div>

      <b-list-group class="d-lg-none" flush>
        <b-list-group-item v-if="$auth.loggedIn" :to="{name:'user-feed', params:{user: $auth.user.username}}">
          <fa-icon icon="rss" class="mr-2" /> Feed
        </b-list-group-item>

        <b-list-group-item :to="{name:'sort', params:{sort:'trending'}}">
          <fa-icon icon="fire" class="mr-2" /> Explore
        </b-list-group-item>

        <b-list-group-item v-if="$config.CURATED_FEED && $config.CURATED_FEED_ACCOUNT !== ''" :to="{name:'sort', params:{sort:'curated'}}">
          <fa-icon icon="star" class="mr-2" /> Curator's Pick
        </b-list-group-item>

        <b-list-group-item v-if="$config.NFT_ENABLED" :to="{name:'nfts'}">
          <fa-icon icon="image" class="mr-2" /> NFTs
        </b-list-group-item>

        <b-list-group-item v-if="$config.DTF_ENABLED" :to="{name:'proposals'}">
          <fa-icon icon="file-alt" class="mr-2" /> Proposals
        </b-list-group-item>

        <b-list-group-item v-if="$config.POOL_ENABLED" :to="{name:'pool'}">
          <fa-icon icon="exchange-alt" class="mr-2" /> Pool
        </b-list-group-item>
      </b-list-group>

      <b-list-group class="mt-3" flush>
        <b-list-group-item class="font-weight-bold" disabled>
          Trade
        </b-list-group-item>

        <b-list-group-item target="_blank" :href="`https://tribaldex.com/trade/${$config.TOKEN}`">
          <fa-icon icon="chart-line" class="mr-2" /> Trade {{ $config.TOKEN }} <fa-icon icon="external-link-alt" class="ml-1" />
        </b-list-group-item>
      </b-list-group>
    </div>

    <template #footer>
      <div class="text-center pb-3 d-md-none">
        <b-button size="sm" @click.prevent="changeColorMode">
          <template v-if="$colorMode.value === 'light'">
            <fa-icon :icon="['far', 'moon']" /> Dark Mode
          </template>

          <template v-else>
            <fa-icon :icon="['far', 'sun']" /> Light Mode
          </template>
        </b-button>
      </div>
    </template>
  </b-sidebar>
</template>

<script>
export default {
  name: 'SidebarMenu',

  methods: {
    changeColorMode () {
      if (this.$colorMode.value === 'dark') {
        this.$colorMode.preference = 'light'
      } else {
        this.$colorMode.preference = 'dark'
      }
    }
  }
}
</script>

<style lang="scss">
.sidebar-menu {
  .svg-inline--fa.fa-external-link-alt {
    width: 12px;
    margin-left: 5px;
    padding-top: 3px;
  }
}
</style>