<template>
  <Modal
    v-model="visible"
    content-class="max-w-md"
    @close="onCancel"
  >
    <div class="bg-white dark:bg-gray-800 rounded-lg shadow-xl">
      <div class="px-6 pt-6 pb-4">
        <div class="flex items-center mb-2">
          <div :class="variant === 'info' ? 'bg-status-info-soft dark:bg-blue-900/30' : 'bg-red-100 dark:bg-red-900/30'" class="rounded-full p-3 mr-4">
            <IconInfo v-if="variant === 'info'" class="w-6 h-6 text-status-info dark:text-blue-400" />
            <IconWarning v-else class="w-6 h-6 text-status-danger dark:text-red-400" />
          </div>
          <div>
            <h3 class="text-lg font-semibold text-theme-strong">
              {{ title }}
            </h3>
            <p class="text-sm text-theme-subtle mt-1">
              {{ message }}
            </p>
          </div>
        </div>
        <slot></slot>
      </div>
      <div class="px-6 pb-6 flex justify-end space-x-3">
        <UiButton variant="secondary" @click="onCancel">
          {{ cancelText }}
        </UiButton>
        <UiButton
          :variant="variant === 'info' ? 'primary' : 'danger'"
          :disabled="confirmDisabled"
          @click="onConfirm"
        >
          {{ confirmText }}
        </UiButton>
      </div>
    </div>
  </Modal>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Modal from './Modal.vue'
import UiButton from '../ui/UiButton.vue'
import { IconInfo, IconWarning } from '@/components/icons'

const props = withDefaults(defineProps<{
  title?: string
  message?: string
  cancelText?: string
  confirmText?: string
  variant?: 'danger' | 'info'
  confirmDisabled?: boolean
}>(), {
  title: 'Confirm Delete',
  message: 'This action cannot be undone.',
  cancelText: 'Cancel',
  confirmText: 'Delete',
  variant: 'danger',
  confirmDisabled: false,
})

const emit = defineEmits<{
  confirm: []
  cancel: []
}>()

const visible = ref(false)

function open() {
  visible.value = true
}

function close() {
  visible.value = false
}

function onCancel() {
  close()
  emit('cancel')
}

function onConfirm() {
  if (props.confirmDisabled) return
  close()
  emit('confirm')
}

defineExpose({ open, close })
</script>
