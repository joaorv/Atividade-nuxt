<script setup lang="ts">
import { reactive, ref, computed } from 'vue'

// Define os metadados da página
useHead({
  title: 'Cadastro de Membro | Nuxt App',
  meta: [{ name: 'description', content: 'Formulário de cadastro com Nuxt 3 e Composition API' }]
})

// Estado reativo do formulário
const form = reactive({
  nome: '',
  email: '',
  curso: '',
  semestre: '',
  habilidades: [] as string[],
  bio: ''
})

const LIMITE_BIO = 200
const enviadoComSucesso = ref(false)
const dadosEnviados = ref<typeof form | null>(null)
const erros = reactive({
  nome: '',
  email: '',
  curso: '',
  semestre: '',
  habilidades: ''
})

// Contador de caracteres reativo
const caracteresRestantes = computed(() => LIMITE_BIO - form.bio.length)

// Opções de cursos e áreas
const opcoesCursos = [
  'Engenharia de Software',
  'Ciência da Computação',
  'Sistemas de Informação',
  'Análise e Desenv. de Sistemas',
  'Design Digital / UI/UX'
]

// Opções de habilidades/interesses
const opcoesHabilidades = [
  'Front-end',
  'Back-end',
  'Mobile',
  'DevOps / Cloud',
  'UI/UX Design',
  'Inteligência Artificial'
]

// Validações
const validarEmail = (email: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)

const validarFormulario = (): boolean => {
  let valido = true

  erros.nome = form.nome.trim() ? '' : 'O nome completo é obrigatório.'
  erros.email = !form.email.trim() 
    ? 'O e-mail é obrigatório.' 
    : !validarEmail(form.email) 
      ? 'Insira um e-mail válido.' 
      : ''
  erros.curso = form.curso ? '' : 'Selecione uma área de atuação.'
  erros.semestre = form.semestre ? '' : 'Informe o período/semestre.'
  erros.habilidades = form.habilidades.length > 0 ? '' : 'Selecione ao menos uma área de interesse.'

  if (erros.nome || erros.email || erros.curso || erros.semestre || erros.habilidades) {
    valido = false
  }

  return valido
}

const resetarFormulario = () => {
  form.nome = ''
  form.email = ''
  form.curso = ''
  form.semestre = ''
  form.habilidades = []
  form.bio = ''
}

const submeterFormulario = () => {
  if (!validarFormulario()) {
    enviadoComSucesso.value = false
    return
  }

  // Log formatado no console para conferência técnica
  console.group('%c🚀 Novo Membro Cadastrado', 'color: #00dc82; font-weight: bold; font-size: 14px;')
  console.table({
    Nome: form.nome,
    Email: form.email,
    Curso: form.curso,
    Semestre: `${form.semestre}º`,
    Habilidades: form.habilidades.join(', '),
    Bio: form.bio || '(Não preenchida)'
  })
  console.groupEnd()

  // Guardar cópia imutável para exibir no feedback
  dadosEnviados.value = JSON.parse(JSON.stringify(form))
  enviadoComSucesso.value = true

  // Reset reativo dos campos
  resetarFormulario()
}
</script>

<template>
  <main class="cadastro-container">
    <header class="cadastro-header">
      <NuxtLink to="/" class="link-voltar">← Voltar para a Home</NuxtLink>
      <h1>Cadastro de Integrante</h1>
      <p>Preencha os dados abaixo para ingressar no ecossistema.</p>
    </header>

    <!-- Feedback visual de Sucesso -->
    <section v-if="enviadoComSucesso && dadosEnviados" class="card-sucesso" role="alert">
      <div class="icone-sucesso">✓</div>
      <div>
        <h3>Cadastro realizado com sucesso!</h3>
        <p>Bem-vindo(a), <strong>{{ dadosEnviados.nome }}</strong> ({{ dadosEnviados.email }}).</p>
        <p class="detalhes-submissao">
          <strong>Trilha:</strong> {{ dadosEnviados.curso }} ({{ dadosEnviados.semestre }}º semestre) |
          <strong>Interesses:</strong> {{ dadosEnviados.habilidades.join(', ') }}
        </p>
      </div>
    </section>

    <!-- Formulário Principal -->
    <form @submit.prevent="submeterFormulario" novalidate class="form-card">
      <!-- Nome -->
      <div class="campo-grupo">
        <label for="nome">Nome Completo *</label>
        <input 
          id="nome"
          v-model="form.nome"
          type="text"
          placeholder="Ex: Alan Turing"
          :class="{ 'campo-erro': erros.nome }"
        />
        <span v-if="erros.nome" class="msg-erro">{{ erros.nome }}</span>
      </div>

      <!-- Email -->
      <div class="campo-grupo">
        <label for="email">E-mail *</label>
        <input 
          id="email"
          v-model="form.email"
          type="email"
          placeholder="exemplo@dominio.com"
          :class="{ 'campo-erro': erros.email }"
        />
        <span v-if="erros.email" class="msg-erro">{{ erros.email }}</span>
      </div>

      <!-- Grid com Curso e Semestre -->
      <div class="grid-duplo">
        <div class="campo-grupo">
          <label for="curso">Curso / Área de Atuação *</label>
          <select id="curso" v-model="form.curso" :class="{ 'campo-erro': erros.curso }">
            <option disabled value="">Selecione uma opção</option>
            <option v-for="curso in opcoesCursos" :key="curso" :value="curso">
              {{ curso }}
            </option>
          </select>
          <span v-if="erros.curso" class="msg-erro">{{ erros.curso }}</span>
        </div>

        <div class="campo-grupo">
          <label for="semestre">Semestre / Período *</label>
          <select id="semestre" v-model="form.semestre" :class="{ 'campo-erro': erros.semestre }">
            <option disabled value="">Selecione</option>
            <option v-for="num in 10" :key="num" :value="num">
              {{ num }}º Semestre
            </option>
          </select>
          <span v-if="erros.semestre" class="msg-erro">{{ erros.semestre }}</span>
        </div>
      </div>

      <!-- Habilidades (Checkboxes) -->
      <div class="campo-grupo">
        <label class="label-titulo">Interesses & Habilidades *</label>
        <div class="habilidades-grid">
          <label 
            v-for="hab in opcoesHabilidades" 
            :key="hab" 
            class="checkbox-tag"
            :class="{ 'tag-selecionada': form.habilidades.includes(hab) }"
          >
            <input 
              v-model="form.habilidades" 
              type="checkbox" 
              :value="hab"
            />
            <span>{{ hab }}</span>
          </label>
        </div>
        <span v-if="erros.habilidades" class="msg-erro">{{ erros.habilidades }}</span>
      </div>

      <!-- Bio / Mensagem -->
      <div class="campo-grupo">
        <div class="label-com-contador">
          <label for="bio">Biografia Curta / Mensagem</label>
          <span class="contador" :class="{ 'limite-proximo': caracteresRestantes < 20 }">
            {{ caracteresRestantes }} caracteres
          </span>
        </div>
        <textarea 
          id="bio"
          v-model="form.bio"
          :maxlength="LIMITE_BIO"
          rows="3"
          placeholder="Compartilhe seus objetivos e experiências prévias..."
        ></textarea>
      </div>

      <!-- Ações -->
      <div class="form-acoes">
        <button type="submit" class="btn-submit">
          Cadastrar Integrante
        </button>
      </div>
    </form>
  </main>
