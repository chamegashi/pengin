<template>
  <div>
    <div v-if="colorDatum === 'red'" :id="id" class="bg-red-600 py-6 px-5"></div>
    <div v-if="colorDatum === 'green'" :id="id" class="bg-green-600 py-6 px-5"></div>
    <div v-if="colorDatum === 'blue'" :id="id" class="bg-blue-600 py-6 px-5"></div>
    <div v-if="colorDatum === 'purple'" :id="id" class="bg-purple-600 py-6 px-5"></div>
    <div v-if="colorDatum === 'yellow'" :id="id" class="bg-yellow-400 py-6 px-5"></div>
    <div v-if="colorDatum === 'black'" :id="id" class="bg-black py-6 px-5"></div>
    <div v-if="colorDatum === ''" :id="id" class="border border-gray-900 border-dashed py-6 px-5"></div>
  </div>
</template>

<script lang="ts">

type Color = 
  'red' |
  'green' |
  'blue' |
  'purple' |
  'yellow' |
  'black' |
  ''

interface Props {
  color: Color,
  id: number
}

import { defineComponent, ref, watch } from '@vue/composition-api';

export default defineComponent({
  props: {
    color: {
      type: String,
      required: true,
    },
    id: {
      type: Number,
      required: true,
    }
  },
  setup(props: Props){
    let colorDatum = ref<Color>(props.color)

    watch(props, () => {
      colorDatum.value = props.color
    })

return {
      colorDatum
    };
  },
});
</script>
