<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

// Recebendo o perfil do App.vue
const props = defineProps({
  perfil: {
    type: String,
    default: "cliente",
  },
});

// Definindo o evento de clique para abrir o chamado
const emit = defineEmits(["abrir-chamado"]);

const tickets = ref([]);
const loading = ref(true);
const erro = ref("");

const fetchTickets = async () => {
  loading.value = true;
  erro.value = "";

  try {
    const response = await axios.get("http://localhost/api/tickets");
    tickets.value = response.data.data || response.data;
  } catch (err) {
    erro.value =
      "Falha ao carregar os chamados. Verifique se a API está rodando.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  fetchTickets();
});
</script>

<template>
  <div
    class="bg-white dark:bg-[#16171d] border border-[#e5e4e7] dark:border-[#2e303a] rounded-xl shadow-sm overflow-hidden"
  >
    <div
      class="p-6 border-b border-[#e5e4e7] dark:border-[#2e303a] flex justify-between items-center bg-gray-50/50 dark:bg-transparent"
    >
      <div>
        <h2 class="text-lg font-semibold text-gray-900 dark:text-gray-100">
          {{
            props.perfil === "agente"
              ? "Fila de Atendimento"
              : "Chamados Recentes"
          }}
        </h2>
        <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">
          Acompanhe o status e as respostas dos chamados abertos.
        </p>
      </div>

      <button
        @click="fetchTickets"
        :disabled="loading"
        class="px-4 py-2 bg-white dark:bg-[#1f2028] border border-gray-200 dark:border-gray-700 rounded-lg text-sm font-medium text-gray-700 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-800 transition-colors flex items-center gap-2 disabled:opacity-50"
      >
        <svg
          :class="{ 'animate-spin': loading }"
          class="w-4 h-4"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"
          ></path>
        </svg>
        Atualizar
      </button>
    </div>

    <div
      v-if="erro"
      class="p-6 bg-rose-50 border-b border-rose-100 text-rose-600 text-sm"
    >
      {{ erro }}
    </div>

    <div class="overflow-x-auto">
      <table class="w-full text-left border-collapse">
        <thead>
          <tr
            class="bg-gray-50 dark:bg-[#1f2028] text-xs font-mono uppercase tracking-wider text-gray-500 dark:text-gray-400 border-b border-[#e5e4e7] dark:border-[#2e303a]"
          >
            <th class="px-6 py-4 font-medium">ID</th>
            <th class="px-6 py-4 font-medium">Título & Descrição</th>
            <th class="px-6 py-4 font-medium">Status</th>
            <!-- Coluna exclusiva do Agente -->
            <th
              v-if="props.perfil === 'agente'"
              class="px-6 py-4 font-medium w-1/3"
            >
              Rascunho da IA
            </th>
          </tr>
        </thead>
        <tbody class="divide-y divide-[#e5e4e7] dark:divide-[#2e303a]">
          <tr v-if="loading && tickets.length === 0">
            <td
              :colspan="props.perfil === 'agente' ? 4 : 3"
              class="px-6 py-12 text-center text-gray-500 text-sm"
            >
              Buscando chamados no banco de dados...
            </td>
          </tr>

          <tr v-else-if="tickets.length === 0">
            <td
              :colspan="props.perfil === 'agente' ? 4 : 3"
              class="px-6 py-12 text-center text-gray-500 text-sm"
            >
              Nenhum chamado encontrado.
            </td>
          </tr>

          <!-- Linha clicável emitindo o ID -->
          <tr
            v-for="ticket in tickets"
            :key="ticket.id"
            @click="emit('abrir-chamado', ticket.id)"
            class="hover:bg-gray-50 dark:hover:bg-[#1f2028]/50 transition-colors cursor-pointer"
          >
            <td class="px-6 py-4 text-sm font-mono text-gray-500">
              #{{ ticket.id }}
            </td>

            <td class="px-6 py-4">
              <div
                class="text-sm font-medium text-gray-900 dark:text-gray-100 mb-1"
              >
                {{ ticket.titulo }}
              </div>
              <div
                class="text-xs text-gray-500 dark:text-gray-400 line-clamp-2"
                :title="ticket.descricao_cliente"
              >
                {{ ticket.descricao_cliente }}
              </div>
            </td>

            <td class="px-6 py-4">
              <span
                :class="[
                  'inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium border',
                  ticket.status === 'aberto'
                    ? 'bg-amber-100 text-amber-800 border-amber-200 dark:bg-amber-500/10 dark:text-amber-400 dark:border-amber-500/20'
                    : ticket.status === 'resolvido'
                      ? 'bg-emerald-100 text-emerald-800 border-emerald-200 dark:bg-emerald-500/10 dark:text-emerald-400 dark:border-emerald-500/20'
                      : 'bg-gray-100 text-gray-800 border-gray-200 dark:bg-gray-800 dark:text-gray-300 dark:border-gray-700',
                ]"
              >
                {{
                  ticket.status ? ticket.status.toUpperCase() : "DESCONHECIDO"
                }}
              </span>
            </td>

            <!-- Célula exclusiva do Agente -->
            <td v-if="props.perfil === 'agente'" class="px-6 py-4">
              <div
                class="text-xs text-gray-600 dark:text-gray-400 bg-gray-50 dark:bg-[#1a1b22] p-3 rounded border border-gray-100 dark:border-gray-800 min-h-[60px] flex items-center"
              >
                <span
                  v-if="ticket.rascunho_ia"
                  class="line-clamp-3"
                  :title="ticket.rascunho_ia"
                >
                  {{ ticket.rascunho_ia }}
                </span>
                <span v-else class="text-gray-400 italic">
                  Aguardando IA...
                </span>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
