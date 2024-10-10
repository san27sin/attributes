<script setup lang="ts">
import DxButton from 'devextreme-vue/button'
import type { Options } from './ObjectAttributeTemplate.vue'
import type { Attribute, ObjectSchema, Schema } from '~/types/attributes'
import { SCHEMA_TYPE, SimpleSchema } from '~/types/attributes'
import type { BusinessObjectType } from '~/types/business-type'

const props = defineProps<{
  storageKey: 'customAttributes' | 'prescribedAttributes'
  options: Options
}>()

defineEmits<{
  (e: 'change'): void
  (e: 'update:template'): void
}>()

const { attributes, schemasStorage } = defineModels<{
  attributes: Attribute[]
  schemasStorage: BusinessObjectType
}>()

// const getLocaleValue = inject<getLocaleValue>('getLocaleValue')!;

// const schemas = computed({
//   get() {
//     return schemasStorage.value[props.storageKey] ?? []
//   },
//   set(value: Schema[]) {
//     schemasStorage.value[props.storageKey] = value
//   },
// })

onBeforeMount(() => {
  attributes.value ||= []
})

function createAttribute() {
  // const schemas = schemasStorage.value

  const prefix = 'Attribute-'
  const generatedCounters = schemasStorage.value
    .filter(schema => schema.name.includes(prefix))
    .map(schema => Number(schema.name.split(prefix)[1]))

  const maxCounter = generatedCounters?.length ? Math.max(...generatedCounters) : 0
  const attrName = `${prefix}${maxCounter + 1}`

  schemasStorage.value.push(new SimpleSchema({ name: attrName, type: SCHEMA_TYPE.number }))
  // attributes.push(new Attribute({ name: attrName, type: SCHEMA_TYPE.number }))
}

function deleteAttribute(schema: Schema, parent?: ObjectSchema) {
  const schemas = parent ? parent.properties : schemasStorage.value
  const parentName = parent?.name ?? [schemasStorage.value.id, props.storageKey]
  if (!schemas)
    throw new Error(`Нет хранилища схем ${parentName}`)

  const idx = schemas.findIndex(({ name }) => name === schema.name)
  if (idx === -1)
    throw new Error(`Cхемы ${schema.name} в схеме ${parentName} не существует`)

  schemas.splice(idx, 1)
}
</script>

<template>
  <div class="passport-attributes-scroll">
    <div
      :class="{ isEmpty: !schemasStorage || !attributes }"
      class="passport-attributes-body"
    >
      <div class="passport-attributes-header">
        <div class="passport-attributes__field">
          Имя
        </div>
        <div class="passport-attributes__field">
          Значение
        </div>
        <div v-if="options.isEditable" class="passport-attributes__field">
          Тип
        </div>
        <div v-if="options.isEditable" class="passport-attributes__field">
          Дополнительно
        </div>
        <div v-if="options.isEditable" class="passport-attributes__field" />
      </div>
      <div v-if="schemasStorage" class="passport-attributes-group passport-attributes-table">
        <SurveyObjectAttribute
          v-for="(schema, idx) in schemasStorage"
          :key="schema.uuid"
          v-model:schema="schemasStorage[idx]"
          :attributes="attributes"
          :storage-key="storageKey"
          :options="options"
          @update:template="$emit('update:template')"
          @delete="deleteAttribute"
          @change="$emit('change')"
        />
        <div class="passport-attribute-line" />
        <div class="passport-attribute-line second" />
      </div>
    </div>
    <DxButton
      v-if="options?.canAddAttributes"
      type="normal"
      text="Добавить атрибут"
      styling-mode="text"
      icon="plus"
      @click="createAttribute()"
    />
  </div>
</template>

<style lang="scss">
.passport-attributes {
  display: flex;
  flex-direction: column;
  // gap: 15px;
  padding-block: 0.5rem;

  &-header {
    display: flex;
    // gap: 20px;
    margin-top: 10px;
    container-type: inline-size;
  }

  &-full {
    display: none;
  }

  &-body {
    min-width: 513px;
    margin-bottom: 10px;
  }

  &-scroll {
    overflow-x: auto;
    margin-bottom: 10px;
  }

  &__field {
    width: 188px;
    font-weight: 500;
    padding: 5px 7px;
    border: 1px solid #f1f1f1;
    border-left: none;
    color: black;
    &:first-child {
      border-left: 1px solid #f1f1f1;
    }
    &:nth-child(2) {
      width: 191px;
    }
    &:nth-child(3) {
      width: 113px;
    }
    &:nth-child(4) {
      width: 105px;
    }
    &:last-child {
      flex: 1;
    }
  }

  &-group {
    // border-bottom: 1px solid #333;
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    // gap: 15px;
    &:last-child {
      border-bottom: none;
    }
    h3 {
      font-size: 16px;
      font-weight: 500;
    }
  }

  &-table {
    // padding-top: 7px;
    border: 1px solid #f1f1f1;
    position: relative;
    padding-bottom: 10px;
    margin-bottom: 10px;
    container-type: inline-size;

    &::after,
    &::before,
    .passport-attribute-line {
      content: '';
      position: absolute;
      top: 0;
      bottom: 0;
      width: 1px;
      background-color: #f1f1f1;
    }
    &::after {
      left: calc(25% - 11px);
    }
    &::before {
      left: calc(50% - 21px);
    }
    .passport-attribute-line {
      left: calc(75% - 30px);
      &.second {
        left: 595px;
      }
    }
  }

  &-businessType {
    width: 100%;
  }

  &-label {
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
  }

  &__title {
    font-weight: 500 !important;
    font-size: 18px !important;
    color: black;
    display: flex;
    gap: 20px;
  }

  .dx-button {
    background-color: #f1f1f1;
    color: black;
    .dx-button-content {
      padding: 6px 10px 6px;
    }
    .dx-icon {
      color: black;
    }
  }

  &-action {
    color: black;
    cursor: pointer;
    padding: 4px;
    border-radius: 5px;
    background-color: #f1f1f1;
    svg {
      transform: rotate(180deg);
      transition: 0.3s transform;
    }
    &.opened {
      svg {
        transform: rotate(0deg);
      }
    }
  }
}

// @container (width < 650px) {
// }
</style>
