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
          >{{ title }}</a
        >
      </div>
      <div id="content-header" class="nhsuk-header__content"></div>
    </div>
  </header>
</template>

<script>
export default {
  data() {
    const path = this.$router.currentRoute.path
    const title = path.indexOf('tki') > -1 ? 'Thomas Kilman Conflict Mode Instrument (TKI)' : 'Merrill & Reid Personal Style Inventory'
    return {
      title: title
    }
  }
}
</script>

<style lang="scss">
@import 'node_modules/nhsuk-frontend/packages/components/header/header';
</style>
