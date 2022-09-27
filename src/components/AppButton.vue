<script setup>
const props = defineProps({
  tag: {
    type: String,
    default: 'button',
  },
  variant: {
    type: String,
    default: 'light',
    validate: (value) => {
      return ['dark', 'light', 'outline-dark'].includes(value)
    },
  },
})

const bgClasses = computed(() => {
  return {
    'bg-neutral-900 text-white': props.variant === 'light',
    'bg-transparent border-neutral-900 border-2': props.variant === 'outline-dark',
    'bg-gray-200 text-gray-900': props.variant === 'gray',
  }
})

const buttonClasses = computed(() => {
  return {
    'text-white': props.variant === 'light',
    'bg-text': props.variant === 'outline-dark',
    'bg-gray-200 text-gray-900': props.variant === 'gray',
  }
})
</script>

<template>
  <component :is="tag" class="py-4 px-8 text-xl relative group cursor-pointer" :class="buttonClasses">
    <span class="absolute top-0 left-0 w-16 h-16 rounded-full group-hover:w-full transition-all ease-out hover:ease-in duration-500 leading-none gs_reveal" :class="bgClasses" />
    <span class="z-10 gs_reveal relative inline-block" :data-delay="0.15" data-y="75%">
      <slot />
    </span>
  </component>
</template>
