<template>
    <div>
        <b-table hover :items="items" :fields="fields" @row-clicked="revisarAsientos">
            <template v-slot:cell(eliminarHorario)="data">
                <b-button variant="danger" @click="deleteItem(data.item.id)">X</b-button>
            </template>
            <template v-slot:cell(eliminarTrayecto)="data">
                <b-button variant="danger" @click="deleteTrayecto(data.item.id)">X</b-button>
            </template>
        </b-table>
        <hr>
        <b-form @submit="onSubmit">
            <b>Agregar Nuevo Horario</b>
            <hr>
            <p>Seleccionar Trayecto</p>
            <b-form-select v-model="trayecto" :options="trayectosForm" class="col-6 sm-2"></b-form-select>
            <b-form-input v-model="newTrayecto" placeholder="Nombre nuevo trayecto" v-show="!trayecto" class="col-6 sm-2"></b-form-input>
            <hr>
            <p>Fecha y Hora de Inicio</p>
            <b-form-datepicker id="datepickerInicio" v-model="fechaInicio" class="col-6"></b-form-datepicker>
            <b-form-timepicker id="timepickerInicio" v-model="horaInicio" class="col-6"></b-form-timepicker>
            <hr>
            <p>Fecha y Hora de Término</p>
            <b-form-datepicker id="datepickerFin" v-model="fechaFin" class="col-6"></b-form-datepicker>
            <b-form-timepicker id="timepickerFin" v-model="horaFin" class="col-6"></b-form-timepicker>
            <hr>
            <p>Seleccionar Bus</p>
            <b-form-select v-model="bus" :options="busesForm" class="col-6 sm-2"></b-form-select>
            <b-form-input v-model="newBus" placeholder="Patente del nuevo Bus" v-show="!bus" class="col-6 sm-2"></b-form-input>
            <hr>
            <p>Seleccionar Chofer</p>
            <b-form-select v-model="chofer" :options="choferesForm" class="col-6 sm-2"></b-form-select>
            <b-form-input v-model="newChofer" placeholder="Nombre del nuevo chofer" v-show="!chofer" class="col-6 sm-2"></b-form-input>
            <b-form-input v-model="newChoferRut" placeholder="Rut del nuevo chofer" v-show="!chofer" class="col-6 sm-2"></b-form-input>
            <hr>
            <b-button variant="primary" type="submit">Agregar Horario</b-button>
            <hr>
        </b-form>
    </div>
</template>

<script>
export default {
    data () {
        return {
            items: [],
            trayectos: {},
            trayectosForm: [
                {
                    'value': null,
                    'text': "Agregar Nuevo Trayecto"
                }
            ],
            buses: {},
            busesForm: [
                {
                    'value': null,
                    'text': "Agregar Nuevo Bus"
                }
            ],
            choferes: {},
            choferesForm: [
                {
                    'value': null,
                    'text': "Agregar Nuevo Chofer"
                }
            ],
            fields: ['id', 'trayecto', 'horaInicio', 'horaFin', 'bus', 'chofer', {key: 'capacidad', sortable: true}, 'eliminarHorario', 'eliminarTrayecto'],
            newTrayecto: '',
            newBus: '',
            newChofer: '',
            newChoferRut: '',
            trayecto: null,
            bus: null,
            chofer: null,
            fechaInicio: '',
            horaInicio: '',
            fechaFin: '',
            horaFin: '',
        }
    },
    async created() {
        const res = await this.$axios.$get('/horario/');
        const resTrayecto = await this.$axios.$get('/trayecto/');
        const resBus = await this.$axios.$get('/bus/');
        const resChofer = await this.$axios.$get('/chofer/');
        var i = 0;
        for (i=0; i < resTrayecto.length; i++){
            this.trayectos[resTrayecto[i]['id']] = resTrayecto[i];
            this.trayectosForm.push({'value': resTrayecto[i]["id"], 'text': resTrayecto[i]["nombre"]});
        }
        for (i=0; i < resBus.length; i++){
            this.buses[resBus[i]['id']] = resBus[i];
            this.busesForm.push({'value': resBus[i]["id"], 'text': resBus[i]["patente"]});
        }
        for (i=0; i < resChofer.length; i++){
            this.choferes[resChofer[i]['id']] = resChofer[i];
            this.choferesForm.push({'value': resChofer[i]["id"], 'text': resChofer[i]["nombre"]});
        }
        for (var i = 0; i < res.length; i++){
            if (res[i]["trayecto"] in this.trayectos){
                res[i]["trayecto"] = this.trayectos[res[i]["trayecto"]]['nombre'];
            }
            if (res[i]["bus"] in this.buses){
            res[i]["bus"] = this.buses[res[i]["bus"]]['patente'];
            }
            if (res[i]["chofer"] in this.choferes){
            res[i]["chofer"] = this.choferes[res[i]["chofer"]]['nombre'];
            }
        }
        this.items = res;
    },
    methods: {
        async deleteItem(id) {
            for (var i in this.items){
                if (this.items[i]['id'] == id){
                   this.items.splice(i, 1);
                   await this.$axios.$delete('/horario/' + id + '/');
                   break;
                }
            }
        },
        async deleteTrayecto(id) {
            for (var i in this.items){
                if (this.items[i]['id'] == id){
                   const res = await this.$axios.$get('/horario/' + id + '/');
                   await this.$axios.$delete('/trayecto/' + res.trayecto + '/');
                   this.$router.push('/trayectos/');
                   break;
                }
            }
        },
        revisarAsientos(record, index){
            this.$router.push('trayectos/' + record['id']);
        },
        async onSubmit (event){
            event.preventDefault();
            if (!this.trayecto){
                const resTray = await this.$axios.$post('/trayecto/', {'nombre': this.newTrayecto});
                this.trayecto = resTray.id;
                this.trayectos[resTray.id] = resTray;
                this.trayectosForm.push({'value': resTray.id, 'text': resTray.nombre})
            }
            if (!this.bus){
                const resBus = await this.$axios.$post('/bus/', {'patente': this.newBus});
                this.bus = resBus.id;
                this.buses[resBus.id] = resBus;
                this.busesForm.push({'value': resBus.id, 'text': resBus.patente});
            }
            if (!this.chofer){
                const resChof = await this.$axios.$post('/chofer/', {'nombre': this.newChofer, 'rut': this.newChoferRut.split('-')[0], 'cv': this.newChoferRut.split('-')[1]});
                this.chofer = resChof.id;
                this.choferes[resChof.id] = resChof;
                this.choferesForm.push({'value': resChof.id, 'text': resChof.nombre})
            }
            var res = await this.$axios.$post('/horario/', {
                'trayecto': this.trayecto,
                'bus': this.bus,
                'chofer': this.chofer,
                'horaInicio': this.fechaInicio + 'T' + this.horaInicio + 'Z',
                'horaFin': this.fechaFin + 'T' + this.horaFin + 'Z'
            })
            res['trayecto'] = this.trayectos[res['trayecto']].nombre;
            res['bus'] = this.buses[res['bus']].patente;
            res['chofer'] = this.choferes[res['chofer']].nombre;
            this.items.push(res);
            this.newTrayecto = '';
            this.newBus = '';
            this.newChofer = '';
            this.newChoferRut = '';
            this.trayecto = null;
            this.bus = null;
            this.chofer = null;
            this.fechaInicio = '';
            this.horaInicio = '';
            this.fechaFin = '';
            this.horaFin = ''
        }
    }
}
</script>