</template>

<style scoped>
.cadastro-container {
  max-width: 680px;
  margin: 2rem auto;
  padding: 0 1.5rem;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  color: #0f172a;
  /* Força todos os elementos nativos (incluindo o dropdown de opções) a usar o tema claro */
  color-scheme: light;
}

.cadastro-header {
  margin-bottom: 2rem;
}

.link-voltar {
  display: inline-block;
  margin-bottom: 1rem;
  color: #00dc82;
  text-decoration: none;
  font-weight: 600;
  transition: opacity 0.2s;
}

.link-voltar:hover {
  text-decoration: underline;
}

.cadastro-header h1 {
  font-size: 2rem;
  font-weight: 800;
  margin: 0;
  color: #0f172a;
}

.cadastro-header p {
  color: #475569;
  margin-top: 0.5rem;
}

.card-sucesso {
  background-color: #f0fdf4;
  border: 1px solid #86efac;
  color: #166534;
  padding: 1rem 1.25rem;
  border-radius: 8px;
  margin-bottom: 1.5rem;
  display: flex;
  align-items: flex-start;
  gap: 1rem;
}

.icone-sucesso {
  background-color: #22c55e;
  color: #ffffff;
  width: 24px;
  height: 24px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  flex-shrink: 0;
}

.card-sucesso h3 {
  margin: 0 0 0.25rem 0;
  font-size: 1rem;
  color: #14532d;
}

.card-sucesso p {
  margin: 0.25rem 0;
  font-size: 0.9rem;
  color: #166534;
}

.detalhes-submissao {
  font-size: 0.825rem;
  color: #15803d;
}

.form-card {
  background-color: #ffffff;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  padding: 2rem;
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.campo-grupo {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
}

.campo-grupo label {
  font-weight: 600;
  font-size: 0.9rem;
  color: #1e293b;
}

.label-com-contador {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.contador {
  font-size: 0.75rem;
  color: #64748b;
}

.limite-proximo {
  color: #ef4444;
  font-weight: bold;
}

/* Campos de entrada com cores claras e texto nítido */
input[type="text"],
input[type="email"],
select,
textarea {
  width: 100%;
  padding: 0.75rem 0.85rem;
  background-color: #ffffff;
  color: #0f172a;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  font-size: 0.95rem;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
  box-sizing: border-box;
}

/* Garante fundo branco e texto legível na lista suspensa do select */
select option {
  background-color: #ffffff;
  color: #0f172a;
}

input::placeholder,
textarea::placeholder {
  color: #94a3b8;
}

input:focus,
select:focus,
textarea:focus {
  border-color: #00dc82;
  box-shadow: 0 0 0 3px rgba(0, 220, 130, 0.2);
}

.campo-erro {
  border-color: #ef4444 !important;
}

.msg-erro {
  font-size: 0.8rem;
  color: #ef4444;
  margin-top: 0.2rem;
}

.grid-duplo {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 1rem;
}

@media (max-width: 540px) {
  .grid-duplo {
    grid-template-columns: 1fr;
  }
}

.habilidades-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-top: 0.25rem;
}

.checkbox-tag {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.4rem 0.8rem;
  border-radius: 20px;
  border: 1px solid #cbd5e1;
  font-size: 0.85rem;
  cursor: pointer;
  user-select: none;
  transition: all 0.15s ease-in-out;
  background-color: #f8fafc;
  color: #1e293b;
}

.checkbox-tag input {
  cursor: pointer;
}

.tag-selecionada {
  border-color: #00dc82;
  background-color: #f0fdf4;
  color: #166534;
  font-weight: 600;
}

.btn-submit {
  width: 100%;
  background-color: #00dc82;
  color: #0f172a;
  padding: 0.85rem;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: background-color 0.2s, transform 0.1s;
}

.btn-submit:hover {
  background-color: #05b86d;
}

.btn-submit:active {
  transform: scale(0.99);
}
</style>