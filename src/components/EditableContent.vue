<template>
  <div v-if="!edit">
    <slot></slot>
  </div>
  <input
    placeholder="what's up?"
    autofocus
    v-bind="$attrs"
    v-model="content"
    @input="updateContent"
    v-else
  />
</template>

<script lang="ts">
import { defineComponent, ref } from "vue";

export default defineComponent({
  inheritAttrs: false,
  props: {
    edit: {
      type: Boolean,
      default: false,
    },
    modelValue: {
      type: String,
    },
  },
  emits: ["update:modelValue"],
  setup(props, context) {
    const content = ref(props.modelValue);
    const updateContent = (event: InputEvent) => {
      const target = event.target as HTMLInputElement;
      context.emit("update:modelValue", target.value);
    };
    return {
      content,
      updateContent,
    };
  },
});
</script>

<style scoped></style>
