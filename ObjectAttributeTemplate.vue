<script lang="ts" setup>
import DxButton from 'devextreme-vue/button'
import DxDropDownBox from 'devextreme-vue/drop-down-box'
import {
  DxDataGrid,
  DxEditing,
  DxPaging,
  DxScrolling,
  DxSelection,
} from 'devextreme-vue/data-grid'

import InlineSvg from 'vue-inline-svg'
import Select from '~/components/ui/Select.vue'
import LocaleSwitcher from '~/components/ui/LocaleSwitcher.vue'

import type { Attribute, Schema } from '~/types/attributes'
import {
  DICTIONARY_MODE,
  DictionarySchema,
  ObjectSchema,
  SCHEMA_TYPE,
  SCHEMA_TYPE_NAME,
  SchemaFactory,
  SimpleSchema,
} from '~/types/attributes'
import { TextTranslate } from '~/ts/common/TextTranslate'
import { capitalizeFirstLetter } from '~/ts/common/StringTools'
import { LOCALES } from '~/types/environment'

export interface Options {
  isEditable: boolean
}

const props = withDefaults(
  defineProps<{
    isChild: boolean
    options?: Options
  }>(),
  {
    options: () => ({
      isEditable: true,
    }),
  },
)

const emit = defineEmits<{
  (e: 'change'): void
  (e: 'delete'): void
}>()

const { attribute, schema, isHidden } = defineModels<{
  attribute: Attribute
  isHidden: boolean
  schema: Schema
}>()

// const { t } = useI18n()

const propertiesDS = computed({
  get() {
    if (schema.value.type !== SCHEMA_TYPE.dictionary)
      return []

    return schema.value.properties.map(name => ({ name }))
  },
  set: updateDictionaryProperties,
})

const dictionaryModes = Object.values(DICTIONARY_MODE).map((mode, idx) => ({
  id: idx + 1,
  title: capitalizeFirstLetter(mode),
  value: mode,
}))

const schemaTypes = Object.values(SCHEMA_TYPE)
  .map((type, idx) => {
    if (props.isChild && type === SCHEMA_TYPE.object)
      return undefined

    return {
      id: idx + 1,
      title: capitalizeFirstLetter(SCHEMA_TYPE_NAME[type]),
      value: type,
    }
  })
  .filter(Boolean)

const localizedValue = computed(() => {
  const attr = attribute.value
  const v = TextTranslate.is(attr.value)
    ? attr.value
    : Object.fromEntries(Object.values(LOCALES).map(locale => [locale, attr.value]))

  return new TextTranslate(v)
})

const valueWithoutLocalization = computed(() => {
  return schema.value.type !== SCHEMA_TYPE.locale
})

const schemaType = computed({
  get() {
    const defaultValue = Object.values(schemaTypes).at(0)?.id
    if (!defaultValue)
      throw new Error('Ошибка получения значения по-умолчанию')

    return schemaTypes.find(el => el.value === schema.value.type)?.id ?? defaultValue
  },
  set(typeId: number) {
    const type = schemaTypes.find(type => type.id === typeId)?.value
    if (!type)
      throw new Error(`Режима справочника ${type} не существует`)

    const newSchema = SchemaFactory.Create({
      ...schema.value,
      type,
      properties: [],
      mode: DICTIONARY_MODE.single,
    })

    // Дефолтное значение справочника. Без него DX ломается
    if (newSchema instanceof DictionarySchema)
      newSchema.properties.push('Значение')

    schema.value = newSchema

    attribute.value.value = undefined
  },
})

const dictionaryMode = computed({
  get() {
    const defaultValue = Object.values(dictionaryModes).at(0)?.id
    if (!defaultValue)
      throw new Error('Ошибка получения значения по-умолчанию')

    if (!(schema.value instanceof DictionarySchema))
      return defaultValue

    const mode = schema.value.mode

    return dictionaryModes.find(el => el.value === mode)?.id ?? defaultValue
  },
  set(modeId: number) {
    if (!(schema.value instanceof DictionarySchema))
      throw new Error(`Атрибут ${schema.value.name} не является справочником`)

    const mode = dictionaryModes.find(mode => mode.id === modeId)?.value
    if (!mode)
      throw new Error(`Режима справочника ${mode} не существует`)

    schema.value.mode = mode
  },
})

function handleInput(event: Event, locale: LOCALES) {
  const target = event.target as HTMLInputElement
  const inputValue = target.type === 'checkbox' ? target.checked : target.value
  if (schema.value.type === SCHEMA_TYPE.locale) {
    const attr = attribute.value
    if (TextTranslate.is(attr.value))
      attr.value[locale] = inputValue
    else attr.value = new TextTranslate({ [locale]: inputValue })
  }
  else {
    attribute.value.value = inputValue
  }

  emit('change')
}

function handlePropertiesChange() {
  // синхронизация изменений DataSource (DX не триггерит computed setter)
  propertiesDS.value = propertiesDS.value.slice()
}

function updateDictionaryProperties(value: Array<{ name: string | number }>) {
  if (schema.value.type !== SCHEMA_TYPE.dictionary)
    return

  schema.value.properties = value.map(({ name }) => name)
}

