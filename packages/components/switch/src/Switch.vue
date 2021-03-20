<template>
  <button
    ref="switchRef"
    class="ix-switch"
    :class="[
      { 'ix-switch-checked': stateChecked },
      { 'ix-switch-disabled': disabled || loading },
      { 'ix-switch-small': isSmallSize },
    ]"
    type="button"
    @mouseup="handleMouseup"
    @click="handleClick"
  >
    <div v-if="loading" class="ix-switch-loading">
      <ix-icon name="loading" />
    </div>
    <div class="ix-switch-inner">
      <template v-if="stateChecked">
        <slot name="checkedChildren">{{ checkedChildren }}</slot>
      </template>
      <template v-else>
        <slot name="unCheckedChildren">{{ unCheckedChildren }}</slot>
      </template>
    </div>
  </button>
</template>
<script lang="ts">
import { computed, defineComponent, ref, watch } from 'vue'
import { SwitchProps } from './types'
import { PropTypes } from '@idux/cdk/utils'
import { IxIcon } from '@idux/components/icon'

export default defineComponent({
  name: 'IxSwitch',
  components: { IxIcon },
  inheritAttrs: false,
  props: {
    checked: PropTypes.bool.def(false),
    disabled: PropTypes.bool.def(false),
    checkedChildren: PropTypes.string.def(''),
    unCheckedChildren: PropTypes.string.def(''),
    size: PropTypes.oneOf(['normal', 'small'] as const).def('normal'),
    loading: PropTypes.bool.def(false),
  },
  emits: ['change', 'update:checked'],
  setup(props: SwitchProps, { emit }) {
    const stateChecked = ref(props.checked ?? false)
    const switchRef = ref<HTMLButtonElement | null>(null)

    const isSmallSize = computed(() => props.size === 'small')

    watch(
      () => props.checked,
      curChecked => {
        stateChecked.value = curChecked ?? false
      },
    )

    const setChecked = (checked: boolean) => {
      stateChecked.value = !checked
      emit('update:checked', stateChecked.value)
    }

    const handleClick = () => {
      if (props.disabled || props.loading) {
        return
      }
      setChecked(stateChecked.value)
      emit('change', stateChecked.value)
    }

    const handleMouseup = () => {
      blur()
    }

    const focus = () => {
      switchRef.value?.focus()
    }

    const blur = () => {
      switchRef.value?.blur()
    }

    return {
      stateChecked,
      handleClick,
      isSmallSize,
      handleMouseup,
      switchRef,
      focus,
      blur,
    }
  },
})
</script>