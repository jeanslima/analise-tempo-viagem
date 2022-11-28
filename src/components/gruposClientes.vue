<template>
    <div class="regras montserrat">
        <b-modal
            class="montserrat"
            v-model="modalNovoGrupo"
            :title="flagForm == 'add' ? 'Criar Grupo' : 'Editar Grupo'"
            size="lg"
            hide-footer
            @hidden="cleanModal"
        >
            <b-form @submit="salvar" @reset="onReset" class="mt-2 montserrat">
                <b-card>
                    <b-row>
                        <b-col>
                            <b-form-group class="mt-2" label="Nome do Grupo:" label-for="nomegrupoID">
                                <b-form-input id="nomegrupoID" v-model="form.nome_grupo" type="text"></b-form-input>
                            </b-form-group>

                            <label for="textarea-default">Observação:</label>
                            <b-form-textarea
                                id="textarea-default"
                                placeholder="adicione uma observação..."
                                v-model="form.descricao"
                            ></b-form-textarea>
                            <br />
                            <label for="lista-id">Pesquisar cliente:</label>
                            <!-- <b-form-input
                                v-model="text"
                                list="lista-id"
                                placeholder="NOME (CNPJ) (CIDADE/UF/PAIS)"
                            ></b-form-input> -->
                            <b-form-select id="lista-id" v-model="text" @change="enviarParaGrid">
                                <!-- <option>Manual Option</option> -->
                                <b-form-select-option
                                    v-for="(cliente, index) in clientes"
                                    :key="index"
                                    :value="cliente"
                                >
                                    {{ cliente.nome_cliente }} ({{ cliente.cnpj }}) ({{ cliente.local }})
                                </b-form-select-option>
                            </b-form-select>
                        </b-col>
                    </b-row>
                    <br />
                    <b-row>
                        <b-col>
                            <ag-grid-vue
                                style="width: auto; height: 38vh"
                                class="ag-theme-alpine"
                                :columnDefs="columnDefsClientes"
                                :defaultColDef="defaultColDefClientes"
                                :rowData="form.clientes"
                                :rowDragManaged="true"
                                :animateRows="true"
                                @grid-ready="onGridReadyClientes"
                                :rowSelection="rowSelectionClientes"
                                @selection-changed="onSelectionChangedClientes"
                                :getContextMenuItems="getContextMenuItemsClientes"
                            >
                            </ag-grid-vue>
                        </b-col>
                    </b-row>
                </b-card>
                <b-row class="m-3" align-h="end">
                    <b-button type="reset" variant="secondary">Limpar</b-button>
                    <b-button class="ml-2" type="submit" variant="success">{{ flagFormComputed }}</b-button>
                </b-row>
            </b-form>
        </b-modal>

        <b-card>
            <b-row>
                <b-col sm="12" md="4" align-self="center">
                    <b-input-group size="sm" class="mb-2">
                        <b-input-group-prepend is-text>
                            <b-icon icon="search"></b-icon>
                        </b-input-group-prepend>
                        <b-form-input
                            id="filter-text-box"
                            @input="onFilterTextBoxChanged"
                            v-model="searchInput"
                            class="form-control inputPesquisa"
                            type="search"
                            placeholder="Busque por qualquer termo..."
                        ></b-form-input>
                    </b-input-group>
                </b-col>
                <b-col sm="12" md="2" offset-md="6">
                    <b-button @click="addGrupo" variant="success" style="float: right; margin-bottom: 15px"
                        ><b-icon icon="plus" aria-hidden="true"></b-icon> Grupo</b-button
                    >
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
                        :getContextMenuItems="getContextMenuItems"
                        :masterDetail="true"
                        :detailCellRendererParams="detailCellRendererParams"
                        @first-data-rendered="onFirstDataRendered"
                    >
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

// const slugString = (str) =>
//     str
//         .normalize('NFD')
//         .replace(/(\r\n|\n|\r)/gm, '')
//         .replace(/[\u0300-\u036f]/g, '')
//         .toLowerCase()
//         .trim()
//         .split(' ')
//         .join('_');

