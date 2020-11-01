<template lang="pug">
    q-list.text-white(separator)
        .row.items-center.justify-between
            q-item-label.text-white(header) History
            q-btn(icon="delete" @click="deleteHistory" color="grey" size=".75rem" flat)
        q-item(v-for="item in reverseChronologicalHistory" :key="item.id")
            q-item-section
                .text-center {{item.sum}}
</template>
<script>
import { db } from 'boot/localbase'
import { mapState, mapActions } from 'vuex'
export default {
  mounted () {
    this.getHistory()
  },
  methods: {
    ...mapActions('history', ['setHistory']),

    async getHistory () {
      try {
        const history = await db.collection('history').get()
        this.setHistory([...history])
      } catch (error) {
        this.$q.notify({
          message: 'Error retrieving history',
          color: 'negative',
          icon: 'fas fa-times'
        })
      }
    },
    async deleteHistory () {
      try {
        await db.collection('history').delete()
        this.setHistory([])
        this.$q.notify({
          message: 'History deleted',
          color: 'positive',
          icon: 'fas fa-check'
        })
      } catch (error) {
        this.$q.notify({
          message: 'Error deleting history',
          color: 'negative',
          icon: 'fas fa-times'
        })
      }
    }
  },
  computed: {
    ...mapState('history', ['history']),
    reverseChronologicalHistory () {
      return [...this.history].sort((a, b) => b.createdAt - a.createdAt)
    }
  }
}
</script>
