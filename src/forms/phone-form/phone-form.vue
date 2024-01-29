<template>
  <div class="phone-form">
    <KeepAlive>
      <component
        :is="component"
        :creation="creation"
        :data="phone"
        :main-disabled="mainDisabled"
        :person-uuid="personUuid"
        :user-name="userName"
        @confirmed="confirmed"
        @error="emit('error', $event)"
        @update-confirm="updateConfirm"
        @update-data="updateData"
      >
        <template v-slot="{ phoneDisabled }">
          <div class="phone-input">
            <div class="phone-input__row">
              <ORInput
                ref="input"
                class="phone-input__input phone-input__input_stretched"
                :disabled="phoneDisabled || isInputDisabled"
                label="Номер телефона"
                label-as-placeholder
                :model-value="phone.phone"
                required
                type="phone-v2"
                @update:country-code="updateCountry"
                @update:model-value="updateData('phone', $event)"
              />

              <ORButton
                v-if="isButtonVisible"
                class="phone-input__button"
                :color-scheme="ButtonColorScheme.LIGHTGREEN"
              >
                <ORIcon name="phone" size="20" />
              </ORButton>
            </div>
          </div>
        </template>
      </component>
    </KeepAlive>
  </div>
</template>

<script lang="ts" setup>
import {
  computed,
  defineAsyncComponent,
  markRaw,
  nextTick,
  ref,
  watch,
  type Raw,
} from "vue";
import {
  ButtonColorScheme,
  ORButton,
  ORInput,
  ORLoader,
  TagColor,
  type ITag,
  type ORCountryCode,
} from "@flora/uikit";
import type { EditableContactPhone, IContactModalProps } from "@/types";
import { api } from "@/api";
import PhoneData from "./components/phone-data.vue";
import type { IErrorInfo } from "@flora/common-data";

const PhoneConfirm = defineAsyncComponent({
  loader: () => import("./components/phone-confirm.vue"),
  delay: 0,
  loadingComponent: ORLoader,
});

const props = defineProps<{
  personUuid: string;
  creation: boolean;
  mainDisabled?: boolean;
  phone: EditableContactPhone;
  userName: string;
  uuid: string | null;
}>();

const emit = defineEmits<{
  error: [payload: IErrorInfo];
  setModalProps: [props: IContactModalProps];
  updateData: [value: EditableContactPhone];
  updateFooterHidden: [value: boolean];
}>();

defineOptions({
  inheritAttrs: false,
});

const input = ref();

const isConfirming = ref(false);

const updateConfirm = (value: boolean) => {
  isConfirming.value = value;
  emit("updateFooterHidden", value);

  if (!value) {
    emit("setModalProps", {
      title: "Номер телефона",
    });
  } else {
    emit("setModalProps", {
      title: "Подтверждение номера",
      back() {
        updateConfirm(false);
      },
      backLeft: true,
      footerHidden: true,
    });
  }
};

const confirmed = () => {
  updateConfirm(false);
  updateData("verified", true);
};

const component = computed<Raw<typeof PhoneData | typeof PhoneConfirm>>(() =>
  isConfirming.value ? markRaw(PhoneConfirm) : markRaw(PhoneData)
);

const updateData = <Key extends keyof EditableContactPhone>(
  key: Key,
  value: EditableContactPhone[Key]
) => {
  emit("updateData", {
    ...props.phone,
    [key]: value,
  });
};

const isGettingCountry = ref(false);

const countryUuid = ref<string | null>(null);

const updateCountry = async (countryCode: ORCountryCode) => {
  isGettingCountry.value = true;

  try {
    const data = await api.getCountry(countryCode);
    if (!data?.length) return;

    countryUuid.value = data[0].uuid;
  } catch (error) {
    emit("error", {
      message: "Не удалось получить уникальный идентификатор страны",
      error,
    });
  } finally {
    isGettingCountry.value = false;

    await nextTick();

    const inputs = input.value?.$el.getElementsByTagName("input");
    if (inputs && inputs[1]) inputs[1].focus();
  }
};

const isInputDisabled = computed(
  () => props.phone.archived || props.phone.verified || isGettingCountry.value
);

const isButtonVisible = computed(
  () => !props.creation && !isConfirming.value && props.phone.verified && false
);

defineExpose({
  create() {
    if (!countryUuid.value) {
      throw Error("Отсутствует уникальный идентификатор страны");
    }

    return api.createPhone(props.personUuid, {
      telephones: {
        number: props.phone.phone,
        is_main: props.phone.main,
        comments: props.phone.comment,
        country_uuid: countryUuid.value,
      },
      user_d: props.userName,
      dml: "I",
    });
  },
  save() {
    if (!props.uuid) {
      throw Error("Отсутствует уникальный идентификатор номера телефона");
    }

    if (!countryUuid.value) {
      throw Error("Отсутствует уникальный идентификатор страны");
    }

    return api.updatePhone(props.uuid, {
      telephones: {
        number: props.phone.phone,
        is_main: props.phone.main,
        comments: props.phone.comment,
        country_uuid: countryUuid.value,
      },
      user_d: props.userName,
      dml: "U",
    });
  },
  archive() {
    if (!props.uuid) {
      throw Error("Отсутствует уникальный идентификатор номера телефона");
    }

    return api.archivePhone(props.uuid);
  },
  unarchive() {
    throw Error("Разархивирование недоступно");
  },
});

watch(
  [
    () => props.creation,
    () => props.phone.archived,
    () => props.phone.verified,
  ],
  ([creation, archived, verified]) => {
    if (creation) return;

    const tags: ITag[] = [];

    if (archived) {
      tags.push({
        text: "Архив",
        color: TagColor.GREY,
        icon: "expired",
      });
    }

    if (verified) {
      tags.push({
        text: "Подтвержден",
        color: archived ? TagColor.GREY : TagColor.GREEN,
        icon: "check",
      });
    }

    emit("setModalProps", {
      title: "Номер телефона",
      tags,
    });
  },
  {
    immediate: true,
  }
);
</script>

<style scoped>
.phone-form {
  position: relative;
}

.phone-form:has(.or-loader) {
  min-height: 50px;
}

.phone-input {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.phone-input__row {
  display: flex;
  gap: 8px;
}

.phone-input__input_stretched {
  flex: 1;
}

.phone-input__button {
  width: 68px;
  height: 68px;
  padding: 0;
}
</style>