export default {
    name: 'GruposClientes',
    components: {
        AgGridVue,
    },
    computed: {
        flagFormComputed() {
            return this.flagForm == 'add' ? 'Criar' : 'Editar';
        },
    },
    data() {
        return {
            modalNovoGrupo: false,
            arrayGrupoSelecionado: [],
            flagForm: 'add',
            text: '',
            clientes: [
                {
                    nome_cliente: 'JBS',
                    cnpj: '123-1231.32.123-213',
                    local: 'Argentina',
                },
                {
                    nome_cliente: 'AURORA',
                    cnpj: '321-512.44.234-345',
                    local: 'Brasil',
                },
                {
                    nome_cliente: 'NESTLE',
                    cnpj: '4524-131.32.31443-13',
                    local: 'Chile',
                },
                {
                    nome_cliente: 'PIRACANJUBA',
                    cnpj: '4245-512.455.341-1344',
                    local: 'Equador',
                },
            ],
            form: {},
            formDefault: {
                nome_grupo: '',
                descricao: '',
                clientes: [],
            },

            // AGGRID TODOS OS GRUPOS
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
                flex: 1,
                // floatingFilter: true,
            },

            // AGGRID TODOS OS CLIENTES
            columnDefsClientes: null,
            rowDataClientes: [],
            gridApiClientes: null,
            searchInputClientes: '',
            rowSelectionClientes: null,
            defaultColDefClientes: {
                initialWidth: 200,
                sortable: true,
                resizable: true,
                menuTabs: ['filterMenuTab'],
                filter: true,
                // floatingFilter: true,
            },
        };
    },
    created() {
        this.rowSelection = 'single';
        this.form = { ...this.formDefault };
        this.getApi();

        // AGGRID VISUALIZAÇÃO DE CLIENTES NO GRUPO
        this.detailCellRendererParams = {
            detailGridOptions: {
                columnDefs: [
                    { field: 'nome_cliente', headerName: 'Nome' },
                    { field: 'cnpj', headerName: 'CNPJ' },
                    { field: 'local', headerName: 'Local' },
                ],
                defaultColDef: {
                    flex: 1,
                },
            },
            getDetailRowData: (params) => {
                console.log(params.data);
                params.successCallback(params.data.clientes);
            },
        };
    },
    methods: {
        // AGGRID TODOS OS GRUPOS
        onSelectionChanged() {
            const selectedRows = this.gridApi.getSelectedRows();
            this.arrayGrupoSelecionado = selectedRows;
            console.log(this.arrayGrupoSelecionado);
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
        getContextMenuItems(params) {
            this.gridApi.deselectAll();
            params.node.setSelected(true);

            var result = [
                {
                    name: 'Editar',
                    action: () => {
                        this.flagForm = 'edit';
                        this.editarRegistro(this.arrayGrupoSelecionado[0]);
                    },
                },
                {
                    name: 'Excluir',
                    action: () => {
                        this.excluirRegistro(params.node.data.objectId);
                    },
                },
            ];
            return result;
        },
        // AGGRID TODOS OS CLIENTES
        onSelectionChangedClientes() {
            const selectedRows = this.gridApiClientes.getSelectedRows();
            this.arrayClienteSelecionado = selectedRows;
            console.log(this.arrayClienteSelecionado);
        },
        onGridReadyClientes(params) {
            this.gridApiClientes = params.api;
            this.gridColumnApiCLientes = params.columnApi;
        },
        onFilterTextBoxChangedClientes() {
            this.gridApiClientes.setQuickFilter(this.searchInput);
        },
        onPrintQuickFilterTextsClientes() {
            this.gridApiCLientes.forEachNode(function (rowNode, index) {
                console.log('Row ' + index + ' quick filter text is ' + rowNode.quickFilterAggregateText);
            });
        },
        getContextMenuItemsClientes(params) {
            this.gridApiClientes.deselectAll();
            params.node.setSelected(true);

            var result = [
                {
                    name: 'Excluir',
                    action: () => {
                        this.form.clientes.splice(params.node.rowIndex, 1);
                    },
                },
            ];
            return result;
        },
        onFirstDataRendered(params) {
            // arbitrarily expand a row for presentational purposes
            setTimeout(function () {
                params.api.getDisplayedRowAtIndex(1).setExpanded(true);
            }, 0);
        },
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

            if (this.form.nome_grupo == '') {
                this.notificacao('ERRO!', 'Adicione um nome para o grupo atual', 'danger');
                return false;
            }
            if (this.form.descricao == '') {
                this.notificacao('ERRO!', 'Adicione uma descrição para o grupo', 'danger');
                return false;
            }
            if (this.form.clientes.length === 0) {
                this.notificacao('ERRO!', 'Adicione pelo menos 1 cliente para o grupo atual', 'danger');
                return false;
            }

            this.flagForm == 'add'
                ? this.salvarApi()
                : this.confirmarEdicaoRegistro(this.arrayGrupoSelecionado[0].objectId);
        },
        onReset(event) {
            event.preventDefault();
            this.form = { ...this.formDefault };
        },
        formatDados(param) {
            if (param == null) {
                return '--';
            } else {
                return param.toUpperCase().replace('_', ' ').replace('_', ' ').replace('_', ' ').replace('_', ' ');
            }
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
                url: 'https://atvcrud.b4a.app/classes/grupos_clientes',
                headers: {
                    'X-Parse-Application-Id': 'mVhLIhh3SEYGOWiPqou0wu6YlBO79okFDVuFOVxB',
                    'X-Parse-Master-Key': 'Li5S5aNM6mbXCY45KOGXf1n4w199m1adF25OKUMD',
                    'Content-Type': 'application/json',
                },
            };

            const response = await axios.request(options);
            const responseData = await response.data.results;
            this.rowData = responseData;
            console.log(responseData);
        },
        async salvarApi() {
            var data = this.form;
            axios
                .post('https://atvcrud.b4a.app/classes/grupos_clientes', data, {
                    headers: {
                        'X-Parse-Application-Id': 'mVhLIhh3SEYGOWiPqou0wu6YlBO79okFDVuFOVxB',
                        'X-Parse-Master-Key': 'Li5S5aNM6mbXCY45KOGXf1n4w199m1adF25OKUMD',
                        'Content-Type': 'application/json',
                    },
                })
                .then((response) => {
                    console.log('data salvo!', response);
                    this.notificacao('Sucesso!', 'Grupo de clientes salva com sucesso!', 'success');
                    this.modalNovoGrupo = false;
                    this.getApi();
                })
                .catch((error) => {
                    console.log(error);
                });
        },
        editarRegistro(params) {
            this.flagForm = 'edit';
            this.form = { ...params };
            this.modalNovoGrupo = true;
        },
        formatarTipo(value) {
            var slug = null;
            switch (value) {
                case 'brasil_risk':
                    slug = 'Brasil Risk';
                    break;
                case 'jbs_transportadora':
                    slug = 'JBS Transportadora';
                    break;
            }
            return slug;
        },
        async confirmarEdicaoRegistro(objectId) {
            delete this.form.createdAt;
            delete this.form.updatedAt;

            const data = { ...this.form };
            const options = {
                method: 'PUT',
                url: `https://atvcrud.b4a.app/classes/grupos_clientes/${objectId}`,
                headers: {
                    'X-Parse-Application-Id': 'mVhLIhh3SEYGOWiPqou0wu6YlBO79okFDVuFOVxB',
                    'X-Parse-Master-Key': 'Li5S5aNM6mbXCY45KOGXf1n4w199m1adF25OKUMD',
                    'Content-Type': 'application/json',
                },
                data,
            };

            const response = await axios.request(options);
            console.log('data editado!', response);

            this.notificacao('Sucesso!', 'Grupo de clientes editado com sucesso!', 'success');
            this.modalNovoGrupo = false;
            this.getApi();
        },
        async excluirRegistro(objectId) {
            this.$bvModal
                .msgBoxConfirm(`Deseja excluir esse grupo?`, {
                    title: 'Confirmação',
                    size: 'md',
                    okVariant: 'success',
                    okTitle: 'Sim',
                    cancelTitle: 'Não',
                    footerClass: 'p-2',
                    hideHeaderClose: false,
                    centered: true,
                })
                .then((value) => {
                    value ? this.excluir(objectId) : console.log(value);
                })
                .catch((err) => {
                    console.log(err);
                });
        },
        async excluir(objectId) {
            const options = {
                method: 'DELETE',
                url: `https://atvcrud.b4a.app/classes/grupos_clientes/${objectId}`,
                headers: {
                    'X-Parse-Application-Id': 'mVhLIhh3SEYGOWiPqou0wu6YlBO79okFDVuFOVxB',
                    'X-Parse-Master-Key': 'Li5S5aNM6mbXCY45KOGXf1n4w199m1adF25OKUMD',
                    'Content-Type': 'application/json',
                },
            };

            await axios
                .request(options)
                .then(function (response) {
                    this.notificacao('Sucesso!', 'Grupo excluido com sucesso!', 'success');
                    this.modalNovoGrupo = false;
                    console.log('data excluido!', response);
                })
                .catch(function (error) {
                    console.error(error);
                });
            this.getApi();
        },
        enviarParaGrid() {
            this.form.clientes.push(this.text);
            console.log(this.text);
        },
    },
    watch: {},
    beforeMount() {
        // AGGRID TODOS OS GRUPOS
        this.columnDefs = [
            { field: 'nome_grupo', headerName: 'Nome do Grupo', width: 300 },
            { field: 'descricao', headerName: 'Descrição', width: 500 },
            {
                field: 'clientes',
                headerName: 'Clientes',
                valueFormatter: (params) => {
                    const plural = params.value.length > 1 ? ' CLIENTES' : ' CLIENTE';
                    return params.value.length + plural;
                },
                width: 700,
                cellRenderer: 'agGroupCellRenderer',
            },
        ];
        // AGGRID TODOS OS CLIENTES
        this.columnDefsClientes = [
            { field: 'nome_cliente', headerName: 'Nome' },
            { field: 'cnpj', headerName: 'CNPJ' },
            { field: 'local', headerName: 'Local' },
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
