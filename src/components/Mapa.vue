<script setup>
import { ref, watch } from 'vue'
import MapaSvg from './MapaSvg.vue'
import EstadosJson from '../data/estados.json'

const estado = ref(null)
const nome_estado = ref(null)
const info = ref()
const cidades = ref()
const list_cidades = ref()
const search = ref()

watch(estado, (uf) => {
  let filter = EstadosJson.filter((r) => r.sigla === uf)
  if (filter.length > 0) {
    nome_estado.value = filter[0].uf
    findNoticias(filter[0].cod)
  } else {
    nome_estado.value = uf
  }
})

async function findNoticias(code) {
  cidades.value = null
  search.value = null
  let url = 'https://servicodados.ibge.gov.br/api/v1/localidades/estados/code/municipios'.replace(
    'code',
    code
  )
  info.value = 'Buscando cidades do estado selecionado...'
  const response = await fetch(url)
  cidades.value = await response.json()
  list_cidades.value = cidades.value
  info.value = cidades.value.length + ' Cidades'
}

watch(search, (text) => {
  if (text === null) {
    return false
  } else if (text.length === 0) {
    list_cidades.value = cidades.value
  } else {
    list_cidades.value = cidades.value.filter((r) =>
      r.nome
        .toLowerCase()
        .normalize('NFD')
        .replace(/[\u0300-\u036f]/g, '')
        .includes(
          text
            .toLowerCase()
            .normalize('NFD')
            .replace(/[\u0300-\u036f]/g, '')
        )
    )
  }
})
</script>

<template>
  <div class="sm:grid sm:grid-cols-2 sm:gap-8">
    <div>
      <h1 class="text-slate-500 text-2xl mb-4">Mapa do Brasil e Cidades por Estado</h1>
      <MapaSvg v-model="estado" />
    </div>

    <div>
      <span v-if="estado" class="text-slate-500">
        Estado Selecionado: <b>{{ nome_estado }}</b>
      </span>
      <div v-if="info" class="mt-5">
        <p class="text-white">{{ info }}</p>
        <input
          type="search"
          placeholder="Pesquisar cidade"
          class="w-full py-2 px-2 rounded"
          v-model="search"
        />
        <ul
          v-if="cidades"
          class="border border-gray-200 text-sm shadow-md mt-2 rounded max-h-[80vh] overflow-auto"
        >
          <li
            v-for="cidade in list_cidades"
            :key="cidade.id"
            class="px-4 py-1 bg-white hover:bg-sky-100 hover:text-sky-900 transition-all duration-300 ease-in-out"
          >
            {{ cidade.nome }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>