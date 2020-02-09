<template>
  <div id="app">
    <loading
      :active.sync="isLoading">
    </loading>
    <router-view/>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  computed: {
    ...mapGetters(['isLoading']),
  },
};
</script>

<style lang="scss">
@import 'bootstrap/scss/bootstrap';
</style>
