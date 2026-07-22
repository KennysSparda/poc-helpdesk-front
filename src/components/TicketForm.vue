<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";

// Recebendo props do App.vue
const props = defineProps({
  ticketId: { type: [Number, String], default: null },
  perfil: { type: String, default: "cliente" },
  usuario: { type: String, default: "" },
});

const emit = defineEmits(["voltar"]);

const titulo = ref("");
const descricao_cliente = ref("");
const rascunho_ia = ref("");
const status = ref("aberto");

const loading = ref(false);
const sucesso = ref(false);
const erro = ref("");

// Computed para saber se estamos editando ou criando
const isEditing = computed(() => !!props.ticketId);

// Busca os dados se for uma visualização/edição
onMounted(async () => {
  if (isEditing.value) {
    loading.value = true;
    try {
      const response = await axios.get(
        `http://localhost/api/tickets/${props.ticketId}`,
      );
      const ticket = response.data.data || response.data;
      titulo.value = ticket.titulo;
      descricao_cliente.value = ticket.descricao_cliente;
      rascunho_ia.value = ticket.rascunho_ia || "";
      status.value = ticket.status || "aberto";
    } catch (err) {
      erro.value = "Erro ao carregar os detalhes do chamado.";
      console.error(err);
    } finally {
      loading.value = false;
    }
  }
});

