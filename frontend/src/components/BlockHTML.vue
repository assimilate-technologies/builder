<template>
	<div ref="component" class="!relative" v-html="html"></div>
</template>
<script setup lang="ts">
import Block from "@/utils/block";
import { computed, ref } from "vue";

const component = ref<HTMLElement | null>(null);
const props = defineProps({
	block: {
		type: Block,
		required: true,
	},
	data: {
		type: Object,
		default: null,
	},
});
const html = computed(
	() => `
		<div class="absolute top-0 bottom-0 right-0 left-0"></div>
		${props.block.getInnerHTML()}
	`,
);

defineExpose({
	component,
});
</script>
