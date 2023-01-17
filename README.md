# dawa-quasar-vue3-autocomplete

Dawa has an API to get all valid addresses in Demnark.

So this component makes it possible to "autocomplete" a input field; into a valid address in Denmark, by making requests to Dawa's API while typing

Component for a Dawa autocomplete are using Vue 3 Typescript/Composition API with Quasar <q-select>

It's not really a downloadable project; but more like a copy/paste project

I hope someone might find this useful

Usage:
```
<script setup lang="ts">
import { ref, watch } from 'vue'
import { IDawaSearchObject } from 'src/components/models'
import DawaAddress from 'src/components/DawaAddressComponent.vue'
  
const dawaAddressInput = ref<IDawaSearchObject>( { isSelectedAddress: false, searchaddress: '' } )

watch(
  () => dawaAddressInput,
  async (newValue, oldValue) => {
    if (newValue.isSelectedAddress) {
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
