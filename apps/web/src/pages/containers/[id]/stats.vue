<template lang="pug">
q-card-section.q-pa-none
  q-card-section
    q-custom-page-containers-stats
  q-expansion-item.bg-blue-grey-10.full-width(v-if='debug' label='Debug' icon='mdi-bug' dark dense)
    q-card.overflow-auto.bg-blue-grey-8.text-white(:style='{maxHeight: "250px", height: "250px"}')
      q-card-section.q-pa-xs
        pre.q-ma-none(v-html='JSON.stringify(props.container, null, 2)')
</template>

<script lang="ts" setup>
const { debug } = useDebug()
const props = defineProps<{ container: any }>()
</script>
