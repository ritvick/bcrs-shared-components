<template>
  <v-card flat id="detail-comment-container">
    <v-textarea
      ref="textarea"
      :outlined="isStyle('outlined')"
      :filled="isStyle('filled')"
      auto-grow
      :rows="rowCount"
      id="detail-comment-textarea"
      :counter="maxLength"
      :rules="rules"
      :value="value"
      :placeholder="placeholder"
      :autofocus="autofocus"
      @input="emitInput($event)"
    />
  </v-card>
</template>

<script lang="ts">
import Vue from 'vue'
import { Component, Prop, Watch, Emit } from 'vue-property-decorator'
import { Debounce } from 'vue-debounce-decorator'

@Component({})
export default class DetailComment extends Vue {
  /** Array of validations rules for the textarea. */
  get rules (): Array<(v) => boolean | string> {
    // exclude whitespace in minimum length check
    // include whitespace in maximum length check
    return [
      val => (val && val.trim().length > 0) || 'Detail is required.',
      val => (val && val.length <= this.maxLength) || 'Maximum characters exceeded.'
    ]
  }

  /** Public method to reset Vuetify validation on textarea. */
  public resetValidation (): void {
    (this.$refs.textarea as any).resetValidation()
  }

  /** Comment (v-model) passed into this component (required). */
  @Prop({ default: '' }) readonly value!: string

  /** Placeholder passed into this component (optional). */
  @Prop({ default: '' }) readonly placeholder!: string

  /** Max Length passed into this component (optional). */
  @Prop({ default: 4096 }) readonly maxLength!: number

  /** Autofocus passed into this component (optional). */
  @Prop({ default: false }) readonly autofocus!: boolean

  @Prop({ default: 5 }) readonly rowCount!: number

  @Prop({ default: 'outlined' }) readonly textAreaStyle!: string

  /** Called when component is created. */
  created (): void {
    // inform parent of initial validity
    this.emitValid(this.value)
  }

  protected isStyle (style: string): boolean {
    return this.textAreaStyle.toLowerCase() === style.toLowerCase()
  }

  /**
   * Called when prop changes (ie, v-model is updated by parent).
   * This method is debounced to prevent excessive validation.
   */
  @Watch('value')
  @Debounce(300)
  private onValueChanged (val: string): void {
    this.emitValid(val)
  }

  /** Emits an event with the changed comment (ie, updated v-model). */
  @Emit('input')
  protected emitInput (val: string): void { }

  /** Emits an event indicating whether or not this component is valid. */
  @Emit('valid')
  private emitValid (val: string): boolean {
    // component is valid if every rule is valid
    return this.rules.every(rule => rule(val) === true)
  }
}
</script>

<style lang="scss" scoped>
#detail-comment-container {
  line-height: 1.2rem;
  font-size: 0.875rem;
}
</style>
