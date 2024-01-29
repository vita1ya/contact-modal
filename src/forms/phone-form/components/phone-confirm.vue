<template>
  <div class="phone-confirm">
    <KeepAlive>
      <template v-if="!isCodeChecking">
        <div class="phone-confirm__content">
          <slot :phone-disabled="isCodeSended"></slot>

          <ORInput
            v-if="isCodeSended"
            ref="input"
            label="Код из СМС"
            label-as-placeholder
            :model-value="code"
            pattern="[0-9]+"
            restrict-typing
            @update:model-value="checkCode"
          />
        </div>

        <div class="phone-confirm__footer">
          <ORButton
            :color-scheme="ButtonColorScheme.LIGHTPINK"
            :disabled="areErrors || isCodeSended"
            :loading="isCodeSending"
            @click="sendCode"
          >
            <VueCountdown
              v-if="isCodeSended"
              v-slot="{ minutes, seconds }"
              :time="TIME"
              :transform="transformCountdownTime"
              @end="isCodeSended = false"
            >
              {{ `${minutes}:${seconds}` }}
            </VueCountdown>

            <template v-else>Подтвердить по SMS</template>
          </ORButton>
        </div>
      </template>

      <ORLoader v-else class="phone-confirm__loader" />
    </KeepAlive>
  </div>
</template>

<script lang="ts" setup>
import { computed, nextTick, ref } from "vue";
import VueCountdown from "@chenfengyuan/vue-countdown";
import { ButtonColorScheme, ORButton, ORInput, ORLoader } from "@flora/uikit";
import { useVuelidate } from "@vuelidate/core";
import type { EditableContactPhone } from "@/types";
import { api } from "@/api";
import type { IErrorInfo } from "@flora/common-data";

const props = defineProps<{
  personUuid: string;
  data: EditableContactPhone;
  userName: string;
}>();

const emit = defineEmits<{
  (e: "confirmed");
  (e: "error", payload: IErrorInfo);
  (e: "updateConfirm", value: boolean);
  <Key extends keyof EditableContactPhone>(
    e: "updateData",
    key: Key,
    value: EditableContactPhone[Key]
  );
}>();

defineOptions({
  inheritAttrs: false,
});

const v$ = useVuelidate();

const TIME = 2 * 60000 - 1;
const CODE_LENGTH = 4;

const input = ref();

const code = ref("");

const isCodeSended = ref(false);

const isCodeSending = ref(false);

const isCodeChecking = ref(false);

const sendCode = async () => {
  isCodeSending.value = true;

  try {
    await api.sendPhoneVerificationCode(props.personUuid, props.data.phone);

    isCodeSended.value = true;

    await nextTick();

    input.value?.$el.querySelector("input")?.focus();
  } catch (error) {
    emit("error", {
      message: "Не удалось отправить код",
      error,
    });
  } finally {
    isCodeSending.value = false;
  }
};

const checkCode = async (value: string) => {
  if (value.length <= CODE_LENGTH) code.value = value;

  if (!isCodeSended.value || code.value.length !== CODE_LENGTH) {
    return;
  }

  isCodeChecking.value = true;

  try {
    await api.checkPhoneVerificationCode(props.personUuid, {
      user_d: props.userName,
      code: code.value,
      telephones: {
        number: props.data.phone,
      },
    });

    emit("confirmed");
  } catch (error) {
    emit("error", {
      message: "Не удалось подтвердить код",
      error,
    });
  } finally {
    isCodeChecking.value = false;
  }
};

const transformCountdownTime = (props: Record<string, number>) => {
  const formattedProps: Record<string, string> = {};

  Object.entries(props).forEach(([key, value]) => {
    formattedProps[key] = value < 10 ? `0${value}` : String(value);
  });

  return formattedProps;
};

const areErrors = computed(() => !!v$.value.$errors.length);
</script>

<style scoped>
.phone-confirm {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 36px;
}

.phone-confirm:has(.phone-confirm__loader) {
  min-height: 50px;
}

.phone-confirm__content {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
</style>
