<template lang="pug">
  q-layout.bg-dark(view="lHh Lpr lFf")
    q-header
      q-toolbar.bg-dark
        q-btn(flat dense round icon="fas fa-history" aria-label="Menu" @click="leftDrawerOpen = !leftDrawerOpen")

        q-toolbar-title Calculator

    q-drawer(v-model="leftDrawerOpen"  dark show-if-above bordered)
      History
    q-page-container
      router-view
</template>

<script>
import History from '../components/History'
export default {
  name: 'MainLayout',
  components: {
    History
  },
  data () {
    return {
      leftDrawerOpen: false
    }
  }
}
</script>
