<script setup>
import { ref } from "vue";
import TicketForm from "./components/TicketForm.vue";
import TicketList from "./components/TicketList.vue";

// Estado global simples para controlar a tela e simular a sessão
const telaAtual = ref("lista");
const menuAberto = ref(true);
const usuarioLogado = ref("cliente@revenda.com.br");
const ticketSelecionado = ref(null);

// NOVO: Controle de Perfil (Mock)
const perfilAtual = ref("cliente"); // 'cliente' ou 'agente'

const alternarPerfil = () => {
  perfilAtual.value = perfilAtual.value === "cliente" ? "agente" : "cliente";
  // Opcional: Se quiser mudar o email fake dependendo do perfil
  usuarioLogado.value =
    perfilAtual.value === "cliente"
      ? "cliente@revenda.com.br"
      : "suporte@revenda.com.br";
};

// Funções de navegação
const navegarPara = (tela, ticket = null) => {
  telaAtual.value = tela;
  ticketSelecionado.value = ticket;
};
</script>

<template>
  <div
    class="flex h-screen bg-gray-100 dark:bg-[#08060d] font-sans text-gray-900 dark:text-gray-100"
  >
    <!-- Sidebar -->
    <aside
      :class="[
        menuAberto ? 'w-64' : 'w-20',
        'bg-gray-900 text-white transition-all duration-300 flex flex-col shadow-xl z-20',
      ]"
    >
      <div
        class="h-16 flex items-center justify-center border-b border-gray-800"
      >
        <span v-if="menuAberto" class="font-bold text-lg tracking-wider"
          >HELPDESK <span class="text-purple-500">AI</span></span
        >
        <span v-else class="font-bold text-xl text-purple-500">H</span>
      </div>

      <nav class="flex-1 p-4 space-y-2">
        <button
          @click="navegarPara('lista')"
          :class="[
            telaAtual === 'lista'
              ? 'bg-purple-600 text-white'
              : 'text-gray-400 hover:bg-gray-800 hover:text-white',
            'w-full flex items-center gap-3 px-3 py-2.5 rounded-lg transition-colors',
          ]"
        >
          <svg
            class="w-5 h-5 flex-shrink-0"
            fill="none"
            stroke="currentColor"
            viewBox="0 0 24 24"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M4 6h16M4 10h16M4 14h16M4 18h16"
            ></path>
          </svg>
          <span v-if="menuAberto" class="font-medium">
            {{
              perfilAtual === "cliente"
                ? "Meus Chamados"
                : "Fila de Atendimento"
            }}
          </span>
        </button>
      </nav>

      <!-- Perfil fake na base do menu -->
      <div
        class="p-4 border-t border-gray-800 flex items-center gap-3 overflow-hidden"
      >
        <div
          :class="[
            perfilAtual === 'cliente' ? 'bg-purple-500' : 'bg-emerald-500',
            'w-8 h-8 rounded-full flex items-center justify-center font-bold flex-shrink-0 text-white transition-colors',
          ]"
        >
          {{ perfilAtual === "cliente" ? "C" : "A" }}
        </div>
        <div v-if="menuAberto" class="flex flex-col">
          <span class="text-sm font-medium leading-tight">
            {{ perfilAtual === "cliente" ? "Cliente Teste" : "Agente Nível 1" }}
          </span>
          <span class="text-xs text-gray-400 leading-tight truncate">{{
            usuarioLogado
          }}</span>
        </div>
      </div>
    </aside>

    <main class="flex-1 flex flex-col overflow-hidden">
      <!-- Topbar -->
      <header
        class="h-16 bg-white dark:bg-[#16171d] border-b border-[#e5e4e7] dark:border-[#2e303a] shadow-sm flex items-center justify-between px-6 z-10 transition-colors"
      >
        <div class="flex items-center gap-4">
          <button
            @click="menuAberto = !menuAberto"
            class="text-gray-500 hover:text-gray-700 dark:hover:text-gray-300 focus:outline-none"
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
                d="M4 6h16M4 12h16M4 18h7"
              ></path>
            </svg>
          </button>
          <h1 class="text-lg font-semibold text-gray-800 dark:text-gray-100">
            {{
              telaAtual === "lista"
                ? "Painel de Chamados"
                : ticketSelecionado
                  ? "Detalhes do Chamado"
                  : "Novo Chamado"
            }}
          </h1>
        </div>

        <div class="flex items-center gap-6">
          <button
            v-if="telaAtual === 'lista' && perfilAtual === 'cliente'"
            @click="navegarPara('formulario')"
            class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-colors shadow-sm flex items-center gap-2"
          >
            <span>+</span> Novo Chamado
          </button>
        </div>
      </header>

      <div class="flex-1 overflow-auto p-6 bg-gray-50 dark:bg-[#08060d]">
        <!-- Renderiza a Lista de Tickets -->
        <div v-if="telaAtual === 'lista'">
          <!-- Passamos o perfil atual e um evento para quando clicarem em uma linha da tabela -->
          <TicketList
            :perfil="perfilAtual"
            @abrir-chamado="(id) => navegarPara('formulario', id)"
          />
        </div>

        <!-- Renderiza o Formulário (Agora serve para Criar ou Visualizar/Editar) -->
        <div v-else-if="telaAtual === 'formulario'">
          <TicketForm
            :ticket-id="ticketSelecionado"
            :usuario="usuarioLogado"
            :perfil="perfilAtual"
            @voltar="navegarPara('lista')"
          />
        </div>
      </div>
    </main>
  </div>

  <div
    class="px-6 py-4 bg-[#f4f3ec] dark:bg-[#1f2028] border-t border-[#e5e4e7] dark:border-[#2e303a] flex justify-between items-center z-20 relative"
  >
    <div class="flex items-center gap-2">
      <span
        :class="[
          perfilAtual === 'cliente' ? 'bg-purple-500' : 'bg-emerald-500',
          'w-3 h-3 rounded-full inline-block transition-colors',
        ]"
      ></span>
      <span
        class="text-xs font-mono font-medium text-[#08060d] dark:text-[#f3f4f6] uppercase tracking-wider"
      >
        Helpdesk Core • Canal do
        {{ perfilAtual === "cliente" ? "Cliente" : "Suporte" }}
      </span>
    </div>
    <!-- SELETOR DE PERFIL (MOCK) -->
    <div
      class="flex items-center gap-2 bg-gray-100 dark:bg-[#1f2028] p-1 rounded-lg border border-gray-200 dark:border-[#2e303a]"
    >
      <button
        @click="perfilAtual = 'cliente'"
        :class="[
          perfilAtual === 'cliente'
            ? 'bg-white dark:bg-[#2e303a] shadow-sm'
            : 'text-gray-500 hover:text-gray-700 dark:hover:text-gray-300',
          'px-3 py-1.5 text-xs font-medium rounded-md transition-all',
        ]"
      >
        Visão Cliente
      </button>
      <button
        @click="perfilAtual = 'agente'"
        :class="[
          perfilAtual === 'agente'
            ? 'bg-white dark:bg-[#2e303a] shadow-sm'
            : 'text-gray-500 hover:text-gray-700 dark:hover:text-gray-300',
          'px-3 py-1.5 text-xs font-medium rounded-md transition-all',
        ]"
      >
        Visão Agente
      </button>
    </div>
    <span class="text-xs font-mono text-[#6b6375] dark:text-[#9ca3af]"
      >v1.0-poc</span
    >
  </div>
</template>
