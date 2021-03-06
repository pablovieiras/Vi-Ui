<template>
  <vi-wrapper
    mini-spacing
    vertical
    justify-content="flex-start"
    class="ViComponent ViInput"
    :class="{ 'ViInput--elevated': isOpen }"
    :style="{ width: componentWidth }"
    :value="false"
  >
    <vi-input-label v-bind="{ for: id, label, instruction }"/>
    <multiselect
      class="ViInput__Multiselect"
      :class="{
        'ViInput__Multiselect--multiple': multiple,
        'ViInput__Multiselect--pills': pill,
        'ViInput__Multiselect--checkbox': checkbox,
        'ViInput__Multiselect--select-all-hidden': hideSelectAll,
      }"
      @open="openEvent"
      @tag="tagEvent"
      @close="closeEvent"
      @input="inputEvent"
      @change="changeEvent"
      @select="selectEvent"
      @remove="removeEvent"
      @search-change="searchEvent"
      v-bind="newProps"
      :tabindex="0"

    >
      <template
        slot="clear"
        slot-scope="{ search }"
      >
        <slot
          name="clear"
          :search="search"
        >
          <template v-if="checkbox && !hideSelectAll">
            <div
              class="ViInput__CheckAll"
              :style="{
                width: optionsWidthString,
              }"
            >
              <span
                tabindex="1"
                class="multiselect__checkoption"
                :class=" {
                  'multiselect__checkoption--selected': isAllChecked,
                  'multiselect__checkoption--inderteminate': isSomeCheckedButNotAll,
                }"
                @keydown.enter.prevent="selectAll"
                @keydown.space.prevent="selectAll"
                @mousedown.prevent="selectAll"
              >
                <span class="ViInput__MultiselectCheckbox">{{ checkAllLabelComp }}</span>
                <span class="ViInput__Total">({{ totalValueLabel }})</span>
              </span>
            </div>
          </template>
        </slot>
      </template>
      <template
        slot="caret"
        slot-scope="{ toggle }"
      >
        <slot
          name="caret"
          :toggle="toggle"
        >
          <div
            v-show="!loading"
            tabindex="0"
            class="multiselect__select"
            @keydown.enter.prevent.stop="toggle"
            @keydown.space.prevent.stop="toggle"
            @mousedown.prevent.stop="toggle"
          />
        </slot>
      </template>
      <template
        slot="option"
        slot-scope="{ option }"
      >
        <slot
          name="option"
          v-bind="{ option }"
        >
          <span
            :class="{ 'ViInput__MultiselectCheckbox': checkbox }"
            :title="getOptionLabel(option)"
          >
            {{ getOptionLabel(option) }}
          </span>
        </slot>
      </template>

      <template
        slot="tag"
        slot-scope="{ option, search, remove }"
      >
        <slot
          name="tag"
          v-bind="{ option, search, remove }"
        >
          <template v-if="pill">
            <span class="multiselect__tag">
              <span>{{ getOptionLabel(option) }}</span>
              <i
                aria-hidden="true"
                tabindex="1"
                @keydown.enter.prevent="removeElement(option)"
                @mousedown.prevent="removeElement(option)"
                class="multiselect__tag-icon"
              />
            </span>
          </template>
          <template
            @close="closeEvent"
            v-else-if="getOptionLabel(value[0]) === getOptionLabel(option)
            && (!isOpen && searchable || !searchable)"
          >
            {{ totalValueLabel }}
          </template>
        </slot>
      </template>

      <template
        slot="singleLabel"
        slot-scope="scope"
      >
        <slot
          name="singleLabel"
          slot-scope="scope"
        >
          {{ getOptionLabel(scope.option) }}
        </slot>
      </template>
      <template slot="beforeList">
        <slot name="beforeList" />
      </template>
      <template slot="afterList">
        <slot name="afterList" />
      </template>
      <template
        slot="limit"
        v-if="internalValue && internalValue.length > limit"
      >
        <slot name="limit">
          <strong
            class="multiselect__strong"
            v-text="limitText(internalValue.length - limit)"
          />
        </slot>
      </template>
      <template slot="noResult">
        <slot name="noResult">Nenhum resultado disponível</slot>
      </template>
      <template
        slot="maxElements"
        v-if="multiple && max === internalValue.length"
      >
        <slot name="maxElements">
          Maximo de {{ max }} opções selecionadas.
          Remova uma opção selecionada para escolher outra opção.
        </slot>
      </template>
      <template slot="placeholder">
        <slot name="placeholder">
          {{ placeholder }}
        </slot>
      </template>
    </multiselect>

  </vi-wrapper>
