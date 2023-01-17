# dawa-quasar-vue3-autocomplete
Component for a Dawa autocomplete using Vue 3 Typescript/Composition API

..hmm. Something like this. (I hope)
It not really a downloadable project; but more like a copy/paste project

I hope you get my idea with this Quasar <q-select> implementation

Usage:
```
<script setup lang="ts">
import { ref } from 'vue'
import { IDawaSearchObject } from './models'
import DawaAddress from 'src/components/DawaAddressComponent.vue'
  
const dawaAddressInput = ref<IDawaSearchObject>( { isSelectedAddress: false, searchaddress: '' } )

watch(
  () => dawaAddressInput,
  async (newValue, oldValue) => {
    if (newValue.dawaInfo.isSelectedAddress) {
      console.log('Watch - dawaAddress changed')
    }
  }
)
</script>

<template>
  <q-form class="q-px-sm">
      <DawaAddress v-model="dawaAddressInput" color="secondary"/>
  </q-form>
</template>
```
