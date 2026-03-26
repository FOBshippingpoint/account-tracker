<template>
  <Teleport to="body">
    <div v-if="modelValue" class="fixed inset-0 z-50 flex flex-col justify-end" @click.self="close">
      <div 
        class="absolute inset-0 bg-black/40 backdrop-blur-[2px] animate-fade-in" 
        @click="close"
      ></div>
      
      <div 
        class="animate-slide-up relative w-full overflow-hidden bg-white shadow-2xl transition-colors dark:bg-gray-900"
        :class="[maxHeight, roundedClass]"
      >
        <!-- Header -->
        <div class="z-10 flex items-center justify-between bg-white px-6 py-4 shadow-sm dark:bg-gray-800">
          <div>
            <h2 class="text-xl font-bold text-gray-800 dark:text-gray-100">{{ title }}</h2>
            <p v-if="subtitle" class="mt-0.5 text-xs text-gray-400 dark:text-gray-500">{{ subtitle }}</p>
          </div>
          <CloseButton @click="close" />
        </div>

        <!-- Content -->
        <div class="custom-scrollbar flex-1 overflow-y-auto" :class="contentClass">
          <slot></slot>
        </div>

        <!-- Optional Footer -->
        <div v-if="$slots.footer" class="z-10 border-t border-gray-100 bg-white p-4 shadow-[0_-10px_30px_rgba(0,0,0,0.05)] dark:border-gray-700 dark:bg-gray-800">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
  </Teleport>
</template>

<script setup lang="ts">
import CloseButton from "./CloseButton.vue";

interface Props {
  modelValue: boolean;
  title: string;
  subtitle?: string;
  maxHeight?: string; // e.g. "h-[85vh]"
  roundedClass?: string; // e.g. "rounded-t-[2rem]"
  contentClass?: string;
}

withDefaults(defineProps<Props>(), {
  maxHeight: "h-auto",
  roundedClass: "rounded-t-3xl",
  contentClass: "px-4 py-6"
});

const emit = defineEmits(["update:modelValue", "close"]);

const close = () => {
  emit("update:modelValue", false);
  emit("close");
};
</script>
