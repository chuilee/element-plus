<template>
  <div
    ref="contentRef"
    v-bind="contentAttrs"
    :style="contentStyle"
    :class="contentClass"
    role="tooltip"
    @mouseenter="(e) => $emit('mouseenter', e)"
    @mouseleave="(e) => $emit('mouseleave', e)"
  >
    <slot />
  </div>
</template>

<script lang="ts" setup>
import { computed, inject, onMounted, provide, ref, unref, watch } from 'vue'
import { createPopper } from '@popperjs/core'
import { useNamespace, useZIndex } from '@element-plus/hooks'
import {
  POPPER_CONTENT_INJECTION_KEY,
  POPPER_INJECTION_KEY,
} from '@element-plus/tokens'
import { popperContentEmits, usePopperContentProps } from './content'
import { buildPopperOptions, unwrapMeasurableEl } from './utils'
import {
  usePopperContent,
  usePopperContentDOM,
  usePopperContentFocusTrap,
} from './composables'

import type { WatchStopHandle } from 'vue'

defineOptions({
  name: 'ElPopperContent',
})

const props = defineProps(usePopperContentProps)
const emit = defineEmits(popperContentEmits)

const { attributes, arrowRef, contentRef, styles, instanceRef, role, update } =
  usePopperContent(props)

const {
  ariaModal,
  arrowStyle,
  contentAttrs,
  contentClass,
  contentStyle,
  updateZIndex,
} = usePopperContentDOM(props, {
  styles,
  attributes,
  role,
})

const { nextZIndex } = useZIndex()
const ns = useNamespace('popper')
const popperContentRef = ref<HTMLElement>()
const arrowOffset = ref<number>()
provide(POPPER_CONTENT_INJECTION_KEY, {
  arrowStyle,
  arrowRef,
  arrowOffset,
})
const contentZIndex = ref(props.zIndex || nextZIndex())

const updatePopper = (shouldUpdateZIndex = true) => {
  update()
  shouldUpdateZIndex && updateZIndex()
}

const togglePopperAlive = () => {
  updatePopper(false)
}

onMounted(() => {
  watch(() => props.visible, togglePopperAlive, { immediate: true })
})

defineExpose({
  /**
   * @description popper content element
   */
  popperContentRef,
  /**
   * @description popperjs instance
   */
  popperInstanceRef: instanceRef,
  /**
   * @description method for updating popper
   */
  updatePopper,

  /**
   * @description content style
   */
  contentStyle,
})
</script>