</template>


<script>
import Multiselect from 'vue-multiselect';
import ViWrapper from './Wrapper.vue';
import ViInputLabel from '../helperComponents/InputLabel.vue';
import { scaleMixin, widthMixin } from '../mixins/sizes';
import inputMixin from '../mixins/input';
/**
 * ###### Além das props descritas aqui, usamos a props e eventos do **Vue-Multiselect**.
 * [Documentação completa do Vue-Multiselect](https://vue-multiselect.js.org/#sub-props)
 */

export default {
  extends: Multiselect,
  name: 'ViSelect',
  components: {
    Multiselect,
    ViWrapper,
    ViInputLabel,
  },
  mixins: [scaleMixin, widthMixin, inputMixin],
  props: {
    /**
     * Substitui label do Vue-Multisect.
     */
    optionLabel: {
      type: String,
      default: null,
    },
    /**
     * Label do campo do Vi-Ui.
     */
    label: {
      type: String,
      default: null,
    },
    /**
     * Visualização de tags, comportamento original do Vue-Multiselect.
     */
    pill: {
      type: Boolean,
      default: false,
    },
    /**
     * Exibe um checkbox para cada opção e outro para marcar/desmarcar todos.
     * Também força o valor de `multiple` para `true`.
     */
    checkbox: {
      type: Boolean,
      default: false,
    },
    /**
     * Label da opção "Marcar tudo" quando `checkbox` igual `true`.
     */
    checkAllLabel: {
      type: String,
      default: 'Marcar tudo',
    },
    /**
     * Label da opção "Marcar todas as opções da busca" quando `checkbox` igual `true`.
     */
    checkSelectionLabel: {
      type: String,
      default: 'Marcar todas as opções da busca',
    },
    /**
     * Label da opção "Desmarcar tudo" quando `checkbox` igual `true`.
     */
    uncheckAllLabel: {
      type: String,
      default: 'Desmarcar tudo',
    },
    /**
     * Label da opção "Desmarcar todas as opções da busca" quando `checkbox` igual `true`.
     */
    uncheckSelectionLabel: {
      type: String,
      default: 'Desmarcar todas as opções da busca',
    },
    /**
     * Label dentro so select quando fechado, existem opções selecionadas e `pill` igual `false`.
     * ##NUMBER## é substituído pelo total de opções selecionadas.
     */
    selectClosedLabel: {
      type: String,
      default: '##NUMBER## opções marcadas',
    },
    /**
     * String que aparece quando o mouse ou o foco está em uma opção.
     */
    selectLabel: {
      type: String,
      default: '',
    },
    /**
     * String que aparece em uma opção selecionada.
     */
    selectedLabel: {
      type: String,
      default: 'Selecionado',
    },
    /**
     * String que aparece quando o mouse ou o foco está em uma opção selecionada.
     */
    deselectLabel: {
      type: String,
      default: 'Selecionado',
    },
    /**
     * Placeholder do campo.
     */
    placeholder: {
      type: String,
      default: 'Selecione uma opção',
    },
    /**
     * Valor do campo.
     * @model
     */
    value: {
      type: [String, Number, Object, Array],
      default: null,
    },
    /**
     * Se for true, oculta a opção "Marcar todos"
     */
    hideSelectAll: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      searchValue: '',
      isOpen: false,
      optionsWidth: 'calc(100% - 2px)',
    };
  },
  computed: {
    optionsWidthString() {
      return Number.isNaN(this.optionsWidth - 0)
        ? this.optionsWidth
        : `${this.optionsWidth}px`;
    },
    totalValueLabel() {
      return this.selectClosedLabel.replace('##NUMBER##', this.totalValue);
    },
    newProps() {
      const props = Object.assign({}, this.$props);
      delete props.label;
      if (this.optionLabel !== null) props.label = this.optionLabel;
      if (this.placeholder === null) delete props.placeholder;
      if (this.checkbox) {
        props.selectedLabel = '';
        props.deselectLabel = '';
        props.clearOnSelect = false;
        props.multiple = true;
      }
      return props;
    },
    isAllChecked() {
      if (!this.value || !this.filteredOptions) return false;
      return this.filteredOptions
        .every(option => this.containValue(option, this.value));
    },
    isSomeCheckedButNotAll() {
      if (!this.value || !this.filteredOptions || this.isAllChecked) return false;
      return this.filteredOptions
        .some(option => this.containValue(option, this.value));
    },
    checkAllLabelComp() {
      if (this.searchValue) {
        return this.isAllChecked
          ? this.uncheckSelectionLabel
          : this.checkSelectionLabel;
      }
      return this.isAllChecked
        ? this.uncheckAllLabel
        : this.checkAllLabel;
    },
    totalValue() {
      if (!this.value) return 0;
      return this.value.length;
    },
    filteredOptions() {
      if (!this.searchValue) return this.options;
      const pattern = new RegExp(this.searchValue, 'i');
      return this.options
        .filter(option => pattern.test(this.customLabel(option, this.optionLabel)));
    },
  },
  methods: {
    containValue(optionToCompare, obj) {
      if (typeof optionToCompare !== 'object') return obj.includes(optionToCompare);
      const labelToCompare = this.getOptionLabel(optionToCompare);
      return obj.some(optionCompared => this.getOptionLabel(optionCompared) === labelToCompare);
    },
    setWidthOptions() {
      if (!this.isOpen) return;
      this.optionsWidth = this.$el
        .getElementsByClassName('multiselect__element')[0]
        .clientWidth;
    },
    selectAll() {
      let newValue = this.value || [];

      if (this.isAllChecked) {
        newValue = newValue
          .filter(option => !this.containValue(option, this.filteredOptions));
      } else {
        const uniqueValues = [...newValue];
        this.filteredOptions.map((option) => {
          if (!this.containValue(option, uniqueValues)) {
            uniqueValues.push(option);
          }
          return null;
        });

        newValue = uniqueValues
          .filter(option => this.containValue(option, this.filteredOptions));
      }

      this.$emit('input', newValue);
    },
    getOptionLabel(option) {
      if (!option) return '';
      if (option.isTag) return option.label;
      if (option.$isLabel) return option.$groupLabel;

      const label = this.customLabel(option, this.optionLabel);
      if (!label) return '';
      return label;
    },
    inputEvent(value, id) {
      this.$emit('input', value, id);
    },
    changeEvent(value, id) {
      this.$emit('change', value, id);
    },
    selectEvent(value, id) {
      this.$emit('select', value, id);
    },
    removeEvent(value, id) {
      this.$emit('remove', value, id);
    },
    tagEvent(value, id) {
      this.$emit('tag', value, id);
    },
    searchEvent(value, id) {
      this.searchValue = value;
      this.$emit('search-change', value, id);
    },
    closeEvent(value, id) {
      this.isOpen = false;
      this.$emit('close', value, id);
    },
    openEvent(id) {
      setTimeout(this.setWidthOptions, 80);
      this.isOpen = true;
      this.$emit('open', id);
    },
  },
};
</script>