function getAttributeInputType(type: string) {
  switch (type) {
    case SCHEMA_TYPE.number:
      return 'number'

    case SCHEMA_TYPE.locale:
      return 'text'

    case SCHEMA_TYPE.date:
      return 'date'

    case SCHEMA_TYPE.bool:
      return 'checkbox'

    default:
      return 'text'
  }
}

function onGridSelectionChanged(data: { selectedRowKeys: Array<{ name: string }> }) {
  attribute.value.value = data.selectedRowKeys.map(i => i.name)
  emit('change')
}

function changeVisibilityObjectAttributes() {
  isHidden.value = !isHidden.value
}

function addChild() {
  if (!(schema.value instanceof ObjectSchema))
    throw new Error(`Добавление потомка в атрибут ${schema.value.name} запрещено`)

  const prefix = 'Property-'
  const generatedCounters = Object.keys(schema.value.properties)
    .filter(key => key.includes(prefix))
    .map(name => Number(name.split(prefix)[1]))

  const maxCounter = generatedCounters?.length ? Math.max(...generatedCounters) : 0
  const attrName = `${prefix}${maxCounter + 1}`

  schema.value.properties.push(new SimpleSchema({ name: attrName, type: SCHEMA_TYPE.number }))
  isHidden.value = false
}
</script>

<template>
  <div class="passport-attributes-attribute-wrap" :class="{ isChild }">
    <div class="passport-attributes-attribute-left">
      <InlineSvg
        v-if="schema.type === SCHEMA_TYPE.object"
        src="/icons/chevron-down.svg"
        class="attribute-visibility-toggle"
        :class="{ opened: !isHidden }"
        :title="isHidden ? $t('global.open') : $t('global.hide')"
        @click="changeVisibilityObjectAttributes()"
      />

      <input
        v-if="options.isEditable"
        v-model="schema.name"
        :title="schema.name"
        class="attribute-name passport-attributes-attribute__input"
      >
      <p v-else class="passport-attributes-attribute__input attribute-name" :title="schema.name">
        {{ schema.name }}
      </p>

      <p
        v-if="!options.isEditable && schema.type === SCHEMA_TYPE.object"
        class="passport-attributes-attribute__input attribute-name"
        :title="schema.name"
        hidden
      >
        {{
          `${$t("global.attributes")} ${
            schema.properties?.length ? `: ${schema.properties.length}` : ` ${$t("global.no")}`
          }`
        }}
      </p>
      <LocaleSwitcher
        v-if="schema.type !== SCHEMA_TYPE.object"
        top="-17px"
        right="5px"
        :hide="valueWithoutLocalization"
      >
        <template #default="{ locale }">
          <input
            v-if="schema.type !== SCHEMA_TYPE.dictionary"
            :value="localizedValue[locale]"
            :checked="Boolean(localizedValue[locale])"
            :type="getAttributeInputType(schema.type)"
            class="passport-attributes-attribute__input"
            @input="handleInput($event, locale)"
          >
          <DxDropDownBox
            v-else
            :value="attribute.value"
            :search-enabled="true"
            :data-source="schema.properties"
            :defer-rendering="false"
            :show-clear-button="true"
            :drop-down-options="{ width: 400 }"
            placeholder="Выберите значение"
          >
            <template #content>
              <DxDataGrid
                :data-source="propertiesDS"
                :hover-state-enabled="true"
                :column-auto-width="true"
                :allow-column-resizing="true"
                :use-icons="true"
                :show-column-headers="false"
                height="100%"
                class="attribute-dropdown"
                @saved="handlePropertiesChange"
                @selection-changed="onGridSelectionChanged($event)"
              >
                <DxSelection :mode="schema.mode" />
                <DxEditing
                  :allow-updating="options.isEditable"
                  :allow-adding="options.isEditable"
                  :allow-deleting="options.isEditable"
                  :confirm-delete="false"
                  :use-icons="true"
                  mode="row"
                />
                <DxPaging :enabled="true" :page-size="10" />
                <DxScrolling mode="virtual" />
              </DxDataGrid>
            </template>
          </DxDropDownBox>
        </template>
      </LocaleSwitcher>

      <DxButton
        v-if="options.isEditable && schema.type === SCHEMA_TYPE.object"
        type="normal"
        :text="$t('global.add')"
        class="attributes-add"
        styling-mode="text"
        icon="plus"
        @click="addChild"
      />
      <Select
        v-if="options.isEditable"
        v-model="schemaType"
        :items="schemaTypes"
        :disabled="!options.isEditable"
        class="dropdown_grey"
      />

      <Select
        v-if="options.isEditable && schema.type === SCHEMA_TYPE.dictionary"
        v-model="dictionaryMode"
        :items="dictionaryModes"
        :disabled="!options.isEditable"
        class="dropdown_grey"
      />
    </div>
    <DxButton
      v-if="options.isEditable"
      type="normal"
      :text="$t('global.delete')"
      styling-mode="text"
      icon="trash"
      @click="$emit('delete')"
    />
  </div>
</template>
