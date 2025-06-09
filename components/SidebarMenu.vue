<template>
  <div class="sidebar-menu-container d-none d-md-block">
    <!-- Mobile sidebar content is handled in layouts/default.vue -->
  </div>
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
// Styles are now in _layout.scss
</style>