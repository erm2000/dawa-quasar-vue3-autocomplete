<script setup lang="ts">
import { QSelect } from 'quasar'
import { ref, nextTick, PropType, computed } from 'vue'
import { IDawaSearchObject } from './models'
import axios from 'axios'

const props = defineProps({
  modelValue: {
    type: Object as PropType<IDawaSearchObject>,
    required: true
  }
})

const emit = defineEmits(
  ['update:modelValue']
)

const dawaModel = computed({
  get () {
    return props.modelValue
  },
  set (value) {
    return emit('update:modelValue', value)
  }
})

const selectRef = ref<QSelect>()
const refInputField = ref()
const dawaOptions = ref([] as IDawaSearchObject[])

const doSearchDawa = async (val: any, debug: boolean) => {
  // Make Dawa API request and return
  const options = [] as IDawaSearchObject[]
  try {
    const caretPos = refInputField.value?.selectionStart ?? 18
    const getData = `https://api.dataforsyningen.dk/autocomplete?q=${val}&caretpos=${caretPos}&fuzzy=`
    const result = await axios.get(getData)
    const dawaData = await result.data

    dawaData.forEach((x: any) => {
      if (debug) { console.log(x) }
      const newId = x.type === 'vejnavn' || x.type === 'adgangsadresse' ? x.tekst : x.forslagstekst
      // Let push new option to the option list
      options.push({ isSelectedAddress: false, searchaddress: newId, dawaInfo: x })
    })
  } catch (ex) {
    console.log(ex)
  }
  return options
}

const filterFn = async (val : any, update : any, abort : any) => {
  // Only make Dawa API request when input contains at least 3 chars
  if (val.length < 3) {
    abort()
    return
  }
  update(async () => {
    if (val) {
      dawaOptions.value = await doSearchDawa(val, false)
      nextTick(() => {
        // Select first item in dropdown option list
        setStartTimeToMiddle()
      })
    }
  })
}

const onFocusHandler = (event: any) => {
  // I need a ref to the actual <input> element inside <q-select>
  // .. to set caret position in the input field
  if (!refInputField.value) { refInputField.value = event.target }
}

const onBlueHandler = async (/* event: FocusEvent */) => {
  dawaOptions.value = await doSearchDawa(dawaModel.value.searchaddress, true)
  const matchingAddress = dawaOptions.value?.filter((x) => x.dawaInfo.forslagstekst === dawaModel.value.dawaInfo.forslagstekst && x.dawaInfo.type === 'adresse')

  if (matchingAddress.length === 0 && dawaModel.value.searchaddress !== '') {
    // Address not matching
    selectRef.value?.updateInputValue(selectRef.value.modelValue.searchaddress, true)
    selectRef.value?.showPopup()
    nextTick(() => {
      refInputField.value?.setSelectionRange(dawaModel.value.dawaInfo.caretpos, dawaModel.value.dawaInfo.caretpos)
    })
  } else if (matchingAddress.length === 1) {
    // Matching address found onBlur (when leaving the q-select field)
    selectRef.value?.updateInputValue(matchingAddress[0].dawaInfo.forslagstekst, true)
    dawaModel.value.isSelectedAddress = true
  }
}

const newSelection = async (item: IDawaSearchObject) => {
  const addressList = ref(await doSearchDawa(item.dawaInfo.forslagstekst, true))
  const matchingAddress = addressList.value.filter((x) => x.dawaInfo.forslagstekst === item.dawaInfo.forslagstekst && x.dawaInfo.type === 'adresse')
  if (matchingAddress.length === 1) {
    // Matching address selected in the option dropdown list
    dawaModel.value = matchingAddress[0]
    dawaModel.value.isSelectedAddress = true
  } else {
    // Present new values in dropdown as this is still not a valid address
    dawaOptions.value = addressList.value
    selectRef.value?.updateInputValue(dawaModel.value.searchaddress ?? '', true)
    selectRef.value?.showPopup()
    nextTick(() => {
      refInputField.value?.setSelectionRange(dawaModel.value.dawaInfo.caretpos, dawaModel.value.dawaInfo.caretpos)
    })
  }
}

const setStartTimeToMiddle = () => {
  if (dawaOptions.value?.length > 0) {
    selectRef.value?.setOptionIndex(-1)
    selectRef.value?.moveOptionSelection(1, true)
  }
}
</script>

<template>
  <q-select
    v-bind="$attrs"
    v-model="dawaModel"
    ref="selectRef"
    label="Adresse"
    use-input
    input-debounce="0"
    hide-selected
    fill-input
    option-label="searchaddress"
    :options="dawaOptions"
    @filter="filterFn"
    hide-dropdown-icon
    transition-show="jump-down"
    transition-hide="jump-up"
    autocomplete="off"
    autocorrect="off"
    id="myDataInput"
    @blur="onBlueHandler"
    @focus="onFocusHandler"
    @update:model-value="newSelection"
  >
    <template #option="scope">
        <q-item v-bind="scope.itemProps" dense class="q-px-xs">
            <q-item-section avatar dense  class="q-px-none" style="min-width:35px">
                <q-icon :class="{ 'text-secondary': scope.selected }" :name="scope.opt.dawaInfo.type == 'adresse' ? 'home' : 'fa-solid fa-city'" dense></q-icon>
            </q-item-section>
            <q-item-section>
                <div :class="{ 'text-bold text-secondary': scope.selected }">{{scope.opt.searchaddress.replace(', ,',',')}}</div>
            </q-item-section>
        </q-item>
    </template>

    <template #append>
        <div>
            <q-icon v-if="dawaModel.dawaInfo && dawaModel.dawaInfo.type === 'adresse'" name="check_circle" color="secondary" class="q-mr-sm"></q-icon>
            <q-icon v-else name="fa-solid fa-circle-exclamation" color="red" class="q-mr-sm"></q-icon>
        </div>
    </template>
  </q-select>
</template>
