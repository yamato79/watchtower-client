<template>
    <label :class="classes" v-bind="attributes">
        <slot></slot>
    </label>    
</template>

<script lang="ts">
import { defineComponent, computed } from "vue";

export default defineComponent({
    name: "x-form-label",
    inheritAttrs: false,
    setup(props, { attrs }) {
        /**
         * The base classes for the component.
         */
        const baseClasses = "font-montserrat block text-sm font-medium text-gray-900 capitalize";

        /**
         * The computed classes for the component.
         */
        const classes = computed(() => `x-form-label ${baseClasses}` );

        /**
         * The attributes for the component.
         */
        const attributes = computed(() => attrs);

        return {
            attributes,
            classes,
        };
    },
});
</script>