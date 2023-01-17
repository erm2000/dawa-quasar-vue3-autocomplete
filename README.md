# dawa-quasar-vue3-autocomplete
Component for a Dawa autocomplete using Vue 3 Typescript/Composition API

Usage:
```
<script setup lang="ts">
import { ref } from 'vue'
import { IDawaSearchObject } from './models'
import DawaAddress from 'src/components/DawaAddressComponent.vue'
  
const dawaAddressInput = ref<IDawaSearchObject>( { isSelectedAddress: false, searchaddress: '' } )

</script>

<template>
    <q-card v-bind="$attrs">
        <q-form class="q-px-sm">
            <DawaAddress v-model="dawaAddressInput" color="secondary"/>
        </q-form>
    </q-card>
</template>
```
