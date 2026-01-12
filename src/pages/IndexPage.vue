<template>
  <q-page padding class="flex flex-center bg-grey-2">
    <q-card style="width: 100%; max-width: 500px" class="q-pa-md shadow-3">
      <div class="text-h5 q-mb-lg text-center text-primary text-bold">Sorteador Interativo</div>
      <q-card-section>
        <!-- Lista de Campos de Nomes -->
        <div class="q-gutter-y-md q-mb-md">
          <div
            v-for="(item, index) in listaNomes"
            :key="index"
            class="row q-col-gutter-sm items-center"
          >
            <div class="col">
              <q-input
                v-model="item.nome"
                filled
                :label="`Nome ${index + 1}`"
                dense
                @keyup.enter="adicionarCampo"
              />
            </div>
            <div class="col-auto">
              <q-btn
                round
                flat
                color="negative"
                icon="delete"
                size="sm"
                @click="removerCampo(index)"
                :disable="listaNomes.length <= 1"
              />
            </div>
          </div>
        </div>

        <!-- Botão para Adicionar mais campos -->
        <q-btn
          outline
          color="primary"
          icon="add"
          label="Adicionar Nome"
          class="full-width q-mb-md"
          @click="adicionarCampo"
        />
      </q-card-section>

      <q-card-section>
        <div class="row q-col-gutter-md">
          <div class="col-6">
            <q-btn
              color="primary"
              class="full-width"
              label="Sortear Um"
              :loading="carregando"
              @click="executarAcao('unico')"
            />
          </div>
          <div class="col-6">
            <q-btn
              color="secondary"
              class="full-width"
              label="Formar Duplas"
              :loading="carregando"
              @click="executarAcao('duplas')"
            />
          </div>
        </div>
      </q-card-section>

      <q-separator q-my-md v-if="carregando || resultadoUnico || duplas.length > 0" />

      <!-- Área de Feedback / Skeleton -->
      <q-card-section v-if="carregando">
        <div class="text-center text-grey-7 q-mb-sm italic">Sorteando...</div>
        <q-skeleton type="rect" height="100px" animation="pulse" class="rounded-borders" />
        <div class="q-mt-md">
          <q-skeleton v-for="n in 3" :key="n" type="text" class="q-mt-xs" />
        </div>
      </q-card-section>

      <!-- Resultados Reais (Aparecem após o loading) -->
      <q-card-section v-else-if="resultadoUnico || duplas.length > 0">
        <div class="text-h6 q-mb-sm text-center">🎉 Resultado 🎉</div>

        <!-- Resultado Único -->
        <transition appear enter-active-class="animated bounceIn">
          <div
            v-if="resultadoUnico"
            class="text-center q-pa-xl bg-primary text-white rounded-borders shadow-2"
          >
            <div class="text-caption">O escolhido foi:</div>
            <div class="text-h3 text-bold text-uppercase">{{ resultadoUnico }}</div>
          </div>
        </transition>

        <!-- Resultado Duplas -->
        <transition appear enter-active-class="animated fadeIn">
          <q-list v-if="duplas.length > 0" bordered separator class="rounded-borders">
            <q-item v-for="(dupla, index) in duplas" :key="index" class="q-py-md">
              <q-item-section avatar>
                <q-avatar color="secondary" text-color="white">{{ index + 1 }}</q-avatar>
              </q-item-section>
              <q-item-section>
                <q-item-label class="text-h6">
                  {{ dupla[0] }} <span class="text-grey-6">&</span> {{ dupla[1] || '---' }}
                </q-item-label>
                <q-item-label caption v-if="!dupla[1]" class="text-orange-9 text-bold">
                  Ficou sem par
                </q-item-label>
              </q-item-section>
            </q-item>
          </q-list>
        </transition>

        <q-btn
          flat
          color="grey-7"
          label="Limpar Tudo"
          class="q-mt-lg full-width"
          @click="limparResultados"
        />
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script setup>
import { ref } from 'vue'
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Estados
const listaNomes = ref([{ nome: '' }, { nome: '' }])
const carregando = ref(false)
const resultadoUnico = ref('')
const duplas = ref([])

// Gerenciamento de Campos
const adicionarCampo = () => {
  listaNomes.value.push({ nome: '' })
}

const removerCampo = (index) => {
  listaNomes.value.splice(index, 1)
}

// Lógica de Sorteio com Delay (Skeleton)
const executarAcao = (tipo) => {
  // 1. Filtrar nomes válidos (não vazios)
  const nomesValidos = listaNomes.value
    .map((item) => item.nome.trim())
    .filter((nome) => nome !== '')

  // 2. Validações básicas
  if (nomesValidos.length < (tipo === 'duplas' ? 2 : 1)) {
    $q.notify({
      message: tipo === 'duplas' ? 'Insira pelo menos 2 nomes.' : 'Insira pelo menos 1 nome.',
      color: 'negative',
      icon: 'warning',
    })
    return
  }

  // 3. Iniciar processo de "Loading"
  limparResultados()
  carregando.value = true

  // 4. Aguardar 5 segundos (Skeleton visível)
  setTimeout(() => {
    carregando.value = false

    if (tipo === 'unico') {
      const rand = Math.floor(Math.random() * nomesValidos.length)
      resultadoUnico.value = nomesValidos[rand]
    } else {
      const embaralhados = embaralhar([...nomesValidos])
      const resultadoTemp = []
      for (let i = 0; i < embaralhados.length; i += 2) {
        resultadoTemp.push([embaralhados[i], embaralhados[i + 1] ? embaralhados[i + 1] : null])
      }
      duplas.value = resultadoTemp
    }
  }, 5000) // 5000ms = 5 segundos
}

// Algoritmo de embaralhamento
const embaralhar = (array) => {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[array[i], array[j]] = [array[j], array[i]]
  }
  return array
}

const limparResultados = () => {
  resultadoUnico.value = ''
  duplas.value = []
}
</script>

<style scoped>
.animated {
  animation-duration: 0.5s;
}
</style>
