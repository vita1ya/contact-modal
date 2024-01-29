<template>
  <div class="email-form">
    <div class="email-form__row">
      <ORInput
        class="email-form__input email-form__input_stretched"
        :disabled="email.archived"
        label="Адрес почты"
        label-as-placeholder
        :model-value="email.email"
        required
        :validations="validations.email"
        @update:model-value="updateData('email', $event)"
      />

      <ORButton
        v-if="!creation && !email.archived && false"
        class="email-form__button"
        :color-scheme="ButtonColorScheme.LIGHTGREEN"
      >
        <ORIcon name="email" size="24" />
      </ORButton>
    </div>

    <div v-if="false" class="email-form__row">
      <ORInput label="Дата добавления" label-as-placeholder type="date" />

      <ORInput
        disabled
        label="Дата архивации"
        label-as-placeholder
        type="date"
      />
    </div>

    <ORTextarea
      v-if="false"
      :disabled="email.archived"
      label="Комментарий"
      label-as-placeholder
      rows="2"
    />

    <ORCheckbox
      class="email-form__checkbox"
      :disabled="email.archived || mainDisabled"
      :model-value="email.main"
      text="Основной"
      @update:model-value="updateData('main', $event)"
    />
  </div>
</template>

<script lang="ts" setup>
import { computed, watch } from "vue";
import {
  ButtonColorScheme,
  ORButton,
  ORCheckbox,
  ORInput,
  ORTextarea,
  TagColor,
  type ITag,
} from "@flora/uikit";
import { email as emailValidationRule, helpers } from "@vuelidate/validators";
import type { EditableContactEmail, IContactModalProps } from "@/types";
import { api } from "@/api";
import type { IErrorInfo } from "@flora/common-data";

const props = defineProps<{
  personUuid: string;
  creation: boolean;
  email: EditableContactEmail;
  mainDisabled?: boolean;
  userName: string;
  uuid: string | null;
}>();

const emit = defineEmits<{
  error: [payload: IErrorInfo];
  setModalProps: [props: IContactModalProps];
  updateData: [value: EditableContactEmail];
  updateFooterHidden: [value: boolean];
}>();

defineOptions({
  inheritAttrs: false,
});

const updateData = <Key extends keyof EditableContactEmail>(
  key: Key,
  value: EditableContactEmail[Key]
) => {
  emit("updateData", {
    ...props.email,
    [key]: value,
  });
};

const validations = computed(() => ({
  email: {
    length: helpers.withMessage(
      "Некорректный адрес электронной почты",
      emailValidationRule
    ),
  },
}));

defineExpose({
  create() {
    return api.createEmail(props.personUuid, {
      email: {
        email_address: props.email.email,
        is_main: props.email.main,
      },
      user_d: props.userName,
      dml: "I",
    });
  },
  save() {
    if (!props.uuid) {
      throw Error("Отсутствует уникальный идентификатор номера телефона");
    }

    return api.updateEmail(props.uuid, {
      email: {
        email_address: props.email.email,
        is_main: props.email.main,
      },
      user_d: props.userName,
      dml: "U",
    });
  },
  archive() {
    if (!props.uuid) {
      throw Error("Отсутствует уникальный идентификатор номера телефона");
    }

    return api.archiveEmail(props.uuid);
  },
  unarchive() {
    throw Error("Разархивирование недоступно");
  },
});

watch(
  [() => props.creation, () => props.email.archived],
  ([creation, archived]) => {
    if (creation) return;

    const tags: ITag[] = [];

    if (archived) {
      tags.push({
        text: "Архив",
        color: TagColor.GREY,
        icon: "expired",
      });
    }

    emit("setModalProps", {
      title: "Почта",
      tags,
    });
  },
  {
    immediate: true,
  }
);
</script>

<style scoped>
.email-form {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.email-form__row {
  display: flex;
  gap: 8px;
}

.email-form__input_stretched {
  flex: 1;
}

.email-form__button {
  width: 68px;
  height: 68px;
  padding: 0;
}

.email-form__checkbox {
  margin-top: 8px;
}
</style>
