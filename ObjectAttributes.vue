<script setup lang="ts">
import InlineSvg from 'vue-inline-svg'
import type { Options } from './ObjectAttributeTemplate.vue'
import type { Attribute } from '~/types/attributes'
import type { BusinessObjectType } from '~/types/business-type'

defineProps<{
  storageKey: 'customAttributes' | 'prescribedAttributes'
  options: Options
}>()

defineEmits<{
  (e: 'openDetailedInfo'): void
  (e: 'change'): void
}>()

const { schemasStorage, attributes } = defineModels<{
  schemasStorage: BusinessObjectType[]
  attributes: Attribute[]
}>()

const isAttributesVisible = ref(true)
const attributesInfo = ref({
  ru: `<p>
        Здесь вы можете настроить переменные, которые будут использоваться в шаблонах.
        Атрибут можно вставить в шаблон таким образом: {{ Attribute-1 }}, или {{ Attribute-1.Property-1 }}, если это объект.
        Атрибуты бывают 5 типов:
        <ol>
          <li>1. String - обычное текстовое значение</li>
          <li>2. Number - обычное числовое значение</li>
          <li>3. Translate - строка с переводом на другую языковую версию шаблона</li>
          <li>4. Dictionary - справочник со списком выбираемых значений (выводятся через запятую)</li>
          <li>5. Group - группа из атрибутов</li>
        </ol>
      </p>`,
  en: `<p>
        Here you can configure the variables that will be used in the templates.
         The attribute can be inserted into the template like this: {{ Attribute-1 }}, or {{ Attribute-1.Property-1 }} if it is an object.
         There are 5 types of attributes:
        <ol>
          <li>1. String - regular text value</li>
          <li>2. Number - regular number value</li>
          <li>3. Translate - string with translation for another language template version</li>
          <li>4. Dictionary - directory with a list of selectable values (displayed by commas)</li>
          <li>5. Group - group of attributes</li>
        </ol>
      </p>`,
})
</script>

<template>
  <div class="passport-attributes">
    <label
      class="passport-attributes-label"
      @click="isAttributesVisible = !isAttributesVisible"
    >
      <h2 class="passport-attributes__title">
        Атрибуты задачи:
        <InfoWithTooltip :content="attributesInfo" />
      </h2>
      <div class="passport-attributes-tools">
        <!-- <div class="passport-attributes-full">
          <InlineSvg
            src="/img/icons/book-open.svg"
            width="15"
            height="15"
            :title="$t('global.open')"
            @click.stop="$emit('openDetailedInfo')"
          />
        </div> -->
        <div
          class="passport-attributes-action"
          :class="{ opened: !isAttributesVisible }"
        >
          <InlineSvg
            src="/icons/chevron-down.svg"
            class="attributes-visibility-toggle"
            width="24"
            height="24"
            :title="isAttributesVisible ? 'Скрыть' : 'Показать'"
          />
        </div>
      </div>
    </label>
    <div
      v-if="isAttributesVisible"
      class="passport-attributes-block"
    >
      <!-- <div
        v-for="(schemaStorage, index) in schemasStorages"
        :key="schemaStorage.id"
        class="passport-attributes-group"
      > -->
      <SurveyObjectAttributesGroup
        v-model:attributes="attributes"
        v-model:schemasStorage="schemasStorage"
        :storage-key="storageKey"
        :options="options"
        @change="$emit('change')"
      />
      <!-- </div> -->
    </div>
  </div>
</template>
