<template>
    <div class="row">
        <div class="col-sm-6">
             <div class="card">
                <div class="card-body">
                    <div class="row">
                        <div class="col-12">
                            **Escanear Codigo con PDA**
                            <form v-on:submit.prevent="guardar(codigo_barras)">
                                <Input title="Codigo de Barras" type="number" :focusSelect="status_focus" v-model="codigo_barras"></Input>
                                <button type="submit" hidden></button>
                            </form>
                            <div v-if="alert!=null" :class="'alert alert-'+alert.status" role="alert">
                                {{ alert.data }}
                            </div>
                        </div>
                    </div>
                </div>
             </div>
        </div>
        
    </div>
</template>
<script>
import { mapState,mapMutations } from 'vuex'

import Input from '../dragon-desing/dg-input.vue'
export default {
    components:{
        Input
    },
    data() {
        return {
            status_focus: true,
            turnos:[],
            codigo_barras: null,
            respuesta: null,
            // turno_id: 0,
            alert: null,
            error: null,
            barcode: '',
            reading: false
        }
    },
    computed: {
        ...mapState(['cuenta']),
    },
    created() {
        document.addEventListener("keydown",this.pda);
    },
    beforeDestroy() {
        document.removeEventListener("keydown",this.pda);
    },
    methods: {
        pda(e) {
            const textInput = e.key || String.fromCharCode(e.keyCode);
            if (textInput=="Enter"&&this.barcode!="") {
                // console.log(this.barcode);
                this.guardar(this.barcode);
                // alert(this.barcode);
                this.barcode="";
            }else if (textInput!="Enter") {
                this.barcode+=textInput;
            }

            if(!this.reading){
                this.reading=true;
                setTimeout(()=>{
                    this.barcode="";
                    this.reading=false;
                }, 200);
            }
        },
        url(foto){
            return url_base+'/../storage/operador/'+foto;
        },
        guardar(codigo){
            this.$nextTick(() =>{
                if (codigo.length==8) {
                    this.codigo_barras=null;
                    var t=this;

                    db.transaction((tx)=>{
                        var query_add="INSERT INTO MARCAS(codigo_operador,fecha,fundo_id) VALUES (?,?,?)";
                        var params=[codigo,moment().format('YYYY-MM-DD HH:mm'),t.cuenta.fundo_id];
                        tx.executeSql(query_add, params, function(tx, res) {
                            t.marcaRegistrada();
                        });
                    });
                    //     console.log(cod_barras_paso);
                    //     /**
                    //      * Comprobacion.
                    //      */
                    //     var fecha_consulta=(moment().format('HH')<moment().format('07')) ? moment().subtract(1,'day').format('YYYY-MM-DD') : moment().format('YYYY-MM-DD');
                        
                        
                        

                    //     tx.executeSql('SELECT rowid,*,datetime("now","-14 hour") FROM MARCADOR WHERE codigo_operador=? AND ingreso > ?  ORDER BY rowid DESC LIMIT 1', [cod_barras_paso,moment().subtract(15,'hour').format('YYYY-MM-DD')], function (tx, results) {

                    //         var anterior_marca=null;
                            
                    //         for (let j = 0; j < results.rows.length; j++) {
                    //             anterior_marca = results.rows.item(j);
                    //             console.log(anterior_marca);
                                
                    //         }

                    //         if (anterior_marca!=null) {
                    //             // var fecha_limite = moment().subtract(1,'minutes');
                                
                    //             if(
                    //                 (anterior_marca.salida==null&&moment().diff(anterior_marca.ingreso,'minutes')<1)
                    //                 ||(anterior_marca.salida!=null&&moment().diff(anterior_marca.salida,'minutes')<1)) {
                    //                 t.alert={
                    //                     status: "warning",
                    //                     data: "Usted Marco recientemente"
                    //                 }
                    //             }else{
                    //                 var hora_fecha_actual=moment();
                    //                 var hora_fecha_limite=moment().startOf('day').add(8,'hours');
                    //                 var fecha_ayer=moment().subtract(1,'days').format('YYYY-MM-DD');
                    //                 if (
                    //                     anterior_marca==null||
                    //                     anterior_marca.salida!=null||
                    //                     (
                    //                         anterior_marca.salida==null&&
                    //                         fecha_ayer==moment(anterior_marca.ingreso).format('YYYY-MM-DD')&&
                    //                         hora_fecha_actual>hora_fecha_limite
                    //                     )
                    //                 ) 
                    //                 {
                    //                     /**
                    //                      * Agregar Marca
                    //                      */
                    //                     tx.executeSql('INSERT INTO MARCADOR (codigo_operador,ingreso,fecha_ref,fundo_id,enviado,cuenta_id) VALUES (?,?,?,?,"NO",?)',
                    //                         [
                    //                             cod_barras_paso,
                    //                             moment().format('YYYY-MM-DD HH:mm'),
                    //                             moment().format('YYYY-MM-DD'),
                    //                             t.cuenta.fundo_id,
                    //                             t.cuenta.id
                    //                         ]
                    //                     ,t.marcaRegistrada); 
                    //                 }else{
                    //                     /**
                    //                      * Actualizar Marca
                    //                      */
                    //                     tx.executeSql('UPDATE MARCADOR SET salida="'+moment().format('YYYY-MM-DD HH:mm')+'" WHERE rowid=?',[anterior_marca.rowid],t.marcaRegistrada);
                    //                 }
                    //             }
                    //         }else{
                    //             tx.executeSql('INSERT INTO MARCADOR (codigo_operador,ingreso,fecha_ref,fundo_id,enviado,cuenta_id) VALUES (?,?,?,?,"NO",?)',
                    //                 [
                    //                     cod_barras_paso,
                    //                     moment().format('YYYY-MM-DD HH:mm'),
                    //                     moment().format('YYYY-MM-DD'),
                    //                     t.cuenta.fundo_id,
                    //                     t.cuenta.id
                    //                 ],
                    //             t.marcaRegistrada,
                    //             errorCB
                    //             );                                
                    //         }

                    //     });
                    // });
                }else{
                    this.codigo_barras=null;
                    this.alert={
                        status: 'danger',
                        data: 'Código no Valido'
                    }
                    this.clearAlert();
                }
            })
        },
        marcaRegistrada(){
            this.alert={
                status: 'info',
                data: 'Marca registrada.'
            };
        },
        clearAlert(){
            setTimeout(() => {
                this.alert=null;
            }, 10000);
        }
    },
}
</script>