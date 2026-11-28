<template>
  <Teleport to="body">
    <Transition name="modal">
      <div
        ref="rootRef"
        v-show="modelValue"
        class="fixed inset-0 z-[100] flex items-center justify-center"
        @mousedown="onMouseDown"
        @mouseup="onMouseUp"
      >
        <div class="modal-overlay absolute inset-0 bg-black bg-opacity-50" />
        <div ref="panelRef" tabindex="-1" class="modal-panel relative w-full mx-4" :class="contentClass" :style="contentStyle">
          <div v-if="hasShell" class="modal-card flex w-full flex-col rounded-lg shadow-xl" :style="{ maxHeight }">
            <div class="flex justify-between items-center px-6 py-4 border-b border-divider">
              <div class="min-w-0">
                <h3 class="text-lg font-semibold text-theme-strong">{{ title }}</h3>
                <p v-if="description" class="mt-1 text-sm text-theme-subtle">{{ description }}</p>
              </div>
              <div class="flex items-center gap-2">
                <slot name="header-actions" />
                <button
                  v-if="closeable"
                  type="button"
                  class="text-theme-faint text-theme-faint-hover"
                  :aria-label="closeLabel || t('common.close')"
                  :disabled="closeDisabled"
                  @click="close"
                >
                  <IconClose class="w-6 h-6" />
                </button>
              </div>
            </div>
            <slot name="subheader" />
            <slot />
            <div v-if="$slots.footer" :class="footerClass">
              <slot name="footer" />
            </div>
          </div>
          <slot v-else />
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { computed, provide, ref, useSlots, watch, onUnmounted } from 'vue'
import { useI18n } from 'vue-i18n'
import { IconClose } from '@/components/icons'
import { modalPanelKey, registerModal } from './modalFocus'

const props = withDefaults(defineProps<{
  modelValue: boolean
  contentClass?: string
  contentStyle?: string
  persistent?: boolean
  title?: string
  description?: string
  maxHeight?: string
  footerClass?: string
  closeable?: boolean
  closeDisabled?: boolean
  closeLabel?: string
}>(), {
  contentClass: '',
  contentStyle: '',
  persistent: false,
  title: undefined,
  description: undefined,
  maxHeight: '90vh',
  footerClass: 'flex justify-end space-x-3 px-6 py-4 border-t border-divider',
  closeable: true,
  closeDisabled: false,
  closeLabel: '',
})

const slots = useSlots()
const { t } = useI18n()

// Existing callers pass their whole dialog as the default slot. The shell is
// only enabled once a caller opts in with a title, footer or header-actions slot.
const hasShell = computed(() => props.title !== undefined || !!slots.headerActions || !!slots.footer)

const emit = defineEmits<{
  'update:modelValue': [value: boolean]
  close: []
}>()

function close() {
  emit('update:modelValue', false)
  emit('close')
}

const mouseDownTarget = ref<EventTarget | null>(null)
const panelRef = ref<HTMLElement | null>(null)
const rootRef = ref<HTMLElement | null>(null)
provide(modalPanelKey, panelRef)

function isOutsidePanel(target: EventTarget | null): boolean {
  const panel = panelRef.value
  if (!panel || !(target instanceof Node)) return false
  return !panel.contains(target)
}

function onMouseDown(e: MouseEvent) {
  mouseDownTarget.value = isOutsidePanel(e.target) ? e.target : null
}

function onMouseUp(e: MouseEvent) {
  if (
    !props.persistent &&
    mouseDownTarget.value !== null &&
    isOutsidePanel(e.target)
  ) {
    close()
  }
  mouseDownTarget.value = null
}

let unregister: (() => void) | undefined

watch(() => props.modelValue, (open) => {
  unregister?.()
  unregister = undefined
  if (open && rootRef.value && panelRef.value) {
    unregister = registerModal(rootRef.value, panelRef.value, close)
  }
}, { immediate: true, flush: 'post' })

watch(rootRef, root => {
  if (root && props.modelValue && panelRef.value && !unregister) {
    unregister = registerModal(root, panelRef.value, close)
  }
}, { flush: 'post' })

onUnmounted(() => unregister?.())
</script>

<style scoped>
/* --- enter / leave transitions (Vue <Transition name="modal">) --- */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}
.modal-enter-active .modal-panel,
.modal-leave-active .modal-panel {
  transition: all 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}
.modal-enter-from .modal-panel,
.modal-leave-to .modal-panel {
  opacity: 0;
  transform: scale(0.95) translateY(-10px);
}
</style>
