<template>
  <div id="app">
    <mcv-topbar />
    <router-view></router-view>
    <mcv-footer />
  </div>
</template>

<script>
import McvTopbar from '@/components/Topbar'
import {actionTypes} from '@/store/modules/auth'
import McvFooter from '@/components/Footer'

export default {
  name: 'App',
  components: {
    McvTopbar,
    McvFooter
  },

  mounted() {
    this.$store.dispatch(actionTypes.getCurrentUser)
  }
}
</script>
