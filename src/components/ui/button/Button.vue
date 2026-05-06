<script setup lang="ts">
import { computed } from 'vue'
import { cn } from '@/lib/utils'

const props = withDefaults(
  defineProps<{
    variant?: 'default' | 'secondary' | 'outline' | 'ghost' | 'destructive'
    size?: 'default' | 'sm' | 'lg' | 'icon'
    loading?: boolean
    class?: string
  }>(),
  {
    variant: 'default',
    size: 'default',
    loading: false,
    class: '',
  },
)

const classes = computed(() =>
  cn(
    'inline-flex items-center justify-center gap-2 whitespace-nowrap rounded-md text-sm font-medium transition-colors',
    'focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2',
    'disabled:pointer-events-none disabled:opacity-50',
    props.variant === 'default' && 'bg-primary text-primary-foreground shadow hover:bg-primary/90',
    props.variant === 'secondary' && 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
    props.variant === 'outline' && 'border border-input bg-background shadow-sm hover:bg-accent hover:text-accent-foreground',
    props.variant === 'ghost' && 'hover:bg-accent hover:text-accent-foreground',
    props.variant === 'destructive' && 'bg-destructive text-destructive-foreground shadow-sm hover:bg-destructive/90',
    props.size === 'default' && 'h-10 px-4 py-2',
    props.size === 'sm' && 'h-8 rounded-md px-3 text-xs',
    props.size === 'lg' && 'h-11 rounded-md px-8',
    props.size === 'icon' && 'h-10 w-10',
    props.class,
  ),
)
</script>

<template>
  <button :class="classes" :disabled="loading">
    <span v-if="loading" class="h-4 w-4 animate-spin rounded-full border-2 border-current border-t-transparent" />
    <slot />
  </button>
</template>
