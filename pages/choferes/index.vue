<template>
    <div>
        <b-table hover :items="items" :fields="fields" @row-clicked="revisarChofer">
            <template v-slot:cell(eliminar)="data">
                <b-button variant="danger" @click="deleteItem(data.item.id)">Eliminar Chofer</b-button>
            </template>
        </b-table>
        <b-form inline @submit="onSubmit">
            <label class="sr-only" for="inline-form-input-name">Nombre Chofer</label>
            <b-form-input id="inline-form-input-name" placeholder="Nombre" class="mb-2 mr-sm-2 mb-sm-0" v-model="form.nombre"/>
            <label class="sr-only" for="inline-form-input-name">Rut Chofer</label>
            <b-form-input id="inline-form-input-name" placeholder="Rut" class="mb-2 mr-sm-2 mb-sm-0" v-model="form.rut"/>
            <b-button variant="primary" type="submit">Agregar</b-button>
        </b-form>
    </div>
</template>

<script>
export default {
    data () {
        return {
            items: [],
            fields: ['nombre', 'rut', 'cv', 'eliminar'],
            form: {
                'nombre': '',
                'rut': ''
            }
        }
    },
    async created() {
        const res = await this.$axios.$get('/chofer/');
        this.items = res
    },
    methods: {
        async deleteItem(id) {
            for (var i in this.items){
                if (this.items[i]['id'] == id){
                   this.items.splice(i, 1);
                   await this.$axios.$delete('/chofer/' + id + '/');
                   break;
                }
            }
        },
        revisarChofer(record, index){
            this.$router.push('choferes/' + record['id']);
        },
        async onSubmit(event){
            event.preventDefault();
            if (this.form.rut.includes('-')){
                const res = await this.$axios.$post('/chofer/', {'nombre': this.form.nombre, 'rut': this.form.rut.split('-')[0], 'cv': this.form.rut.split('-')[1]});
                this.items.push(res);
                this.form.nombre = '';
                this.form.rut = '';
            }
        }
    }
}
</script>