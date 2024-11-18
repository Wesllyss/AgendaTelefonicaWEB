<template>
  <div class="container">
    <h2>Agenda Telefônica</h2>
    <div class="form-container">
      <InputText v-model="novoContato.nome" placeholder="Nome" class="input-field" />
      <InputMask v-model="novoContato.telefone" mask="(99) 99999-9999" placeholder="Telefone" class="input-field" />
      <InputText v-model="novoContato.endereco" placeholder="Endereço" class="input-field" />
      <Button
        label="Salvar Contato"
        icon="pi pi-save"
        @click="saveContato"
        class="button"
      />
      <Button
        v-if="editando"
        label="Cancelar"
        icon="pi pi-times"
        @click="cancelarEdicao"
        class="button cancel-button"
      />
    </div>

    <div class="search-container">
      <InputText v-model="BuscarContato" placeholder="Buscar Contato" class="input-field" />
    </div>

    <div class="Contato-list">
      <div v-for="(contato, index) in FiltrarContato" :key="contato.contatoId" class="Contato-item">
        <div>
          <strong>{{ contato.nome }}</strong><br />
          {{ contato.telefone }}<br />
          {{ contato.endereco }}
        </div>
        <div>
          <Button
            label="Editar"
            icon="pi pi-pencil"
            @click="editarContato(contato)"
            class="edit-button"
          />
          <Button
            label="Remover"
            icon="pi pi-times"
            @click="removeContato(contato.contatoId)"
            class="remove-button"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from 'vue';
import axios from 'axios';
import InputMask from 'primevue/inputmask';
import InputText from 'primevue/inputtext';
import Button from 'primevue/button';

export default {
  components: {
    InputMask,
    InputText,
    Button,
  },
  setup() {
    const Contato = ref([]);
    const novoContato = ref({ contatoId: null, nome: '', telefone: '', endereco: '' });
    const BuscarContato = ref('');
    const editando = ref(false);

    
    const getContatos = async () => {
      try {
        const response = await axios.get('https://localhost:5001/api/Contato');
        Contato.value = response.data;
      } catch (error) {
        console.error("Erro ao buscar contatos:", error);
      }
    };

    
    const saveContato = async () => {
      if (novoContato.value.nome && novoContato.value.telefone && novoContato.value.endereco) {
        try {
          if (editando.value) {
            
            await axios.put(`https://localhost:5001/api/Contato/${novoContato.value.contatoId}`, novoContato.value);
            const index = Contato.value.findIndex(c => c.contatoId === novoContato.value.contatoId);
            Contato.value[index] = { ...novoContato.value };
            editando.value = false;
          } else {
            
            const response = await axios.post('https://localhost:5001/api/Contato', novoContato.value);
            Contato.value.push(response.data);
          }
          novoContato.value = { contatoId: null, nome: '', telefone: '', endereco: '' };
        } catch (error) {
          console.error("Erro ao salvar contato:", error);
        }
      }
    };

    
    const editarContato = (contato) => {
      novoContato.value = { ...contato };  
      editando.value = true;
    };

    
    const cancelarEdicao = () => {
      novoContato.value = { contatoId: null, nome: '', telefone: '', endereco: '' };
      editando.value = false;
    };

    
    const removeContato = async (id) => {
      try {
        await axios.delete(`https://localhost:5001/api/Contato/${id}`);
        Contato.value = Contato.value.filter(contato => contato.contatoId !== id);  // Remove o contato da lista local
      } catch (error) {
        console.error("Erro ao remover contato:", error);
      }
    };

    const FiltrarContato = computed(() => {
      return Contato.value.filter(contato =>
        contato.nome.toLowerCase().includes(BuscarContato.value.toLowerCase()) ||
        contato.telefone.includes(BuscarContato.value)
      );
    });

    
    getContatos();

    return {
      Contato,
      novoContato,
      BuscarContato,
      editando,
      saveContato,
      editarContato,
      cancelarEdicao,
      removeContato,
      FiltrarContato,
    };
  }
};
</script>

<style scoped>
.container {
  width: 500px;
  padding: 30px;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.form-container {
  display: flex;
  flex-direction: column;
  gap: 15px;
  width: 100%;
  align-items: center;
}

.input-field {
  width: 100%;
  font-size: 1.1rem;
}

.button {
  margin-top: 15px;
  font-size: 1.1rem;
}

.cancel-button {
  background-color: gray;
  color: white;
}

.search-container {
  margin-top: 20px;
}

.Contato-list {
  margin-top: 20px;
  width: 100%;
}

.Contato-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: 15px 0;
  width: 100%;
}

.edit-button {
  background-color: blue;
  color: white;
  margin-right: 10px;
}

.remove-button {
  background-color: red;
  color: white;
}
</style>