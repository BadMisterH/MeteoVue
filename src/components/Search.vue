<script setup>
import { defineProps } from "vue";

const props = defineProps({
  modelValue: String, 
});

const emit = defineEmits(["update:modelValue", "search"]);

const doSomething = (e) => {
    e.preventDefault();
    const searchValue = props.modelValue?.trim();
    if (searchValue) {
      emit('search', searchValue);
    }
}

</script>

<template>
  <form class="flex flex-row gap-3 justify-center w-full my-8 px-4" @submit="doSomething">
    <div class="relative flex-1 max-w-md">
      <input
        class="w-full px-6 py-4 text-lg bg-white/20 backdrop-blur-md border border-white/30 rounded-2xl text-white placeholder-white/70 focus:outline-none focus:ring-2 focus:ring-white/50 focus:border-white/50 transition-all duration-300"
        type="text"
        placeholder="🔍 Rechercher une ville..."
        :value="props.modelValue"
        @input="emit('update:modelValue', $event.target.value)"
      />
    </div>

    <button 
      type="submit"
      class="px-8 py-4 bg-white/20 hover:bg-white/30 border border-white/30 hover:border-white/50 text-white font-semibold rounded-2xl transition-all duration-300 focus:outline-none focus:ring-2 focus:ring-white/50"
    >
      Rechercher
    </button>
  </form>
</template>
