<template>
	<InlineInput
		type="autocomplete"
		:label="label"
		:enableSlider="true"
		:options="[
			{
				label: 'Auto',
				value: 'auto',
			},
			{
				label: 'Fit Content',
				value: 'fit-content',
			},
			{
				label: 'Stretch',
				value: '100%',
			},
		]"
		:modelValue="blockController.getStyle(props.property)"
		:unitOptions="['px', '%', 'vw', 'vh']"
		@update:modelValue="(val) => blockController.setStyle(property, val)"></InlineInput>
</template>
<script setup lang="ts">
import InlineInput from "@/components/Controls/InlineInput.vue";
import { styleProperty } from "@/utils/block";
import blockController from "@/utils/blockController";
import { PropType } from "vue";

const props = defineProps({
	property: {
		type: String as PropType<styleProperty>,
		required: true,
	},
	label: {
		type: String,
		required: true,
	},
});
</script>
