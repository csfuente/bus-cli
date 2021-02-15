<template>
    <div>
        <b-table hover :items="items" :fields="fields" @row-clicked="revisarBus">
            <template v-slot:cell(eliminar)="data">
                <b-button variant="danger" @click="deleteItem(data.item.id)">Eliminar Bus</b-button>
            </template>
        </b-table>
        <b-form inline @submit="onSubmit">
            <label class="sr-only" for="inline-form-input-name">Agregar Nueva Patente de bus </label>
            <b-form-input id="inline-form-input-name" placeholder="Patente" class="mb-2 mr-sm-2 mb-sm-0" v-model="form.patente"/>
            <b-button variant="primary" type="submit">Agregar</b-button>
        </b-form>
    </div>
</template>

<script>
export default {
    data () {
        return {
            items: [],
            fields: ['patente', 'eliminar'],
            form: {
                'patente': ''
            }
        }
    },
    async created() {
        const res = await this.$axios.$get('/bus/');
        this.items = res
    },
    methods: {
        async deleteItem(id) {
            for (var i in this.items){
                if (this.items[i]['id'] == id){
                   this.items.splice(i, 1);
                   await this.$axios.$delete('/bus/' + id + '/')
                   break;
                }
            }
        },
        revisarBus(record, index){
            this.$router.push('buses/' + record['id']);
        },
        async onSubmit(event){
            event.preventDefault();
            const res = await this.$axios.$post('/bus/', {'patente': this.form.patente});
            this.items.push(res);
            this.form.patente = ''
        }
    }
}
</script>