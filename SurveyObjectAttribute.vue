<script lang="ts" setup>
import type { Options } from './SurveyObjectAttributeTemplate.vue'
import type { ObjectSchema, Schema } from '~/types/attributes'
import { Attribute, SCHEMA_TYPE } from '~/types/attributes'

defineProps<{
  storageKey: 'customAttributes' | 'prescribedAttributes'
  options: Options
}>()

const emit = defineEmits<{
  (e: 'change'): void
  (e: 'change:name'): void
  (e: 'update:template'): void
  (e: 'delete', schema: Schema, parent?: ObjectSchema): void
}>()

const { attributes, schema } = defineModels<{
  attributes: Attribute[]
  schema: Schema
}>()

const isObjectAttributesShow = ref(false)

function deleteAttribute(schema: Schema, parent?: ObjectSchema) {
  emit('delete', schema, parent)
}

function updateAttribute(schema: Schema, attribute: Attribute) {
  const idx = getAttribute(schema, { returnIndex: true })
  if (idx === -1)
    attributes.value.push(attribute)
  else attributes.value.splice(idx, 1, attribute)
}

function getAttribute(
  schema: Schema,
  options?: { returnIndex?: true, autoCreate?: Schema }
): number
function getAttribute(
  schema: Schema,
  options?: { returnIndex?: false, autoCreate?: Schema }
): Attribute | undefined
function getAttribute(
  schema: Schema,
  options?: { returnIndex?: boolean, autoCreate?: Schema },
): Attribute | number | undefined {
  const callback = (attr: Attribute) => attr.schemaUUID === schema.uuid
  let idx = attributes.value?.findIndex(callback)

  if (idx === -1 && options?.autoCreate)
    idx = attributes.value.push(new Attribute({ schemaUUID: schema.uuid })) - 1

  return options?.returnIndex ? idx : attributes.value[idx]
}
</script>

<template>
  <div class="passport-attributes-attribute" :class="schema.type">
    <SurveyObjectAttributeTemplate
      v-model:isHidden="isObjectAttributesShow"
      v-model:schema="schema"
      :attribute="getAttribute(schema, { autoCreate: schema })"
      :is-child="false"
      :options="options"
      @update:attribute="updateAttribute(schema, $event)"
      @delete="deleteAttribute(schema)"
      @change="$emit('change')"
    />
    <div
      v-if="!isObjectAttributesShow && schema.type === SCHEMA_TYPE.object"
      class="passport-attributes-attribute-children"
    >
      <div
        v-for="(childSchema, idx) in schema.properties"
        :key="childSchema.uuid"
        class="passport-attributes-attribute"
      >
        <SurveyObjectAttributeTemplate
          v-model:schema="schema.properties[idx]"
          v-model:isHidden="isObjectAttributesShow"
          :attribute="getAttribute(childSchema, { autoCreate: childSchema })"
          :is-child="true"
          :options="options"
          @update:attribute="updateAttribute(childSchema, $event)"
          @update:template="emit('update:template')"
          @delete="deleteAttribute(childSchema, schema as ObjectSchema)"
          @change="$emit('change')"
        />
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.passport-attributes {
  &::last-child {
    background-color: red !important;
    .passport-attributes-attribute-wrap {
      &:after {
        display: none;
      }
    }
  }
  .attributes-add {
    width: 160px;
  }
  &-tools {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  &-attribute {
    display: flex;
    flex-direction: column;
    width: 100%;
    position: relative;
    padding: 7px 0;

    &__input {
      padding: 5px 7px;
      position: relative;
      min-width: 170px;
      width: 170px;
      height: 100%;
      border-radius: 5px;
      background-color: #f1f1f1;
      color: black;
      white-space: nowrap;
      overflow: hidden;

      &[type='checkbox'] {
        filter: grayscale(1);
        cursor: pointer;
      }
    }
    @for $i from 1 through 100 {
      &:nth-child(#{$i}) {
        z-index: 100 - $i;
      }
    }

    &-left {
      display: flex;
      align-items: center;
      gap: 20px;
      .dx-button {
        width: 170px;
      }
      .dx-textbox {
        // width: 170px;
        height: 30px;
        border-radius: 5px !important;
      }
      .dropdown__item {
        font-weight: 400;
      }
    }

    &__key {
      font-weight: 600;
    }

    &-wrap {
      width: 100%;
      display: flex;
      justify-content: space-between;
      position: relative;
      padding: 0.25rem 0.5rem;
      container-type: inline-size;
      &::after {
        content: '';
        position: absolute;
        left: 0;
        right: 0;
        height: 1px;
        background-color: #f1f1f1;
        bottom: -7px;
      }
      &.isChild {
        position: relative;
        .attribute-name {
          width: 160px;
          min-width: 160px;
        }
        &::after {
          content: '';
          position: absolute;
          left: 0;
          width: 10px;
          height: 1px;
          background-color: #f1f1f1;
        }
      }
      @container (width < 800px) {
        .dx-button {
          .dx-button-text {
            display: none;
          }
          .dx-icon {
            margin-right: 0;
          }
          .dx-button-content {
            padding: 6px 6px 6px;
          }
        }
      }
      @container (width < 650px) {
        &.isChild {
          .attribute-name {
            width: 90px;
            min-width: 90px;
          }
        }
        .attributes-add.dx-button {
          width: 100px;
        }
      }
    }

    &-children {
      padding-left: 2px;
      padding-top: 5px;
      margin-left: 7px;
      margin-top: 13px;
      border-left: 2px solid #f1f1f1;
      display: flex;
      flex-direction: column;
      gap: 15px;
    }
  }
}

.attribute-visibility-toggle {
  position: absolute;
  left: -15px;
  transition: 0.3s transform;
  color: black;
  &.opened {
    transform: rotate(180deg);
  }
}
</style>
