<template>
    <div>
        <b-table hover :items="items" :fields="fields">
            <template v-slot:cell(eliminar)="data">
                <b-button variant="warning" @click="deleteItem(data.item.id)">Liberar asiento</b-button>
            </template>
        </b-table>
        <hr>
        <b>Asignar Asiento</b>
        <hr>
        <b-form @submit="onSubmit">
            <b-form-select v-model="pasajero" :options="pasajerosForm" class="col-6 sm-2"></b-form-select>
            <b-form-input v-model="newPasajero" placeholder="Nombre del Nuevo Pasajero" v-show="!pasajero" class="col-6 sm-2"></b-form-input>
            <b-form-input v-model="newPasajeroRut" placeholder="Rut del Nuevo Pasajero" v-show="!pasajero" class="col-6 sm-2"></b-form-input>
            <b-form-select v-model="libre" :options="asientosLibres" class="col-6 sm-2"></b-form-select>
            <b-button variant="primary" type="submit">Asignar Asiento</b-button>
        </b-form>
        <hr>
        <b>Actualizar Bus</b>
        <b-form @submit="onSubmitBus">
            <b-form-select v-model="chofer" :options="choferesForm" class="col-6 sm-2"></b-form-select>
            <b-form-select v-model="bus" :options="busesForm" class="col-6 sm-2"></b-form-select>
            <b-button variant="primary" type="submit">Actualizar Datos</b-button>
        </b-form>
        <hr>
    </div>
</template>

<script>
export default {
    data () {
        return {
            items: [],
            horario: null,
            pasajeros: {},
            pasajerosForm: [
                {
                    'value': null,
                    'text': 'Agregar Nuevo Pasajero'
                }
            ],
            busesForm: [
                {
                    'value': null,
                    'text': 'Actualizar Bus'
                }
            ],
            choferesForm: [
                {
                    'value': null,
                    'text': 'Actualizar Chofer'
                }
            ],
            bus: null,
            chofer: null,
            newPasajero: '',
            newPasajeroRut: '',
            pasajero: null,
            fields: ['numero', 'pasajero', 'eliminar'],
            asientosLibres : [],
            libre: null
        }
    },
    async created() {
        const res = await this.$axios.$get('/asiento/?horario=' + this.$route.params.id);
        const resPasajeros = await this.$axios.$get('/pasajero/');
        var i = 0
        for (i = 0; i < resPasajeros.length; i++){
            this.pasajeros[resPasajeros[i]['id']] = resPasajeros[i];
            this.pasajerosForm.push({'value': resPasajeros[i].id, 'text': resPasajeros[i].nombre})
        }
        for (i = 0; i < res.length; i++){
            if (res[i]["pasajero"] in this.pasajeros){
                res[i]["pasajero"] = this.pasajeros[res[i]["pasajero"]]['nombre'];
            } else {
                this.asientosLibres.push({
                    'value': i + 1,
                    'text': i + 1
                })
            }
        }
        this.items = res;
        this.horario = await this.$axios.$get('/horario/' + this.$route.params.id + '/');
        const resBuses = await this.$axios.$get('/bus/');
        for (i=0; i < resBuses.length; i++){
            this.busesForm.push({
                'value': resBuses[i].id,
                'text': resBuses[i].patente
            })
        }
        const resChoferes = await this.$axios.$get('/chofer/');
        for (i=0; i < resChoferes.length; i++){
            this.choferesForm.push({
                'value': resChoferes[i].id,
                'text': resChoferes[i].nombre
            })
        }

    },
    methods: {
        async deleteItem(id) {
            for (var i in this.items){
                if (this.items[i]['id'] == id){
                   this.items[i]['pasajero'] = null;
                   await this.$axios.$put('/asiento/' + id + '/', {'numero': this.items[i]['numero'], 'horario': this.items[i]['horario'], 'pasajero': null});
                   this.asientosLibres.push({
                       'value': this.items[i]['numero'],
                       'text': this.items[i]['numero']
                   })
                   break;
                }
            }
        },
        async onSubmit(event) {
            event.preventDefault();
            for (var i = 0; i < this.items.length; i++){
                if (!this.pasajero){
                    const resPasajero = await this.$axios.$post('/pasajero/', {'nombre': this.newPasajero, 'rut': this.newPasajeroRut.split('-')[0], 'cv': this.newPasajeroRut.split('-')[1]});
                    this.pasajeros[resPasajero.id] = resPasajero;
                    this.pasajerosForm.push({
                        'value': resPasajero.id,
                        'text': resPasajero.nombre
                    });
                    this.pasajero = resPasajero.id
                }
                if (this.items[i].numero == this.libre){
                    const res = await this.$axios.$get('/asiento/' + this.items[i].id + '/');
                    const data = {
                        'numero': res.numero,
                        'pasajero': this.pasajero,
                        'horario': res.horario
                    };
                    const resP = await this.$axios.$put('/asiento/' + this.items[i].id + '/', data);
                    this.items[i].pasajero = this.pasajeros[resP.pasajero].nombre;
                    this.pasajero = null;
                    this.newPasajero = '';
                    this.newPasajeroRut = '';
                    for (var j = 0; j < this.asientosLibres.length; j++){
                        if (this.asientosLibres[j]['value'] == this.libre){
                            this.asientosLibres.splice(j, 1);
                            break;
                        }
                    }
                    this.libre= null;
                    break;
                }
            }
        },
        async onSubmitBus (event) {
            event.preventDefault();
            if (this.bus){
                this.horario.bus = this.bus
            }
            if (this.chofer){
                this.horario.chofer = this.chofer
            }
            this.horario = await this.$axios.$put('/horario/' + this.$route.params.id + '/', this.horario);
            this.bus = null;
            this.chofer = null;
        }
    }
}
</script>