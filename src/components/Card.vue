<template>
  <div class="rounded overflow-hidden shadow-lg bg-white py-4 px-6 mb-4">
    <div v-if="title" class="font-bold text-xl mb-2">{{ title }}</div>
    <slot class="text-gray-700 text-base"></slot>
  </div>
</template>

<script>
  export default {
    name: "Card",

    props: ['title']
  }
</script>

<style scoped>

</style>