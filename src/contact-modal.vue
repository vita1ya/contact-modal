<template>
  <ORModal
    :append-to="appendTo"
    v-bind="modalProps"
    :value="modelValue"
    @back="modalProps?.back"
    @close="emit('update:modelValue', false)"
  >
    <ContactWrapper
      :data="data"
      in-modal
      :main-email-disabled="mainEmailDisabled"
      :main-phone-disabled="mainPhoneDisabled"
      :person-uuid="personUuid"
      :preselected-type="preselectedType"
      :user-name="userName"
      @archived="execute(() => emit('archived'))"
      @created="execute(() => emit('created'))"
      @error="emit('error', $event)"
      @saved="execute(() => emit('saved'))"
      @set-modal-props="setModalProps"
      @unarchived="execute(() => emit('unarchived'))"
    />
  </ORModal>
</template>

<script lang="ts" setup>
import { ref } from "vue";
import { ORModal } from "@flora/uikit";
import {
  ContactWrapper,
  type ContactData,
  type IContactModalProps,
  type ContactTypes,
} from "@/index";
import type { IErrorInfo } from "@flora/common-data";

withDefaults(
  defineProps<{
    appendTo: string;
    personUuid: string;
    data?: ContactData;
    mainEmailDisabled?: boolean;
    mainPhoneDisabled?: boolean;
    modelValue: boolean;
    preselectedType?: ContactTypes | null;
    userName: string;
  }>(),
  {
    data: null,
  }
);

const emit = defineEmits<{
  archived: [];
  created: [];
  error: [payload: IErrorInfo];
  saved: [];
  unarchived: [];
  "update:modelValue": [value: boolean];
}>();

const DEFAULT_MODAL_WIDTH = "540";

const modalProps = ref<IContactModalProps | null>(null);

const setModalProps = (newProps: IContactModalProps) => {
  modalProps.value = {
    ...newProps,
    width: newProps.width || DEFAULT_MODAL_WIDTH,
  };
};

const execute = (callback: () => void) => {
  callback();
  emit("update:modelValue", false);
};
</script>
