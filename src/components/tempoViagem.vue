<template>
    <div class="regras montserrat">
        <b-modal
            class="montserrat"
            v-model="modalNovoGrupo"
            title="Tempo de Viagem"
            size="lg"
            hide-footer
            @hidden="cleanModal"
        >
            <b-row align-v="end">
                <b-col>
                    <label for="input-cod">Informe o Código de Viagem</label>
                    <b-form-input id="input-cod" v-model="codViagem" type="text"></b-form-input>
                </b-col>
                <b-col>
                    <b-button variant="success" @click="getApi">Confirmar</b-button>
                </b-col>
            </b-row>
            <br />
            <b-row>
                <b-col>
                    <div v-if="mostrar">
                        Esse veículo demorou <strong>{{ tempoAteAduana }}</strong> para ir de
                        <strong>{{ cidade_origem }}</strong>
                        até Aduana de <strong>{{ nomeAduana }}</strong> na rota
                        <strong>{{ numeroRota }}</strong>
                    </div>
                </b-col>
            </b-row>
        </b-modal>

        <b-card>
            <b-row align-v="center">
                <b-col>
                    <label for="example-datepicker">Data Inicio</label>
                    <b-form-datepicker id="datepickerInicio" v-model="dataInicioBusca" class="mb-2"></b-form-datepicker>
                </b-col>
                <b-col>
                    <label for="example-datepicker">Data Fim</label>
                    <b-form-datepicker id="datepickerFim" v-model="dataFimBusca" class="mb-2"></b-form-datepicker>
                </b-col>
                <b-col style="margin-top: 21px"
                    ><b-button variant="danger" @click="getDestinacaoHistorico">Buscar</b-button></b-col
                >
            </b-row>
            <br />
            <b-row>
                <b-col sm="12" md="3" align-self="center">
                    <b-input-group size="sm" class="mb-2">
                        <b-input-group-prepend is-text>
                            <b-icon icon="search"></b-icon>
                        </b-input-group-prepend>
                        <b-form-input
                            id="filter-text-box"
                            class="form-control inputPesquisa"
                            type="search"
                            placeholder="Busque por qualquer termo..."
                        ></b-form-input>
                    </b-input-group>
                </b-col>
                <b-col align-self="end">
                    <div v-if="mostrar" style="font-size: small; margin-bottom: 15px">
                        Esse veículo demorou <strong>{{ tempoAteAduana }}</strong> para ir de
                        <strong>{{ cidade_origem }}</strong>
                        até Aduana de <strong>{{ nomeAduana }}</strong> na rota
                        <strong>{{ numeroRota }}</strong>
                    </div>
                    <!-- <div v-else style="font-size: small; margin-bottom: 10px">
                        Clique em um frota para ver tempo de viagem do cliente até a Aduana
                    </div> -->
                </b-col>
                <b-col sm="12" md="2" offset-md="1">
                    <!-- <b-button @click="addGrupo" variant="success" style="float: right; margin-bottom: 15px">
                        <b-icon icon="clock" aria-hidden="true"></b-icon> Análise Tempo</b-button
                    > -->
                </b-col>
            </b-row>
            <b-row>
                <b-col>
                    <ag-grid-vue
                        style="width: auto; height: 75vh"
                        class="ag-theme-alpine"
                        :columnDefs="columnDefs"
                        :defaultColDef="defaultColDef"
                        :rowData="rowData"
                        :animateRows="true"
                        :cacheQuickFilter="true"
                        @grid-ready="onGridReady"
                        :rowSelection="rowSelection"
                        @selection-changed="onSelectionChanged"
                        :masterDetail="true"
                        :detailCellRendererParams="detailCellRendererParams"
                    >
                        <!-- :getContextMenuItems="getContextMenuItems" -->
                    </ag-grid-vue>
                </b-col>
            </b-row>
        </b-card>
    </div>
</template>

<script>
import 'ag-grid-community/styles/ag-grid.css';
import 'ag-grid-community/styles/ag-theme-alpine.css';
import 'ag-grid-enterprise';
import { AgGridVue } from 'ag-grid-vue';
import axios from 'axios';

