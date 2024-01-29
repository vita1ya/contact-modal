<template>
  <div class="phone-data">
    <slot :phone-disabled="false"></slot>

    <div class="phone-data__row">
      <ORLink
        v-if="!creation && !data.verified && !data.archived"
        class="phone-data__link phone-data__confirm-link t1"
        @click="confirm"
      >
        Подтвердить
      </ORLink>
    </div>

    <div v-if="false" class="phone-data__row">
      <ORInput label="Дата добавления" label-as-placeholder type="date" />

      <ORInput
        disabled
        label="Дата архивации"
        label-as-placeholder
        type="date"
      />
    </div>

    <ORTextarea
      :disabled="data.archived || data.verified"
      label="Комментарий"
      label-as-placeholder
      :model-value="data.comment"
      rows="2"
      @update:model-value="emit('updateData', 'comment', $event)"
    />

    <ORCheckbox
      class="phone-data__checkbox"
      :disabled="data.archived || data.verified || mainDisabled"
      :model-value="data.main"
      text="Основной"
      @update:model-value="emit('updateData', 'main', $event)"
    />
  </div>
</template>

<script lang="ts" setup>
import { ORCheckbox, ORInput, ORLink, ORTextarea } from "@flora/uikit";
import type { EditableContactPhone } from "@/types";
import type { IErrorInfo } from "@flora/common-data";

defineProps<{
  creation: boolean;
  data: EditableContactPhone;
  mainDisabled?: boolean;
}>();

const emit = defineEmits<{
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

const confirm = () => {
  emit("updateConfirm", true);
};
</script>

<style scoped>
.phone-data {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.phone-data__row {
  display: flex;
  gap: 8px;
}

.phone-data__link {
  color: var(--content-text-secondary);
}

.phone-data__confirm-link {
  margin: 8px 0 16px;
}

.phone-data__checkbox {
  margin-top: 8px;
}
</style>
