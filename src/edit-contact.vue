<template>
  <div class="edit-contact">
    <div class="edit-contact__content">
      <slot></slot>
    </div>

    <div v-if="!footerHidden" class="edit-contact__footer">
      <ORButton
        v-if="archived"
        :color-scheme="ButtonColorScheme.LIGHTGREY"
        @click="emit('unarchive')"
      >
        Разархивировать
      </ORButton>

      <template v-else>
        <ORButton
          :color-scheme="ButtonColorScheme.LIGHTGREY"
          @click="emit('archive')"
        >
          Архивировать
        </ORButton>

        <ORButton
          :color-scheme="ButtonColorScheme.LIGHTPINK"
          :disabled="areErrors || verified"
          @click="emit('save')"
        >
          Сохранить
        </ORButton>
      </template>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { computed, onBeforeMount } from "vue";
import { ButtonColorScheme, ORButton } from "@flora/uikit";
import { useVuelidate } from "@vuelidate/core";
import type { IContactModalProps } from "@/types";

defineProps<{
  archived: boolean;
  footerHidden: boolean;
  verified: boolean;
}>();

const emit = defineEmits<{
  archive: [];
  save: [];
  setModalProps: [props: IContactModalProps];
  unarchive: [];
}>();

const v$ = useVuelidate();

const areErrors = computed(() => !!v$.value.$errors.length);

onBeforeMount(() => {
  emit("setModalProps", {
    title: "Добавить способ связи",
  });
});
</script>

<style scoped>
.edit-contact {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.edit-contact__footer {
  display: flex;
  gap: 8px;
}
</style>
