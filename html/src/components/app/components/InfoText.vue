<template>
  <div class="infotext">
    <h3>{{ props.title }}</h3>
    <h4 v-if="props.text">{{ props.text }}</h4>
  </div>
</template>

<script setup lang="ts">

const props = defineProps<{
  title: string,
  text?: string
}>();
</script>

<style scoped l-ang="scss">
.infotext {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
