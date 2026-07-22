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
const resposta_final = ref("");
const respondido_em = ref(null);
const status = ref("aberto");

const loading = ref(false); // carregando dados / salvando
const regerando = ref(false); // chamando a IA pra gerar um novo rascunho
const sucesso = ref(""); // mensagem de sucesso (varia conforme a ação)
const erro = ref("");

// Computed para saber se estamos editando ou criando
const isEditing = computed(() => !!props.ticketId);

// Guarda o rascunho como veio do servidor, pra saber se o agente já mexeu nele
// antes de decidir se pergunta ou não antes de regerar por cima.
const rascunhoOriginal = ref("");
const rascunhoFoiEditado = computed(
  () => rascunho_ia.value !== rascunhoOriginal.value,
);

// Busca os dados se for uma visualização/edição
const carregarTicket = async () => {
  loading.value = true;
  erro.value = "";
  try {
    const response = await axios.get(
      `http://localhost/api/tickets/${props.ticketId}`,
    );
    const ticket = response.data.data || response.data;
    titulo.value = ticket.titulo;
    descricao_cliente.value = ticket.descricao_cliente;
    rascunho_ia.value = ticket.rascunho_ia || "";
    rascunhoOriginal.value = ticket.rascunho_ia || "";
    resposta_final.value = ticket.resposta_final || "";
    respondido_em.value = ticket.respondido_em || null;
    status.value = ticket.status || "aberto";
  } catch (err) {
    erro.value = "Erro ao carregar os detalhes do chamado.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  if (isEditing.value) {
    carregarTicket();
  }
});

// Pede pra IA (re)gerar o rascunho a partir da descrição do cliente.
// Endpoint assumido: POST /api/tickets/:id/regerar-ia -> { rascunho_ia }
const regerarComIA = async () => {
  if (
    rascunhoFoiEditado.value &&
    !window.confirm("Isso vai substituir o texto atual do rascunho. Continuar?")
  ) {
    return;
  }

  regerando.value = true;
  erro.value = "";
  sucesso.value = "";
  try {
    const response = await axios.post(
      `http://localhost/api/tickets/${props.ticketId}/re-gerar-ia`,
    );
    const novoRascunho =
      response.data?.rascunho_ia ?? response.data?.data?.rascunho_ia ?? "";
    rascunho_ia.value = novoRascunho;
    rascunhoOriginal.value = novoRascunho;
  } catch (err) {
    erro.value = "Não foi possível gerar uma nova sugestão da IA agora.";
    console.error(err);
  } finally {
    regerando.value = false;
  }
};

