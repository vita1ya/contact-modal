<template>
  <div class="form-wrapper">
    <component
      :is="component"
      ref="form"
      :creation="creation"
      :email="email"
      :main-disabled="mainDisabled"
      :person-uuid="personUuid"
      :phone="phone"
      :user-name="userName"
      :uuid="uuid"
      @error="emit('error', $event)"
      @set-modal-props="setModalProps"
      @update-data="emit('updateData', $event)"
      @update-footer-hidden="emit('updateFooterHidden', $event)"
    />
  </div>
</template>

<script lang="ts" setup>
import { computed, defineAsyncComponent, markRaw, ref, type Raw } from "vue";
import { ORLoader } from "@flora/uikit";
import {
  ContactTypes,
  type EditableContactData,
  type EditableContactEmail,
  type EditableContactPhone,
  type IContactModalProps,
} from "@/types";
import type { IErrorInfo } from "@flora/common-data";

const EmailForm = defineAsyncComponent({
  loader: () => import("../forms/email-form.vue"),
  delay: 0,
  loadingComponent: ORLoader,
});

const PhoneForm = defineAsyncComponent({
  loader: () => import("../forms/phone-form/phone-form.vue"),
  delay: 0,
  loadingComponent: ORLoader,
});

const props = defineProps<{
  personUuid: string;
  creation: boolean;
  data: EditableContactData;
  mainDisabled?: boolean;
  type: ContactTypes;
  userName: string;
  uuid: string | null;
}>();

const emit = defineEmits<{
  error: [payload: IErrorInfo];
  setModalProps: [props: IContactModalProps];
  updateData: [value: EditableContactData];
  updateFooterHidden: [value: boolean];
}>();

const component = computed<Raw<typeof EmailForm | typeof PhoneForm>>(() => {
  switch (props.type) {
    case ContactTypes.Email:
      return markRaw(EmailForm);
    default:
      return markRaw(PhoneForm);
  }
});

const setModalProps = (newProps: IContactModalProps) => {
  if (props.creation) return;

  emit("setModalProps", newProps);
};

const phone = computed<EditableContactPhone>(() => {
  if (props.data && "phone" in props.data) return props.data;

  return {
    phone: "",
    comment: "",
    main: false,
    archived: false,
    verified: false,
  };
});

const email = computed<EditableContactEmail>(() => {
  if (props.data && "email" in props.data) return props.data;

  return {
    email: "",
    comment: "",
    main: false,
    archived: false,
  };
});

const form = ref();

defineExpose({
  async create() {
    await form.value?.create();
  },
  async save() {
    await form.value?.save();
  },
  async archive() {
    await form.value?.archive();
  },
  async unarchive() {
    await form.value?.unarchive();
  },
});
</script>

<style scoped>
.form-wrapper {
  position: relative;
  --or-textarea_border-radius: 12px;
}

.form-wrapper:has(.or-loader) {
  min-height: 50px;
}
</style>
