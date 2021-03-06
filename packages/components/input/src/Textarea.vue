<template>
  <span :class="['ix-textarea', classes, $attrs.class]" :style="$attrs.style" :data-count="dataCount">
    <textarea
      ref="textareaRef"
      v-bind="attrs"
      class="ix-textarea-inner"
      :style="{ resize: resize$$ }"
      :disabled="disabled$$"
      :readonly="readonly"
      @compositionstart="onCompositionStart"
      @compositionend="onCompositionEnd"
      @input="onInput"
      @focus="onFocus"
      @blur="onBlur"
    >
    </textarea>
    <span v-if="isClearable" class="ix-textarea-suffix">
      <ix-icon :class="{ 'ix-textarea-clear-icon-hidden': clearHidden }" name="close-circle" @click="onClearClick" />
    </span>
  </span>
</template>
<script lang="ts">
import type { ComputedRef, Ref } from 'vue'
import type { ValueAccessor } from '@idux/cdk/forms'
import type { TextareaConfig } from '@idux/components/config'
import { textareaProps, TextareaProps } from './types'

import { computed, defineComponent, ref } from 'vue'

import { useGlobalConfig } from '@idux/components/config'
import { useAttrs } from '@idux/components/utils'
import { IxIcon } from '@idux/components/icon'
import { useAutoRows } from './useAutoRows'
import { useCommonBindings } from './useCommonBindings'

export default defineComponent({
  name: 'IxTextarea',
  components: { IxIcon },
  props: textareaProps,
  setup(props) {
    const attrs = useAttrs()
    const textareaConfig = useGlobalConfig('textarea')
    const textareaRef = ref(null as unknown as HTMLTextAreaElement)

    const {
      disabled: disabled$$,
      focus,
      blur,
      onCompositionStart,
      onCompositionEnd,
      onInput,
      onFocus,
      onBlur,
      onClearClick,
      isClearable,
      clearHidden,
      isFocused,
      valueAccessor,
    } = useCommonBindings(props, textareaConfig, textareaRef)

    const classes = useClasses(props, textareaConfig, isFocused, disabled$$)
    const dataCount = useDataCount(props, textareaConfig, valueAccessor)
    const resize$$ = computed(() => {
      const autoRows = props.autoRows ?? textareaConfig.autoRows
      const resize = props.resize ?? textareaConfig.resize
      if (autoRows) {
        return resize === 'horizontal' ? 'horizontal' : 'none'
      } else {
        return resize
      }
    })

    const autoRows = computed(() => props.autoRows ?? textareaConfig.autoRows)
    useAutoRows(textareaRef, autoRows, valueAccessor)

    return {
      disabled$$,
      focus,
      blur,
      attrs,
      textareaRef,
      classes,
      dataCount,
      resize$$,
      onCompositionStart,
      onCompositionEnd,
      onInput,
      onFocus,
      onBlur,
      onClearClick,
      isClearable,
      clearHidden,
    }
  },
})

function useClasses(
  props: TextareaProps,
  config: TextareaConfig,
  isFocused: Ref<boolean>,
  disabled: ComputedRef<boolean>,
) {
  return computed(() => {
    const sizeClass = `ix-textarea-${props.size ?? config.size}`
    return {
      [sizeClass]: true,
      'ix-textarea-disabled': disabled.value,
      'ix-textarea-focused': isFocused.value,
      'ix-textarea-with-count': props.showCount ?? config.showCount,
    }
  })
}

function useDataCount(props: TextareaProps, config: TextareaConfig, valueAccessor: ValueAccessor) {
  return computed(() => {
    const showCount = props.showCount ?? config.showCount
    const computeCount = props.computeCount ?? config.computeCount
    const maxCount = props.maxCount ?? config.maxCount
    let dataCount = ''
    if (showCount) {
      const value = valueAccessor.value ?? ''
      dataCount = value.length
      if (computeCount) {
        dataCount = computeCount(value)
      } else if (maxCount) {
        dataCount += ' / ' + maxCount
      }
    }
    return dataCount
  })
}
</script>