const handleSubmit = async () => {
  loading.value = true;
  sucesso.value = false;
  erro.value = "";

  try {
    if (isEditing.value && props.perfil === "agente") {
      // Agente atualizando o chamado
      await axios.put(`http://localhost/api/tickets/${props.ticketId}`, {
        titulo: titulo.value, // Mantém o original
        descricao_cliente: descricao_cliente.value, // Mantém o original
        rascunho_ia: rascunho_ia.value,
        status: status.value,
      });
      sucesso.value = true;
    } else if (!isEditing.value) {
      // Cliente abrindo novo chamado
      await axios.post("http://localhost/api/tickets", {
        titulo: titulo.value,
        descricao_cliente: descricao_cliente.value,
        status: "aberto",
      });
      titulo.value = "";
      descricao_cliente.value = "";
      sucesso.value = true;
    }
  } catch (err) {
    erro.value = "Falha na comunicação com o servidor. Verifique a API.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};
</script>

<template>
  <div
    class="max-w-1xl mx-auto dark:bg-[#16171d] shadow-lg overflow-hidden transition-colors duration-300 relative"
  >
    <!-- Botão Voltar -->
    <button
      @click="emit('voltar')"
      class="absolute top-8 right-8 text-gray-400 hover:text-gray-600 dark:hover:text-gray-200 transition-colors"
    >
      <svg
        class="w-6 h-6"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="M6 18L18 6M6 6l12 12"
        ></path>
      </svg>
    </button>

    <div class="p-8">
      <div class="mb-6 text-left pr-8">
        <h2
          class="text-2xl font-medium text-[#08060d] dark:text-[#f3f4f6] tracking-tight mb-1"
        >
          {{ isEditing ? `Chamado #${props.ticketId}` : "Abrir Novo Chamado" }}
        </h2>
        <p class="text-sm text-[#6b6375] dark:text-[#9ca3af]">
          {{
            isEditing
              ? "Detalhes e interações do ticket."
              : "Relate inconsistências de integração, cadastros ou dúvidas."
          }}
        </p>
      </div>

      <form @submit.prevent="handleSubmit" class="space-y-5 text-left">
        <!-- Título -->
        <div>
          <label
            class="block text-xs font-mono uppercase tracking-wider text-[#6b6375] dark:text-[#9ca3af] mb-2"
            >Assunto / Título</label
          >
          <input
            v-model="titulo"
            type="text"
            :disabled="isEditing"
            required
            placeholder="Ex: Preciso de ajuda com o módulo..."
            :class="[
              'w-full px-4 py-3 rounded-lg border border-[#e5e4e7] dark:border-[#2e303a] bg-transparent text-[#08060d] dark:text-[#f3f4f6] focus:outline-none focus:border-red-500 transition-colors text-sm',
              isEditing
                ? 'opacity-70 bg-gray-50 dark:bg-[#1a1b22] cursor-not-allowed'
                : '',
            ]"
          />
        </div>

        <!-- Descrição -->
        <div>
          <label
            class="block text-xs font-mono uppercase tracking-wider text-[#6b6375] dark:text-[#9ca3af] mb-2"
            >Descrição do Ocorrido</label
          >
          <textarea
            v-model="descricao_cliente"
            rows="4"
            :disabled="isEditing"
            required
            placeholder="Descreva os passos para reproduzir o erro..."
            :class="[
              'w-full px-4 py-3 rounded-lg border border-[#e5e4e7] dark:border-[#2e303a] bg-transparent text-[#08060d] dark:text-[#f3f4f6] focus:outline-none focus:border-red-500 transition-colors text-sm resize-none',
              isEditing
                ? 'opacity-70 bg-gray-50 dark:bg-[#1a1b22] cursor-not-allowed'
                : '',
            ]"
          ></textarea>
        </div>

        <!-- ÁREA EXCLUSIVA DO AGENTE: Rascunho IA e Status -->
        <template v-if="isEditing && props.perfil === 'agente'">
          <div
            class="border-t border-dashed border-[#e5e4e7] dark:border-[#2e303a] pt-5 mt-5"
          >
            <div class="mb-5">
              <label
                class="block text-xs font-mono uppercase tracking-wider text-red-600 dark:text-red-400 mb-2 flex items-center gap-2"
              >
                <svg
                  class="w-4 h-4"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M13 10V3L4 14h7v7l9-11h-7z"
                  ></path>
                </svg>
                Rascunho da IA (Editável)
              </label>
              <textarea
                v-model="rascunho_ia"
                rows="5"
                placeholder="A IA ainda não gerou uma resposta para este chamado..."
                class="w-full px-4 py-3 rounded-lg border border-red-200 dark:border-red-900/50 bg-red-50/30 dark:bg-red-900/10 text-[#08060d] dark:text-[#f3f4f6] focus:outline-none focus:border-red-500 transition-colors text-sm resize-none"
              ></textarea>
            </div>

            <div>
              <label
                class="block text-xs font-mono uppercase tracking-wider text-[#6b6375] dark:text-[#9ca3af] mb-2"
                >Status do Chamado</label
              >
              <select
                v-model="status"
                class="w-full px-4 py-3 rounded-lg border border-[#e5e4e7] dark:border-[#2e303a] bg-white dark:bg-[#1f2028] text-[#08060d] dark:text-[#f3f4f6] focus:outline-none focus:border-red-500 transition-colors text-sm"
              >
                <option value="aberto">Aberto</option>
                <option value="resolvido">Resolvido</option>
              </select>
            </div>
          </div>
        </template>

        <!-- Botão de Ação -->
        <div
          class="pt-4"
          v-if="!isEditing || (isEditing && props.perfil === 'agente')"
        >
          <button
            type="submit"
            :disabled="loading"
            class="w-full py-3.5 px-4 bg-red-600 hover:bg-red-700 text-white font-medium text-sm rounded-lg transition-all duration-200 shadow-md cursor-pointer disabled:opacity-50 flex items-center justify-center gap-2"
          >
            <span v-if="loading">Processando...</span>
            <span v-else>{{
              isEditing ? "Salvar Alterações" : "Abrir Chamado"
            }}</span>
          </button>
        </div>
      </form>

      <!-- Feedbacks -->
      <div
        v-if="sucesso"
        class="mt-6 p-4 bg-emerald-500/10 border border-emerald-500/30 text-emerald-600 dark:text-emerald-400 rounded-lg text-sm flex items-center gap-2"
      >
        <span
          >✓ {{ isEditing ? "Chamado atualizado" : "Chamado registrado" }} com
          sucesso!</span
        >
      </div>

      <div
        v-if="erro"
        class="mt-6 p-4 bg-rose-500/10 border border-rose-500/30 text-rose-600 dark:text-rose-400 rounded-lg text-sm"
      >
        {{ erro }}
      </div>
    </div>
  </div>
</template>
