<template>
  <div :class="`col-12 text-muted text-${align}`">
    <small>{{ content }}</small>
    <h4 class="mt-2">¯\_(ツ)_/¯</h4>
    <slot></slot>
  </div>
</template>

<script>
export default {
  props: {
    content: {
      default: 'Aún no hay nada',
    },
    align: {
      default: 'center',
    },
  },
};
</script>
