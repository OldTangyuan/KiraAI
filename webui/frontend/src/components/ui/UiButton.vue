<template>
  <button :type="type" :class="classes">
    <slot />
  </button>
</template>

<script setup lang="ts">
import { computed } from 'vue'

type Variant = 'primary' | 'secondary' | 'danger' | 'success'

const props = withDefaults(defineProps<{
  variant?: Variant
  type?: 'button' | 'submit' | 'reset'
}>(), {
  variant: 'primary',
  type: 'button',
})

const variantClasses: Record<Variant, string> = {
  primary: 'bg-blue-600 dark:bg-blue-700 text-white hover:bg-blue-700 dark:hover:bg-blue-600',
  secondary: 'border border-control text-theme-body hover:bg-gray-50 dark:hover:bg-gray-700',
  danger: 'bg-red-600 dark:bg-red-700 text-white hover:bg-red-700 dark:hover:bg-red-600',
  success: 'bg-green-600 dark:bg-green-700 text-white hover:bg-green-700 dark:hover:bg-green-600',
}

const classes = computed(() => [
  'px-4 py-2 rounded-lg transition-colors disabled:opacity-50 disabled:cursor-not-allowed',
  variantClasses[props.variant],
])
</script>