export default {
    name: 'TempoViagem',
    components: { AgGridVue },
    computed: {},
    created() {
        this.rowSelection = 'single';
    },
    watch: {
        resultInfoBusca() {
            console.log('watch', this.resultInfoBusca);
        },
    },

    data() {
        return {
            mostrar: false,
            modalNovoGrupo: false,
            cidade_origem: '',
            tempoAteAduana: '',
            nomeAduana: '',
            numeroRota: '',

            dataInicioBusca: '',
            dataFimBusca: '',

            codViagem: '',
            resultBusca: '',

            // TAGS RELACIONADOS AO AGGRID
            arrayLinhaSelecionada: [],
            detailCellRendererParams: null,
            columnDefs: null,
            rowData: [],
            gridApi: null,
            searchInput: '',
            rowSelection: null,
            defaultColDef: {
                initialWidth: 200,
                sortable: true,
                resizable: true,
                menuTabs: ['filterMenuTab'],
                filter: true,
                // flex: 1,
                // floatingFilter: true,
            },
        };
    },
    methods: {
        // FUNÇÕES RELACIONADAS AO AGGRID
        onSelectionChanged() {
            const selectedRows = this.gridApi.getSelectedRows();
            this.arrayLinhaSelecionada = selectedRows;

            this.codViagem = this.arrayLinhaSelecionada[0].id;
            this.getApi();

            console.log(this.arrayLinhaSelecionada[0].id);
        },
        onGridReady(params) {
            this.gridApi = params.api;
            this.gridColumnApi = params.columnApi;
        },
        onFilterTextBoxChanged() {
            this.gridApi.setQuickFilter(this.searchInput);
        },
        onPrintQuickFilterTexts() {
            this.gridApi.forEachNode(function (rowNode, index) {
                console.log('Row ' + index + ' quick filter text is ' + rowNode.quickFilterAggregateText);
            });
        },
        // getContextMenuItems(params) {
        //     this.gridApi.deselectAll();
        //     params.node.setSelected(true);

        //     var result = [
        //         {
        //             name: 'Editar',
        //             action: () => {
        //                 this.flagForm = 'edit';
        //                 this.editarRegistro(this.arrayLinhaSelecionada[0]);
        //             },
        //         },
        //         {
        //             name: 'Excluir',
        //             action: () => {
        //                 this.excluirRegistro(params.node.data.objectId);
        //             },
        //         },
        //     ];
        //     return result;
        // },
        //
        addGrupo() {
            this.flagForm = 'add';
            this.modalNovoGrupo = true;
        },
        cleanModal() {
            this.form.clientes = [];
            this.form = { ...this.formDefault };
            this.text = '';
        },
        async salvar(event) {
            event.preventDefault();

            this.flagForm == 'add'
                ? this.salvarApi()
                : this.confirmarEdicaoRegistro(this.arrayLinhaSelecionada[0].objectId);
        },
        notificacao(titulo, body, variant) {
            this.$bvToast.toast(body, {
                title: titulo,
                variant: variant,
                autoHideDelay: 7000,
                solid: true,
                toaster: 'b-toaster-top-right',
                appendToast: false,
                // headerClass: 'notificação',
                // bodyClass: 'notificação',
            });
        },
        async getApi() {
            const options = {
                method: 'GET',
                url: `https://api.marvel.app.br/destinacao/listar/${this.codViagem}`,
                headers: {
                    'Content-Type': 'application/json',
                    UserID: '5fb7b0e8118d0f006fa035a3',
                    UserEmail: 'andre@gobek.com.br',
                    Authorization: 'Bearer Basic das3SASSDC23c+=',
                },
            };
            const response = await axios.request(options);
            this.resultInfoBusca = response.data;

            var eventos = response.data.eventos,
                dataSaidaCliente = '',
                dataChegadaAduana = '';

            for (var item of eventos) {
                if (!dataSaidaCliente || !dataChegadaAduana) {
                    if (item.nome == 'saida' && item.tipo_local == 'coleta') {
                        var saidaClienteSegundos = Date.parse(item.data_hora);
                        dataSaidaCliente = new Date(item.data_hora).toLocaleString();
                    }
                    if (item.nome == 'chegada' && item.tipo_ponto == 'aduana') {
                        var chegadaAduanaSegundos = Date.parse(item.data_hora);
                        dataChegadaAduana = new Date(item.data_hora).toLocaleString();
                    }
                } else {
                    break;
                }
            }
            if (dataSaidaCliente > dataChegadaAduana) {
                this.notificacao(
                    'Eventos fora de Ordem!',
                    "O evento 'Chegada na Aduana' foi baixado antes do 'Saida do Cliente'"
                );
                return false;
            }

            console.log('dataSaidaCliente', dataSaidaCliente);
            console.log('dataChegadaAduana', dataChegadaAduana);
            var diferencaSegundos = chegadaAduanaSegundos - saidaClienteSegundos;

            var diferencaMinutos = Math.round(diferencaSegundos / 60000);
            console.log(diferencaMinutos, 'minutos');

            var diferencaHoras = Math.round(diferencaMinutos / 60);
            console.log(diferencaHoras, 'horas');

            var diferencaDias = Math.round(diferencaHoras / 24);
            console.log(diferencaDias, 'dias');

            if (diferencaMinutos > 60) {
                if (diferencaHoras > 24) {
                    this.tempoAteAduana = `${diferencaDias} dias`;
                } else {
                    this.tempoAteAduana = `${diferencaHoras} horas`;
                }
            } else {
                this.tempoAteAduana = `${diferencaMinutos} minutos`;
            }

            // console.log(item);

            this.cidade_origem = response.data.coletas[0].cidade_origem;
            this.nomeAduana = response.data.talhao;
            this.numeroRota = response.data.rota;
            console.log(response.data);
            this.mostrar = true;
        },
        async getTempoViagem(codViagem) {
            var tempoAteAduana = '';

            const options = {
                method: 'GET',
                url: `https://api.marvel.app.br/destinacao/listar/${codViagem}`,
                headers: {
                    'Content-Type': 'application/json',
                    UserID: '5fb7b0e8118d0f006fa035a3',
                    UserEmail: 'andre@gobek.com.br',
                    Authorization: 'Bearer Basic das3SASSDC23c+=',
                },
            };
            const response = await axios.request(options);

            var eventos = response.data.eventos,
                dataSaidaCliente = '',
                dataChegadaAduana = '';

            for (var item of eventos) {
                if (!dataSaidaCliente || !dataChegadaAduana) {
                    if (item.nome == 'saida' && item.tipo_local == 'coleta') {
                        var saidaClienteSegundos = Date.parse(item.data_hora);
                        dataSaidaCliente = new Date(item.data_hora).toLocaleString();
                    }
                    if (item.nome == 'chegada' && item.tipo_ponto == 'aduana') {
                        var chegadaAduanaSegundos = Date.parse(item.data_hora);
                        dataChegadaAduana = new Date(item.data_hora).toLocaleString();
                    }
                } else {
                    break;
                }
            }
            if (dataSaidaCliente > dataChegadaAduana) {
                tempoAteAduana = 'Eventos fora de Ordem!';
                return false;
            }

            var diferencaSegundos = chegadaAduanaSegundos - saidaClienteSegundos;

            var diferencaMinutos = Math.round(diferencaSegundos / 60000);
            console.log(diferencaMinutos, 'minutos');

            var diferencaHoras = Math.round(diferencaMinutos / 60);
            console.log(diferencaHoras, 'horas');

            var diferencaDias = Math.round(diferencaHoras / 24);
            console.log(diferencaDias, 'dias');

            if (diferencaMinutos > 60) {
                if (diferencaHoras > 24) {
                    tempoAteAduana = `${diferencaDias} dias`;
                } else {
                    tempoAteAduana = `${diferencaHoras} horas`;
                }
            } else {
                tempoAteAduana = `${diferencaMinutos} minutos`;
            }

            return tempoAteAduana;
        },
        async getDestinacaoHistorico() {
            const options = {
                method: 'GET',
                url: `https://api.marvel.app.br/destinacao/historico/${this.dataInicioBusca}/${this.dataFimBusca}/fim`,
                headers: {
                    'Content-Type': 'application/json',
                    UserID: '5fb7b0e8118d0f006fa035a3',
                    UserEmail: 'andre@gobek.com.br',
                    Authorization: 'Bearer Basic das3SASSDC23c+=',
                },
            };
            const response = await axios.request(options);

            var responseCompleto = [];
            for (var item of response.data) {
                var objCadaLinha = {
                    rota: item.rota,
                    id: item.id,
                    cavalo_fim: item.cavalo_fim,
                    carreta_fim: item.carreta_fim,
                    cidade_origem: item.cidade_origem,
                    cidade_destino: item.cidade_destino,
                    talhao: item.talhao,
                };

                responseCompleto.push(objCadaLinha);
                console.log(responseCompleto.length);
            }
            // pega codigo de viagem
            // faz consulta na api
            // o retorno trnaforma em dias
            // joga os dias em uma propriedade do mesmo item
            // numeroColeta;

            this.rowData = responseCompleto;
            console.log(responseCompleto);
        },
    },
    beforeMount() {
        // AGGRID TODOS OS GRUPOS
        this.columnDefs = [
            { field: 'id', headerName: 'Cod Viagem', width: 130 },
            { field: 'rota', headerName: 'Rota', width: 90 },
            { field: 'cavalo_fim', headerName: 'Frota', width: 90 },
            { field: 'carreta_fim', headerName: 'Carreta', width: 100 },
            { field: 'cidade_origem', headerName: 'Origem', width: 200 },
            { field: 'cidade_destino', headerName: 'Destino', width: 200 },
            { field: 'talhao', headerName: 'Aduana', width: 200 },
            { colId: 'saidaClienteAduana', headerName: 'Saida Cliente > Aduana', width: 200 },
        ];
    },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;500;600;700&display=swap');

.montserrat {
    font-family: 'Montserrat', sans-serif;
}
</style>
