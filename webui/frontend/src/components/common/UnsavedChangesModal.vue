<template>
  <Modal
    :model-value="modelValue"
    content-class="max-w-md"
    @update:model-value="onVisibleChange"
    @close="onStay"
  >
    <div class="bg-white dark:bg-gray-800 rounded-lg shadow-xl">
      <div class="px-6 pt-6 pb-4">
        <div class="flex items-start">
          <div class="rounded-full p-3 mr-4 bg-amber-100 dark:bg-amber-900/30">
            <IconWarning class="w-6 h-6 text-amber-600 dark:text-amber-400" />
          </div>
          <div>
            <h3 class="text-lg font-semibold text-theme-strong">{{ title }}</h3>
            <p class="text-sm text-theme-subtle mt-1">{{ message }}</p>
          </div>
        </div>
        <slot></slot>
      </div>
      <div class="px-6 pb-6 flex flex-col sm:flex-row sm:flex-wrap justify-end gap-3">
        <UiButton variant="secondary" @click="onStay">
          {{ stayText }}
        </UiButton>
        <UiButton variant="secondary" :disabled="busy" @click="onDiscard">
          {{ discardText }}
        </UiButton>
        <UiButton :disabled="busy" @click="onSave">
          {{ saveText }}
        </UiButton>
      </div>
    </div>
  </Modal>
</template>

<script setup lang="ts">
import Modal from './Modal.vue'
import UiButton from '../ui/UiButton.vue'
import { IconWarning } from '@/components/icons'

defineProps<{
  modelValue: boolean
  title: string
  message: string
  saveText: string
  discardText: string
  stayText: string
  /** Disables the save button while a save request is in flight. */
  busy?: boolean
}>()

const emit = defineEmits<{
  'update:modelValue': [value: boolean]
  save: []
  discard: []
  stay: []
}>()

function onVisibleChange(value: boolean) {
  emit('update:modelValue', value)
}

function onSave() {
  emit('save')
}

function onDiscard() {
  emit('discard')
}

/** Escape and backdrop clicks dismiss the dialog, which counts as staying. */
function onStay() {
  emit('stay')
}
</script>

<style scoped>
</style>
