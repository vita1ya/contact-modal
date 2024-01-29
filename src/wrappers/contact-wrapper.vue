<template>
  <div class="contact-wrapper">
    <KeepAlive>
      <template v-if="!isLoading">
        <component
          :is="component"
          :archived="isArchived"
          :footer-hidden="isFooterHidden"
          :preselected="!!preselectedType"
          :type="type"
          :verified="isVerified"
          @archive="archive"
          @create="create"
          @save="save"
          @set-modal-props="setModalProps"
          @unarchive="unarchive"
          @update-type="updateType"
        >
          <FormWrapper
            ref="form"
            :creation="isCreation"
            :data="editableData"
            :main-disabled="mainDisabled"
            :person-uuid="personUuid"
            :type="type"
            :user-name="userName"
            :uuid="uuid"
            @error="emit('error', $event)"
            @set-modal-props="setModalProps"
            @update-data="updateEditableData"
            @update-footer-hidden="updateFooterHidden"
          />
        </component>
      </template>

      <ORLoader v-else class="contact-wrapper__loader" />
    </KeepAlive>
  </div>
</template>

<script lang="ts" setup>
import {
  computed,
  defineAsyncComponent,
  markRaw,
  ref,
  toRaw,
  type Raw,
} from "vue";
import { ORLoader } from "@flora/uikit";
import { useVuelidate } from "@vuelidate/core";
import {
  ContactTypes,
  type ContactData,
  type EditableContactData,
  type IContactModalProps,
} from "@/types";
import FormWrapper from "../wrappers/form-wrapper.vue";
import type { IErrorInfo } from "@flora/common-data";

const CreateContact = defineAsyncComponent({
  loader: () => import("../create-contact.vue"),
  delay: 0,
  loadingComponent: ORLoader,
});

const EditContact = defineAsyncComponent({
  loader: () => import("../edit-contact.vue"),
  delay: 0,
  loadingComponent: ORLoader,
});

const props = withDefaults(
  defineProps<{
    personUuid: string;
    data?: ContactData;
    inModal?: boolean;
    mainEmailDisabled?: boolean;
    mainPhoneDisabled?: boolean;
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
  setModalProps: [props: IContactModalProps];
  unarchived: [];
}>();

const v$ = useVuelidate();

const uuid = ref<string | null>(props.data?.uuid || null);

const editableData = ref<EditableContactData>(
  !props.preselectedType ? structuredClone(toRaw(props.data)) : null
);

const updateEditableData = (value: EditableContactData) => {
  editableData.value = structuredClone(toRaw(value));
};

const isCreation = computed(() => !uuid.value);

const isArchived = computed(() => !!editableData.value?.archived);

const isVerified = computed(
  () =>
    !!editableData.value &&
    "phone" in editableData.value &&
    editableData.value.verified
);

const component = computed<Raw<typeof CreateContact | typeof EditContact>>(() =>
  isCreation.value ? markRaw(CreateContact) : markRaw(EditContact)
);

const initType = (): ContactTypes => {
  if (props.preselectedType) {
    uuid.value = null;
    return props.preselectedType;
  }

  if (props.data) {
    if ("phone" in props.data) return ContactTypes.Phone;
    else if ("email" in props.data) return ContactTypes.Email;
  }

  return ContactTypes.Phone;
};

const type = ref(initType());

const updateType = (value: unknown /* ContactTypes */) => {
  type.value = value as ContactTypes;
  editableData.value = null;
};

const form = ref();

const isLoading = ref(false);

const isFooterHidden = ref(false);

const updateFooterHidden = (value: boolean) => {
  isFooterHidden.value = value;
};

const mainDisabled = computed(
  () =>
    (props.mainPhoneDisabled && type.value === ContactTypes.Phone) ||
    (props.mainEmailDisabled && type.value === ContactTypes.Email)
);

const setModalProps = (modalProps: IContactModalProps) => {
  if (!props.inModal) return;

  emit("setModalProps", modalProps);
};

const create = async () => {
  if (!(await v$.value.$validate())) return;

  isLoading.value = true;

  try {
    await form.value?.create();

    // После перехода на обновленную версию метода, где будет приходить UUID
    // uuid.value = "";

    emit("created");
  } catch (error) {
    emit("error", {
      message: "Не удалось выполнить добавление",
      error,
    });
  } finally {
    isLoading.value = false;
  }
};

const save = async () => {
  if (!(await v$.value.$validate())) return;

  isLoading.value = true;

  try {
    await form.value?.save();

    emit("saved");
  } catch (error) {
    emit("error", {
      message: "Не удалось выполнить сохранение",
      error,
    });
  } finally {
    isLoading.value = false;
  }
};

const archive = async () => {
  isLoading.value = true;

  try {
    await form.value?.archive();

    if (editableData.value) {
      editableData.value.archived = true;
    }

    emit("archived");
  } catch (error) {
    emit("error", {
      message: "Не удалось выполнить архивирование",
      error,
    });
  } finally {
    isLoading.value = false;
  }
};

const unarchive = async () => {
  isLoading.value = true;

  try {
    await form.value?.unarchive();

    if (editableData.value) {
      editableData.value.archived = false;
    }

    emit("unarchived");
  } catch (error) {
    emit("error", {
      message: "Не удалось выполнить разархивирование",
      error,
    });
  } finally {
    isLoading.value = false;
  }
};
</script>

<style scoped>
.contact-wrapper {
  position: relative;
}

.contact-wrapper:has(.or-loader) {
  min-height: 50px;
}
</style>