// Salva o rascunho editado, sem publicar nada pro cliente ainda.
const salvarRascunho = async () => {
  loading.value = true;
  sucesso.value = "";
  erro.value = "";
  try {
    await axios.put(`http://localhost/api/tickets/${props.ticketId}`, {
      rascunho_ia: rascunho_ia.value,
    });
    rascunhoOriginal.value = rascunho_ia.value;
    sucesso.value = "Rascunho salvo. O cliente ainda não vê essa resposta.";
  } catch (err) {
    erro.value = "Falha na comunicação com o servidor. Verifique a API.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Publica a resposta final: é isso que aparece pro cliente.
const enviarResposta = async () => {
  loading.value = true;
  sucesso.value = "";
  erro.value = "";
  try {
    await axios.put(`http://localhost/api/tickets/${props.ticketId}`, {
      rascunho_ia: rascunho_ia.value,
      resposta_final: rascunho_ia.value,
      status: "resolvido",
    });
    rascunhoOriginal.value = rascunho_ia.value;
    resposta_final.value = rascunho_ia.value;
    status.value = "resolvido";
    respondido_em.value = new Date().toISOString();
    sucesso.value = "Resposta enviada! Já aparece no chamado do cliente.";
  } catch (err) {
    erro.value = "Falha na comunicação com o servidor. Verifique a API.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// Cliente abrindo um novo chamado
const criarChamado = async () => {
  loading.value = true;
  sucesso.value = "";
  erro.value = "";
  try {
    await axios.post("http://localhost/api/tickets", {
      titulo: titulo.value,
      descricao_cliente: descricao_cliente.value,
      status: "aberto",
    });
    titulo.value = "";
    descricao_cliente.value = "";
    sucesso.value = "Chamado registrado com sucesso!";
  } catch (err) {
    erro.value = "Falha na comunicação com o servidor. Verifique a API.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// O <form> só cobre o caminho "padrão" de cada perfil (Enter aciona isso).
// Publicar a resposta pro cliente é sempre um clique explícito à parte.
const handleSubmit = () => {
  if (!isEditing.value) {
    criarChamado();
  } else if (props.perfil === "agente") {
    salvarRascunho();
  }
};
</script>

<template>
  <div
    class="max-w-2xl mx-auto my-10 bg-white dark:bg-[#16171d] border border-[#e5e4e7] dark:border-[#2e303a] rounded-xl shadow-lg overflow-hidden transition-colors duration-300 relative"
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
              <div class="flex items-center justify-between mb-2">
                <label
                  class="text-xs font-mono uppercase tracking-wider text-red-600 dark:text-red-400 flex items-center gap-2"
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

                <button
                  type="button"
                  @click="regerarComIA"
                  :disabled="regerando || loading"
                  class="text-xs font-medium text-red-600 dark:text-red-400 hover:text-red-700 dark:hover:text-red-300 border border-red-200 dark:border-red-900/40 rounded-md px-2.5 py-1 flex items-center gap-1.5 disabled:opacity-50 transition-colors"
                >
                  <svg
                    :class="{ 'animate-spin': regerando }"
                    class="w-3.5 h-3.5"
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
                  {{ regerando ? "Gerando..." : "Regerar com IA" }}
                </button>
              </div>

              <textarea
                v-model="rascunho_ia"
                rows="5"
                placeholder="A IA ainda não gerou uma resposta para este chamado..."
                class="w-full px-4 py-3 rounded-lg border border-red-200 dark:border-red-900/50 bg-red-50/30 dark:bg-red-900/10 text-[#08060d] dark:text-[#f3f4f6] focus:outline-none focus:border-red-500 transition-colors text-sm resize-none"
              ></textarea>

              <p
                v-if="rascunhoFoiEditado"
                class="text-xs text-[#6b6375] dark:text-[#9ca3af] mt-1.5"
              >
                Rascunho ainda não salvo.
              </p>
            </div>

            <div class="flex items-center gap-2">
              <span
                class="text-xs font-mono uppercase tracking-wider text-[#6b6375] dark:text-[#9ca3af]"
                >Status:</span
              >
              <span
                :class="[
                  'inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-medium border',
                  status === 'resolvido'
                    ? 'bg-emerald-100 text-emerald-800 border-emerald-200 dark:bg-emerald-500/10 dark:text-emerald-400 dark:border-emerald-500/20'
                    : 'bg-amber-100 text-amber-800 border-amber-200 dark:bg-amber-500/10 dark:text-amber-400 dark:border-amber-500/20',
                ]"
              >
                {{ status === "resolvido" ? "RESOLVIDO" : "ABERTO" }}
              </span>
              <span class="text-xs text-[#6b6375] dark:text-[#9ca3af]"
                >— muda sozinho quando você envia a resposta ao cliente.</span
              >
            </div>
          </div>
        </template>

        <!-- ÁREA EXCLUSIVA DO CLIENTE: Resposta do Suporte -->
        <div
          v-if="isEditing && props.perfil === 'cliente'"
          class="border-t border-dashed border-[#e5e4e7] dark:border-[#2e303a] pt-5 mt-5"
        >
          <label
            class="block text-xs font-mono uppercase tracking-wider text-[#6b6375] dark:text-[#9ca3af] mb-2"
            >Resposta do Suporte</label
          >

          <div
            v-if="status === 'resolvido' && resposta_final"
            class="p-4 rounded-lg border border-red-200 dark:border-red-900/40 bg-red-50/40 dark:bg-red-900/10"
          >
            <p
              class="text-sm text-[#08060d] dark:text-[#f3f4f6] whitespace-pre-line"
            >
              {{ resposta_final }}
            </p>
            <p
              v-if="respondido_em"
              class="text-xs text-[#6b6375] dark:text-[#9ca3af] mt-3"
            >
              Respondido em
              {{ new Date(respondido_em).toLocaleString("pt-BR") }}
            </p>
          </div>

          <div
            v-else
            class="p-4 rounded-lg border border-dashed border-[#e5e4e7] dark:border-[#2e303a] text-sm text-[#6b6375] dark:text-[#9ca3af] italic"
          >
            Nosso time ainda está analisando seu chamado. Assim que houver uma
            resposta, ela aparece aqui.
          </div>
        </div>

        <!-- Botões de Ação -->
        <div
          class="pt-4 flex flex-col gap-3"
          v-if="!isEditing || (isEditing && props.perfil === 'agente')"
        >
          <button
            v-if="isEditing && props.perfil === 'agente'"
            type="button"
            @click="enviarResposta"
            :disabled="loading || !rascunho_ia"
            class="w-full py-3.5 px-4 bg-red-600 hover:bg-red-700 text-white font-medium text-sm rounded-lg transition-all duration-200 shadow-md cursor-pointer disabled:opacity-50 flex items-center justify-center gap-2"
          >
            {{
              status === "resolvido"
                ? "Reenviar Resposta ao Cliente"
                : "Enviar Resposta ao Cliente"
            }}
          </button>

          <button
            type="submit"
            :disabled="loading"
            :class="[
              isEditing
                ? 'bg-white dark:bg-[#1f2028] border border-gray-200 dark:border-gray-700 text-gray-700 dark:text-gray-300 hover:bg-gray-50 dark:hover:bg-gray-800'
                : 'bg-red-600 hover:bg-red-700 text-white shadow-md',
              'w-full py-3.5 px-4 font-medium text-sm rounded-lg transition-all duration-200 cursor-pointer disabled:opacity-50 flex items-center justify-center gap-2',
            ]"
          >
            <span v-if="loading">Processando...</span>
            <span v-else>{{
              isEditing ? "Salvar Rascunho" : "Abrir Chamado"
            }}</span>
          </button>
        </div>
      </form>

      <!-- Feedbacks -->
      <div
        v-if="sucesso"
        class="mt-6 p-4 bg-emerald-500/10 border border-emerald-500/30 text-emerald-600 dark:text-emerald-400 rounded-lg text-sm flex items-center gap-2"
      >
        <span>✓ {{ sucesso }}</span>
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
