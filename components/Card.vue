<script setup>
defineProps({
  title: {
    type: String,
    required: true,
  },
  description: {
    type: String,
    required: true,
  },
});
</script>

<template>
  <div
    class="flex gap-2 bg-white backdrop-blur-md outline-gray-200 outline outline-1 bg-opacity-70 px-4 py-3 rounded-xl shadow-xl font-bold text-gray-700"
  >
    <div class="opacity-50 text-gray-900 pt-0.5">
      <slot />
    </div>
    <div>
      <div class="text-xl text-black">{{ title }}</div>
      <div class="text-lg">{{ description }}</div>
    </div>
  </div>
</template>
