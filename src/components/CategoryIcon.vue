<template>
  <span
    :class="['material-symbols-outlined inline-flex items-center justify-center select-none leading-none', className]"
  >
    {{ name }}
  </span>
</template>

<script setup lang="ts">
defineProps({
  name: {
    type: String,
    required: true,
  },
  className: {
    type: String,
    default: "",
  },
});
</script>
