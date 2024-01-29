<template>
  <div class="create-contact">
    <div class="create-contact__content">
      <ORSelect
        :clearable="false"
        :disabled="preselected"
        label="Тип"
        label-as-placeholder
        :model-value="type"
        no-shadow
        :options="types"
        @update:model-value="emit('updateType', $event)"
      />

      <slot></slot>
    </div>

    <div class="create-contact__footer">
      <ORButton
        :color-scheme="ButtonColorScheme.LIGHTPINK"
        :disabled="areErrors"
        @click="emit('create')"
      >
        Добавить
      </ORButton>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, onBeforeMount } from "vue";
import { ButtonColorScheme, ORButton, ORSelect } from "@flora/uikit";
import { useVuelidate } from "@vuelidate/core";
import { ContactTypes, type IContactModalProps } from "@/types";

defineProps<{
  preselected: boolean;
  type: ContactTypes;
}>();

const emit = defineEmits<{
  create: [];
  setModalProps: [props: IContactModalProps];
  updateType: [value: ContactTypes];
}>();

const v$ = useVuelidate();

const areErrors = computed(() => !!v$.value.$errors.length);

const types = computed(() => Object.values(ContactTypes));

onBeforeMount(() => {
  emit("setModalProps", {
    title: "Добавить способ связи",
  });
});
</script>

<style scoped>
.create-contact,
.create-contact__content {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.create-contact {
  gap: 32px;
}

.create-contact__content {
  gap: 8px;
}
</style>
