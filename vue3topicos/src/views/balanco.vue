<template>
    <div>
        <h1>Balanços</h1>
        <form @submit.prevent="submitBalanco">
            <div>
                <label for="descricao">Descrição:</label>
                <input type="text" v-model="newBalanco.descricao" required>
            </div>
            <div>
                <label for="dataHora">Data/Hora:</label>
                <input type="datetime-local" v-model="newBalanco.dataHora" required>
            </div>
            <div>
                <label for="valor">Valor:</label>
                <input type="number" step="0.01" v-model="newBalanco.valor" required>
            </div>
            <button type="submit">Cadastrar</button>
        </form>

        <h2>Pesquisar Balanços</h2>
        <form @submit.prevent="searchBalancos">
            <div>
                <label for="searchDescricao">Descrição:</label>
                <input type="text" v-model="searchCriteria.descricao">
            </div>
            <div>
                <label for="searchValor">Valor:</label>
                <input type="number" step="0.01" v-model="searchCriteria.valor">
            </div>
            <button type="submit">Buscar</button>
        </form>

        <h2>Lista de Balanços</h2>
        <ul v-if="balancos.length">
            <li v-for="balanco in balancos" :key="balanco.id">
                ID: {{ balanco.id }}, Descrição: {{ balanco.descricao }}, Data/Hora: {{ formatDataHora(balanco.dataHora) }}, Valor: {{ formatValor(balanco.valor) }}
            </li>
        </ul>
        <p v-else>Nenhum registro foi encontrado para os critérios fornecidos.</p>
    </div>
</template>

<script>
import axios from 'axios';
import moment from 'moment';

export default {
    data() {
        return {
            newBalanco: {
                descricao: '',
                dataHora: '',
                valor: 0
            },
            searchCriteria: {
                descricao: '',
                valor: null
            },
            balancos: []
        };
    },
    methods: {
        fetchBalancos() {
            axios.get('/balanco')
                .then(response => {
                    this.balancos = response.data;
                })
                .catch(error => {
                    console.error(error);
                });
        },
        submitBalanco() {
            const payload = {
                ...this.newBalanco,
                dataHora: moment(this.newBalanco.dataHora).toISOString()
            };

            axios.post('/balanco', payload)
                .then(() => {
                    this.fetchBalancos();
                    this.newBalanco = {
                        descricao: '',
                        dataHora: '',
                        valor: 0
                    };
                })
                .catch(error => {
                    console.error(error);
                });
        },
        searchBalancos() {
            const params = new URLSearchParams();
            if (this.searchCriteria.descricao) params.append('descricao', this.searchCriteria.descricao);
            if (this.searchCriteria.valor !== null) params.append('valor', this.searchCriteria.valor);

            axios.get(`/balanco/${this.searchCriteria.descricao}/${this.searchCriteria.valor}`)
                .then(response => {
                    this.balancos = response.data;
                })
                .catch(error => {
                    console.error(error);
                });
        },
        formatDataHora(dataHora) {
            return moment(dataHora).format('DD/MM/YYYY HH:mm');
        },
        formatValor(valor) {
            return valor < 0 ? `${valor} D` : `${valor} C`;
        }
    },
    mounted() {
        this.fetchBalancos();
    }
};
</script>

<style scoped>
form {
    margin-bottom: 1em;
}
label {
    display: block;
    margin-bottom: 0.5em;
}
input {
    margin-bottom: 1em;
}
button {
    display: block;
    margin-top: 1em;
}
</style>
