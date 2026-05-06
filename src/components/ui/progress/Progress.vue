<script setup lang="ts">
import { computed } from 'vue'
import { cn } from '@/lib/utils'

const props = withDefaults(
  defineProps<{
    modelValue?: number
    class?: string
  }>(),
  {
    modelValue: 0,
    class: '',
  },
)

const width = computed(() => `${Math.min(Math.max(props.modelValue, 0), 100)}%`)
</script>

<template>
  <div :class="cn('relative h-3 w-full overflow-hidden rounded-full bg-secondary', props.class)">
    <div class="h-full w-full flex-1 rounded-full bg-primary transition-all duration-500" :style="{ transform: `translateX(calc(${width} - 100%))` }" />
  </div>
</template>
