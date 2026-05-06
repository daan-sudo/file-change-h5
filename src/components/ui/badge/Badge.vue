<script setup lang="ts">
import { computed } from 'vue'
import { cn } from '@/lib/utils'

const props = withDefaults(
  defineProps<{
    variant?: 'default' | 'secondary' | 'outline' | 'success' | 'warning' | 'destructive'
    class?: string
  }>(),
  {
    variant: 'default',
    class: '',
  },
)

const classes = computed(() =>
  cn(
    'inline-flex items-center rounded-md border px-2.5 py-0.5 text-xs font-semibold transition-colors',
    props.variant === 'default' && 'border-transparent bg-primary text-primary-foreground',
    props.variant === 'secondary' && 'border-transparent bg-secondary text-secondary-foreground',
    props.variant === 'outline' && 'text-foreground',
    props.variant === 'success' && 'border-emerald-200 bg-emerald-50 text-emerald-700',
    props.variant === 'warning' && 'border-amber-200 bg-amber-50 text-amber-700',
    props.variant === 'destructive' && 'border-transparent bg-destructive text-destructive-foreground',
    props.class,
  ),
)
</script>

<template>
  <span :class="classes">
    <slot />
  </span>
</template>