<style lang="stylus">
@import '../../node_modules/vue-multiselect/dist/vue-multiselect.min.css'
@import '../themes/main'
@import '../themes/input'

.ViComponent.ViInput
  .ViInput__Multiselect
    .multiselect__tags
      border-radius 0

    &.multiselect--above
      .multiselect__content-wrapper
        border-bottom-width 0

    &.multiselect--active:not(.multiselect--above)
      .multiselect__tags
        border-bottom-color rgba($border-color-main, 0.5)

      &.multiselect--above
        .multiselect__tags
          border-top-color rgba($border-color-main, 0.5)

    &--multiple
      .multiselect__tags
        &-wrap
          float:left

        .multiselect__tag
          background $primary
          border-radius 30px

          &-icon
            border-radius 30px
            &:after
              color rgba($text-color-main, 0.5)

            &:focus
            &:hover
              background darken($primary, 30%)

              &:after
                color: $light

    &--multiple
      &--pill
        .multiselect__tags
          padding 0.55em 0.55em 0

    .multiselect__input
      padding-left 0
      margin-top -1px

    .multiselect__tags
      border 1px solid $border-color-main
      border-bottom-width 2px
      min-height 40px
      padding 0.7em 1em 0
      outline none

      .multiselect__single
        font-size unset
        padding 0 15px 0 0
        @extend .text-truncate

    .multiselect__select
      z-index 3
      &:focus
        box-shadow 0 0 0 1px $border-color-main-focus
        will-change box-shadow
        transition all 0.04s ease-in-out
        outline none
        &:before
          border-color $border-color-main-focus transparent transparent

    .multiselect__content
      width 100%

    .multiselect__content-wrapper
      border-color $border-color-main
      border-bottom-width 2px
      border-radius 0
      z-index 1

    .multiselect__checkoption
      display block
      padding 12px
      min-height 40px
      line-height 16px
      text-decoration none
      text-transform none
      vertical-align middle
      position relative
      cursor pointer
      white-space nowrap

    .multiselect__checkoption
    .multiselect__option
      border-bottom 1px solid rgba($border-color-main, 0.5)
      @extend .text-truncate

      &:after
        font-weight 700
        color rgba($text-color-main, 0.4)

      .ViInput__MultiselectCheckbox
        position relative
        padding 0 0 0 2.2em
        z-index 0

        &:after
        &:before
          background $default
          border-radius 0.3em
          content ''
          height 1.5em
          left 0
          position absolute
          top -2px
          width 1.5em
          transition all 0.04s ease-out

        &:after
          background transparent
          border 0.27em solid $primary
          border-right-width 0.5em
          border-left 0
          border-top 0
          border-radius 0
          opacity 0
          transform rotate(90deg) scale(0.4, 0.3) translate(-0.5em)

      &--selected
        background none
        .ViInput__MultiselectCheckbox
          &:before
            background $primary
          &:after
            border-color $light
            opacity 1
            transform rotate(40deg) scale(0.3, 0.6) translate(-0.2em, -0.15em)
            transition all 0.04s ease-out, opacity 0.1s ease-out,
            transform 0.25s cubic-bezier(0.18, 0.89, 0.32, 1.28)
            will-change transform, opacity

      &--inderteminate
        .ViInput__MultiselectCheckbox
          &:after
            border-bottom-width 0
            opacity 1
            transition all 0.04s ease-out, opacity 0.1s ease-out,
            transform 0.25s cubic-bezier(0.18, 0.89, 0.32, 1.28)
            will-change transform, opacity

      &--highlight
      &:focus:not(.multiselect__checkoption)
      &:hover:not(.multiselect__checkoption)
        color $border-color-main-focus
        box-shadow 0 -1px 0 rgba($border-color-main,0.5)
        background rgba($border-color-main,0.4)
        position relative
        will-change box-shadow
        z-index 1

        &:after
          background inherit
          color rgba($text-color-main, 0.3)

        &.multiselect__option--selected
          color $success

    .ViInput__CheckAll
      position relative
      z-index 2

      .multiselect__checkoption
        border none
        border-bottom 1px solid rgba($border-color-main, 0.5)
        display flex
        font-weight 300
        height 0
        justify-content space-between
        left 1px
        min-height 0
        padding 0 12px
        position absolute
        top 39px
        width 100%
        opacity 0
        overflow hidden
        transition all 0.06s ease-out
        background $light

        .ViInput__MultiselectCheckbox
          &:before
          &:after
            top -5px

      .ViInput__Total
        font-size 80%

    &.multiselect--active
      .ViInput__CheckAll
        .multiselect__checkoption
          height 40px
          min-height 40px
          opacity 1
          padding 12px

    &.multiselect--above
      .ViInput__CheckAll
        .multiselect__checkoption
          border-top 1px solid rgba($border-color-main, 0.5)
          top -39px

    &--checkbox
      .multiselect__single
        display none

      .multiselect__content
        padding 40px 0 0

      &.multiselect--above
        .multiselect__content
          padding 0 0 38px

    &--select-all-hidden
      .multiselect__content
        padding 0
