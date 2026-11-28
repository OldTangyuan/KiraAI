<template>
  <Modal
    v-model="visible"
    content-class="max-w-md"
    :title="$t('settings.token_title')"
    :description="$t('settings.token_desc')"
  >
    <div class="px-6 py-4 flex-1 overflow-y-auto space-y-4">
      <div>
        <label for="modal-token-old" class="block text-sm font-medium text-theme-body mb-2">{{ $t('settings.token_old') }}</label>
        <UiInput
          id="modal-token-old"
          v-model="oldToken"
          type="password"
          :placeholder="$t('settings.token_old_placeholder')"
          class="w-full px-3 py-2 rounded-lg"
        />
      </div>
      <div>
        <label for="modal-token-new" class="block text-sm font-medium text-theme-body mb-2">{{ $t('settings.token_new') }}</label>
        <UiInput
          id="modal-token-new"
          v-model="newToken"
          type="password"
          :placeholder="$t('settings.token_new_placeholder')"
          class="w-full px-3 py-2 rounded-lg"
        />
      </div>
      <div>
        <label for="modal-token-confirm" class="block text-sm font-medium text-theme-body mb-2">{{ $t('settings.token_confirm') }}</label>
        <UiInput
          id="modal-token-confirm"
          v-model="confirmToken"
          type="password"
          :placeholder="$t('settings.token_confirm_placeholder')"
          class="w-full px-3 py-2 rounded-lg"
        />
      </div>
    </div>

    <template #footer>
      <UiButton variant="secondary" @click="visible = false">{{ $t('settings.backup_confirm_cancel') }}</UiButton>
      <UiButton :disabled="tokenChanging" @click="tokenConfirmRef?.open()">{{ tokenChanging ? $t('settings.token_changing') : $t('settings.token_submit') }}</UiButton>
    </template>

    <!-- Confirm Modal -->
    <ConfirmModal
      ref="tokenConfirmRef"
      variant="danger"
      :title="$t('settings.token_submit')"
      :message="$t('settings.token_confirm_dialog')"
      :cancel-text="$t('settings.backup_confirm_cancel')"
      :confirm-text="$t('settings.backup_confirm_ok')"
      @confirm="handleChangeToken"
    />
  </Modal>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue'
import { useI18n } from 'vue-i18n'
import { useRouter } from 'vue-router'
import { notify } from '@/composables/useNotification'
import { useChangeTokenModal } from '@/composables/useChangeTokenModal'
import { changeAccessToken } from '@/api/settings'
import Modal from './Modal.vue'
import ConfirmModal from './ConfirmModal.vue'
import UiButton from '../ui/UiButton.vue'
import UiInput from '../ui/UiInput.vue'

const { t } = useI18n()
const router = useRouter()
const { visible } = useChangeTokenModal()

const oldToken = ref('')
const newToken = ref('')
const confirmToken = ref('')
const tokenChanging = ref(false)
const tokenConfirmRef = ref<InstanceType<typeof ConfirmModal>>()

// Clear inputs when modal opens
watch(visible, (val) => {
  if (val) {
    oldToken.value = ''
    newToken.value = ''
    confirmToken.value = ''
  }
})

async function handleChangeToken() {
  if (!oldToken.value) {
    notify(t('settings.token_old_required'), 'error')
    return
  }
  if (!newToken.value || newToken.value.length < 6) {
    notify(t('settings.token_new_too_short'), 'error')
    return
  }
  if (newToken.value !== confirmToken.value) {
    notify(t('settings.token_mismatch'), 'error')
    return
  }
  tokenChanging.value = true
  try {
    await changeAccessToken(oldToken.value, newToken.value)
    notify(t('settings.token_changed_logout'), 'success')
    visible.value = false
    localStorage.removeItem('jwt_token')
    router.push('/login')
    return
  } catch (err: any) {
    const detail = err?.response?.data?.detail
    if (detail === 'Old token is incorrect') {
      notify(t('settings.token_old_wrong'), 'error')
    } else {
      notify(t('settings.token_change_failed'), 'error')
    }
  } finally {
    tokenChanging.value = false
  }
}
</script>
