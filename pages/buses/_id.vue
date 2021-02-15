<template>
      <b-container>
    <b-row align-v="center">
      <Carta 
        :id='item.id'
        :title='item.title'
        :text='item.text'
      />
    </b-row>
  </b-container>
</template>

<script>
import Carta from '../../components/Carta'
export default {
    data () {
        return {
            item: {
                'id': 0,
                'title': '',
                'text': ''
            }
        }
    },
    components () {
        Carta
    },
    async created () {
        const res = await this.$axios.$get('/bus/' + this.$route.params.id + '/');
        this.item.id = res.id;
        this.item.title = res.patente;
        this.item.text = res.asientos;
    }
}
</script>