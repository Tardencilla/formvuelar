<template>
  <div :class="{'fvl-has-error' : $parent.hasErrors(name)}" class="fvl-color-picker-wrapper">
    <label
      v-if="label"
      :class="labelClass"
      :for="name"
      class="fvl-color-picker-label"
      v-html="label"
    />
    <on-click-outside @do="close()">
      <div class="fvl-color-picker-group">
        <slot name="prefix"></slot>
        <div class="fvl-color-picker-container">
          <input
            :id="id"
            ref="colorinput"
            :value="value"
            :name="name"
            :class="fieldClass"
            :required="required"
            :disabled="disabled"
            :readonly="readonly"
            type="text"
            class="fvl-color-picker"
            :pattern="pattern"
            @keyup.space="toggle()"
          />
          <div ref="colorpicker" class="fvl-color-preview" @click="toggle()">
            <span
              :style="{'background': value}"
              class="inline-block rounded-full border border-white h-4 w-4"
            ></span>
          </div>
        </div>
        <slot name="suffix"></slot>
        <div v-show="isOpen" ref="picker" class="fvl-color-picker-dropdown">
          <chrome-picker
            ref="picker"
            :disable-alpha="format == 'hex'"
            :value="value"
            disable-fields
            @input="updateValue"
          ></chrome-picker>
        </div>
      </div>
      <slot name="hint" />
      <slot :errors="$parent.getErrors(name)" name="errors">
        <validation-errors :errors="$parent.getErrors(name)" />
      </slot>
    </on-click-outside>
  </div>
</template>

<script>
  import { Chrome } from 'vue-color'
  import Popper from 'popper.js'
  import ValidationErrors from './FvlErrors.vue'
  import OnClickOutside from './utilities/OnClickOutside.vue'
  export default {
    components: {
      OnClickOutside,
      ValidationErrors,
      'chrome-picker': Chrome
    },
    props: {
      label: {
        type: String,
        required: false,
        default: null
      },
      name: {
        type: String,
        required: true
      },
      id: {
        type: String,
        default: null
      },
      value: {
        validator: prop => typeof prop === 'string' || prop === null,
        default: '0'
      },
      format: {
        type: String,
        validator: function(value) {
          // The value must match one of these strings
          return ['hex', 'hex8', 'hsl', 'hsv', 'rgba'].indexOf(value) !== -1
        },
        default: 'hex'
      },
      placeholder: {
        type: String,
        required: false,
        default: null
      },
      fieldClass: {
        type: String,
        required: false,
        default: null
      },
      labelClass: {
        type: String,
        required: false,
        default: null
      },
      readonly: {
        type: Boolean,
        required: false,
        default: false
      },
      required: {
        type: Boolean,
        required: false,
        default: false
      },
      disabled: {
        type: Boolean,
        required: false,
        default: false
      },
    },
    data() {
      return {
        isOpen: false,
        patterns: {
          'hex': `[#]([a-fA-F\\d]{6}|[a-fA-F\\d]{3})`,
          'hex8': `[#]([a-fA-F\\d]{8}`,
          'hsl': `[Hh][Ss][Ll][\\(](((([\\d]{1,3}|[\\d\\%]{2,4})[\\,]{0,1})[\\s]*){3})[\\)]`,
          'hsla': `[Hh][Ss][Ll][Aa][\\(](((([\\d]{1,3}|[\\d\\%]{2,4}|[\\d\\.]{1,3})[\\,]{0,1})[\\s]*){4})[\\)]`,
          'hsv': ``,
          'rgb':`[Rr][Gg][Bb][\\(](((([\\d]{1,3})[\\,]{0,1})[\\s]*){3})[\\)]`,
          'rgba':`[Rr][Gg][Bb][Aa][\\(](((([\\d]{1,3}|[\\d\\.]{1,3})[\\,]{0,1})[\\s]*){4})[\\)]`
        }
      }
    },
    computed: {
      pattern(){
        return this.patterns[this.format]
      }
    },
    methods: {
      updateValue(e) {
        this.$emit('update:value', e[this.format])
      },
      setupPopper() {
        if (this.popper === undefined) {
          this.popper = new Popper(this.$refs.colorpicker, this.$refs.picker, {
            placement: 'bottom-end'
          })
        } else {
          this.popper.scheduleUpdate()
        }
      },
      open() {
        if (this.isOpen) return
        this.isOpen = true
        this.$nextTick(() => {
          this.setupPopper()
        })
      },
      close() {
        if (!this.isOpen) return
        this.isOpen = false
        this.$nextTick(() => {
          this.$refs.colorinput.focus()
        })
      },
      toggle() {
        if (this.disabled) return
        this.isOpen ? this.close() : this.open()
      }
    }
  }
</script>