</style>


<docs>
### Select básico

```vue
<template>
  <vi-wrapper vertical>
    <vi-select
      checkbox
      label="Relationship Status"
      v-model="value"
      :options="relationshipStatusOptions"
      :close-on-select="false"
    />
    <pre><code>{{ dataForm }}</code></pre>
  </vi-wrapper>
</template>
<script>
export default {
  data() {
    return {
      value: null,
    };
  },
  computed: {
    relationshipStatusOptions() {
      return [
        'single',
        'in a relationship',
        'engaged',
        'married',
        'its complicated',
        'in an open relationship',
        'widowed',
        'separated',
        'divorced',
      ];
    },
    dataForm() {
      return JSON.stringify(this._data, null, 2);
    },
  },
};
</script>
```

### Select com coleção de objetos

```vue
<template>
  <vi-wrapper vertical>
    <vi-select
      checkbox
      label="Estado"
      v-model="value"
      :options="estados"
      :close-on-select="false"
      option-label="name"
      track-by="id"
    />
    <pre><code>{{ dataForm }}</code></pre>
  </vi-wrapper>
</template>
<script>
export default {
  data() {
    return {
      value: [{ id: 'MG' }]
    };
  },
  computed: {
    estados() {
      return [
        { id: 'MG', name: 'Minas Gerais' },
        { id: 'SP', name: 'São Paulo' },
        { id: 'RJ', name: 'Rio de Janeiro' },
      ];
    },
    dataForm() {
      return JSON.stringify(this._data, null, 2);
    },
  },
};
</script>
```

</docs>
