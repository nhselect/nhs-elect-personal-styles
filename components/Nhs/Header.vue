<template>
  <header class="nhsuk-header nhsuk-header" role="banner">
    <div class="nhsuk-width-container nhsuk-header__container">
      <div class="nhsuk-header__logo nhsuk-header__logo--only">
        <a class="nhsuk-header__link" href="/" aria-label="NHS homepage">
          <NhsElectLogo inverted />
        </a>
      </div>
      <div class="nhsuk-header__transactional-service-name">
        <a class="nhsuk-header__transactional-service-name--link" href="/"
          >Merrill & Reid Personal Style Inventory</a
        >
      </div>
      <div id="content-header" class="nhsuk-header__content"></div>
    </div>
  </header>
</template>

<script>
export default {
  computed: {
    isUserGuide() {
      const path = this.$router.currentRoute.path
      return path.indexOf('user-guide/') > 0
    },
    role() {
      const path = this.$router.currentRoute.path
      if (path.indexOf('user-guide/')) {
        const role = path.split('/')[2]
        if (role) {
          return role
        } else {
          return ''
        }
      }
      return ''
    },
  },
}
</script>

<style lang="scss">
@import 'node_modules/nhsuk-frontend/packages/components/header/header';
</style>
