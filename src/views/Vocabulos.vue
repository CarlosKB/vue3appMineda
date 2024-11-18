<template>
  <div>
    <h1>Lista de Vocábulos</h1>

    <!-- Formulário de Cadastro -->
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="termo">Termo:</label>
        <input type="text" id="termo" v-model="newItem.termo" required />
        <span v-if="errors.termo" class="error">{{ errors.termo }}</span>
      </div>
      <div>
        <label for="significado">Significado:</label>
        <input type="text" id="significado" v-model="newItem.significado" required />
        <span v-if="errors.significado" class="error">{{ errors.significado }}</span>
      </div>
      <div>
        <label for="versao">Versão:</label>
        <input type="number" id="versao" v-model="newItem.versao" />
      </div>

      <!-- Campo de data e hora de desativação -->
      <div>
        <label for="dataHoraDesativacao">Data e Hora de Desativação:</label>
        <input type="datetime-local" id="dataHoraDesativacao" v-model="newItem.dataHoraDesativacao" />
      </div>

      <button type="submit">Cadastrar</button>
    </form>

    <!-- Campos para Buscar por Termo e Versão -->
    <div>
      <label for="searchTermo">Buscar Termo:</label>
      <input type="text" id="searchTermo" v-model="searchParams.termo" />
    </div>
    <div>
      <label for="searchVersao">Buscar Versão:</label>
      <input type="number" id="searchVersao" v-model="searchParams.versao" />
    </div>
    <button @click="searchByTermoAndVersao">Buscar</button>

    <!-- Tabela de Vocábulos -->
    <table v-if="items.length" class="vocab-table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Termo</th>
          <th>Significado</th>
          <th>Versão</th>
          <th>Data de Desativação</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in items" :key="item.id">
          <td>{{ item.id }}</td>
          <td>{{ item.termo }}</td>
          <td>{{ item.significado }}</td>
          <td>{{ item.versao }}</td>
          <td>{{ item.dataHoraDesativacao ? new Date(item.dataHoraDesativacao).toLocaleString() : 'Não Definida' }}</td>
          <td>
            <span v-if="item.dataHoraDesativacao === null">Ativo</span>
            <span v-else>Inativo</span>
          </td>
        </tr>
      </tbody>
    </table>
    <p v-else>Carregando itens...</p>
  </div>
</template>

<script>
import axios from "axios";

// Função para buscar dados da API
async function fetchData(context) {
  try {
    const response = await axios.get(
      "https://8080-carloskb-springboot3app-rbdaypshy4b.ws-us116.gitpod.io/vocabulo"
    );
    if (Array.isArray(response.data)) {
      context.items = response.data;
    } else {
      console.error("A resposta da API não é um array:", response.data);
      context.items = [];
    }
  } catch (error) {
    console.error("Erro ao buscar os dados:", error);
  }
}

// Função para buscar dados por termo e versão
async function searchByTermoAndVersao(context) {
  if (context.searchParams.termo && context.searchParams.versao) {
    try {
      const response = await axios.get(
        `https://8080-carloskb-springboot3app-rbdaypshy4b.ws-us116.gitpod.io/vocabulo/${context.searchParams.termo}/${context.searchParams.versao}`
      );
      if (Array.isArray(response.data)) {
        console.log(response.data);
        
        if (response.data.length > 0) {
          context.items = response.data;
        } else {
          alert("Nenhum item encontrado")
        }

      } else {
        console.error("A resposta da API não é um array:", response.data);
      }
    } catch (error) {
      console.error("Erro ao buscar por termo e versão:", error);
    }
  } else {
    alert("Por favor, insira tanto o termo quanto a versão.");
  }
}

// Função para validar e submeter o formulário
function handleSubmit(context) {
  context.errors = { termo: "", significado: "" }; // Limpa os erros anteriores

  // Verifica se os campos obrigatórios estão preenchidos
  if (!context.newItem.termo) {
    context.errors.termo = "O campo 'Termo' é obrigatório.";
  }
  if (!context.newItem.significado) {
    context.errors.significado = "O campo 'Significado' é obrigatório.";
  }

  // Se não houver erros, faz o cadastro
  if (!context.errors.termo && !context.errors.significado) {
    addItem(context);
  }
}

// Função para adicionar um novo item
async function addItem(context) {
  try {
    const response = await axios.post(
      "https://8080-carloskb-springboot3app-rbdaypshy4b.ws-us116.gitpod.io/vocabulo",
      {
        termo: context.newItem.termo,
        significado: context.newItem.significado,
        versao: context.newItem.versao,
        dataHoraDesativacao: context.newItem.dataHoraDesativacao,
      }
    );
    if (response.data) {
      // Atualiza a tabela com o novo item
      fetchData(context);
      clearForm(context); // Limpa os campos do formulário
    }
  } catch (error) {
    console.error("Erro ao cadastrar o vocábulo:", error);
  }
}

// Função para limpar os campos do formulário
function clearForm(context) {
  context.newItem = { termo: "", significado: "", versao: 1, dataHoraDesativacao: "" };
}

export default {
  data() {
    return {
      items: [], // Armazena os vocábulos recebidos da API
      newItem: {
        termo: "",
        significado: "",
        versao: 1,
        dataHoraDesativacao: "", // Novo campo para data e hora de desativação
      }, // Dados do novo vocábulo a ser cadastrado
      errors: {
        termo: "",
        significado: "",
      }, // Erros de validação
      searchParams: {
        termo: "",
        versao: null,
      }, // Dados para buscar vocábulos
    };
  },
  async mounted() {
    // Carregar dados automaticamente ao entrar na página
    await fetchData(this);
  },
  methods: {
    handleSubmit() {
      handleSubmit(this);
    },
    searchByTermoAndVersao() {
      searchByTermoAndVersao(this);
    },
  },
};
</script>

<style scoped>
h1 {
  color: #333;
  margin-bottom: 20px;
}

.vocab-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
}

.vocab-table th,
.vocab-table td {
  border: 1px solid #ccc;
  padding: 10px;
  text-align: left;
}

.vocab-table th {
  background-color: #007bff;
  color: white;
}

.vocab-table tr:nth-child(even) {
  background-color: #3a3a3a;
}

.vocab-table tr:hover {
  background-color: #2c2c2c;
}

.error {
  color: red;
  font-size: 0.875rem;
}
</style>
