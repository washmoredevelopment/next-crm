<template>
  <Tooltip :text="text" :placement="placement" :hoverDelay="0.3">
    <FeatherIcon name="info" :class="computedIconClass" />
  </Tooltip>
</template>

<script setup>
import { Tooltip, FeatherIcon } from 'frappe-ui'
import { computed } from 'vue'

const props = defineProps({
  text: {
    type: String,
    required: true,
  },
  placement: {
    type: String,
    default: 'top',
  },
  iconClass: {
    type: String,
    default: '',
  },
})

const computedIconClass = computed(() => {
  return props.iconClass || 'h-4 w-4 text-ink-gray-4 cursor-help'
})
</script>
