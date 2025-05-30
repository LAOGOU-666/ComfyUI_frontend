<template>
  <Form
    v-slot="$form"
    class="flex flex-col gap-6"
    :resolver="zodResolver(signInSchema)"
    @submit="onSubmit"
  >
    <!-- Email Field -->
    <div class="flex flex-col gap-2">
      <label class="opacity-80 text-base font-medium mb-2" :for="emailInputId">
        {{ t('auth.login.emailLabel') }}
      </label>
      <InputText
        :id="emailInputId"
        autocomplete="email"
        class="h-10"
        name="email"
        type="text"
        :placeholder="t('auth.login.emailPlaceholder')"
        :invalid="$form.email?.invalid"
      />
      <small v-if="$form.email?.invalid" class="text-red-500">{{
        $form.email.error.message
      }}</small>
    </div>

    <!-- Password Field -->
    <div class="flex flex-col gap-2">
      <div class="flex justify-between items-center mb-2">
        <label
          class="opacity-80 text-base font-medium"
          for="comfy-org-sign-in-password"
        >
          {{ t('auth.login.passwordLabel') }}
        </label>
        <span
          class="text-muted text-base font-medium cursor-pointer select-none"
          :class="{
            'text-link-disabled': !$form.email?.value || $form.email?.invalid
          }"
          @click="handleForgotPassword($form.email?.value, $form.email?.valid)"
        >
          {{ t('auth.login.forgotPassword') }}
        </span>
      </div>
      <Password
        input-id="comfy-org-sign-in-password"
        pt:pc-input-text:root:autocomplete="current-password"
        name="password"
        :feedback="false"
        toggle-mask
        :placeholder="t('auth.login.passwordPlaceholder')"
        :class="{ 'p-invalid': $form.password?.invalid }"
        fluid
        class="h-10"
      />
      <small v-if="$form.password?.invalid" class="text-red-500">{{
        $form.password.error.message
      }}</small>
    </div>

    <!-- Submit Button -->
    <ProgressSpinner v-if="loading" class="w-8 h-8" />
    <Button
      v-else
      type="submit"
      :label="t('auth.login.loginButton')"
      class="h-10 font-medium mt-4"
    />
  </Form>
</template>

<script setup lang="ts">
import { Form, FormSubmitEvent } from '@primevue/forms'
import { zodResolver } from '@primevue/forms/resolvers/zod'
import Button from 'primevue/button'
import InputText from 'primevue/inputtext'
import Password from 'primevue/password'
import ProgressSpinner from 'primevue/progressspinner'
import { useToast } from 'primevue/usetoast'
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'

import { useFirebaseAuthActions } from '@/composables/auth/useFirebaseAuthActions'
import { type SignInData, signInSchema } from '@/schemas/signInSchema'
import { useFirebaseAuthStore } from '@/stores/firebaseAuthStore'

const authStore = useFirebaseAuthStore()
const firebaseAuthActions = useFirebaseAuthActions()
const loading = computed(() => authStore.loading)
const toast = useToast()

const { t } = useI18n()

const emit = defineEmits<{
  submit: [values: SignInData]
}>()

const emailInputId = 'comfy-org-sign-in-email'

const onSubmit = (event: FormSubmitEvent) => {
  if (event.valid) {
    emit('submit', event.values as SignInData)
  }
}

const handleForgotPassword = async (
  email: string,
  isValid: boolean | undefined
) => {
  if (!email || !isValid) {
    toast.add({
      severity: 'warn',
      summary: t('auth.login.emailPlaceholder'),
      life: 5_000
    })
    // Focus the email input
    document.getElementById(emailInputId)?.focus?.()
    return
  }
  await firebaseAuthActions.sendPasswordReset(email)
}
</script>

<style scoped>
.text-link-disabled {
  @apply opacity-50 cursor-not-allowed;
}
</style>
