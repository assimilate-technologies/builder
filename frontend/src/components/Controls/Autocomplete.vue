<template>
	<div class="relative">
		<Combobox
			:modelValue="value"
			@update:modelValue="
				(val) => {
					emit('update:modelValue', val);
				}
			"
			v-slot="{ open }"
			:nullable="nullable"
			:multiple="multiple">
			<div
				class="form-input flex h-7 w-full items-center justify-between gap-2 rounded border-outline-gray-1 bg-surface-gray-1 p-0 text-sm text-text-icons-gray-8 transition-colors hover:border-outline-gray-2 hover:bg-surface-gray-1">
				<!-- {{ displayValue }} -->
				<ComboboxInput
					autocomplete="off"
					@change="query = $event.target.value"
					@focus="() => open"
					:displayValue="
						(option: Option) => {
							if (Array.isArray(option)) {
								return option.map((o) => o.label).join(', ');
							} else if (option) {
								return option.label || option.value || '';
							} else {
								return '';
							}
						}
					"
					:placeholder="!modelValue ? placeholder : null"
					class="h-full w-full rounded border-none bg-transparent p-0 px-2 py-1 pr-5 text-base focus:ring-2 focus:ring-outline-gray-3" />
			</div>
			<ComboboxOptions
				class="absolute right-0 z-50 max-h-[15rem] w-full overflow-y-auto rounded-lg bg-surface-white px-1.5 py-1.5 shadow-2xl"
				v-show="filteredOptions.length">
				<ComboboxOption v-if="query" :value="query" class="flex items-center"></ComboboxOption>
				<ComboboxOption
					v-slot="{ active, selected }"
					v-for="option in filteredOptions"
					:key="option.value"
					:value="option"
					class="flex items-center">
					<li
						class="w-full select-none rounded px-2.5 py-1.5 text-xs"
						:class="{
							'bg-gray-100': active,
							'bg-gray-300': selected,
						}">
						{{ option.label }}
					</li>
				</ComboboxOption>
			</ComboboxOptions>
		</Combobox>
		<div
			class="absolute right-[1px] top-[3px] cursor-pointer p-1 text-text-icons-gray-4 hover:text-text-icons-gray-5"
			@click="clearValue"
			v-show="modelValue">
			<CrossIcon />
		</div>
	</div>
</template>

<script setup lang="ts">
import CrossIcon from "@/components/Icons/Cross.vue";
import { Combobox, ComboboxInput, ComboboxOption, ComboboxOptions } from "@headlessui/vue";
import { ComputedRef, PropType, computed, ref } from "vue";

type Option = {
	label: string;
	value: string;
};

const emit = defineEmits(["update:modelValue"]);

const props = defineProps({
	options: {
		type: Array as PropType<Option[]>,
		default: () => [],
	},
	modelValue: {},
	placeholder: {
		type: String,
		default: "Search",
	},
	showInputAsOption: {
		type: Boolean,
		default: false,
	},
});

const query = ref("");

const multiple = computed(() => Array.isArray(props.modelValue));
const nullable = computed(() => !multiple.value);

const value = computed(() => {
	return (
		props.options.find((option) => option.value === props.modelValue) || {
			label: props.modelValue,
			value: props.modelValue,
		}
	);
}) as ComputedRef<Option>;

const filteredOptions = computed(() => {
	if (query.value === "") {
		return props.options;
	} else {
		const options = props.options.filter((option) => {
			return (
				option.label.toLowerCase().includes(query.value.toLowerCase()) ||
				option.value.toLowerCase().includes(query.value.toLowerCase())
			);
		});
		if (props.showInputAsOption) {
			options.unshift({
				label: query.value,
				value: query.value,
			});
		}
		return options;
	}
});

const clearValue = () => emit("update:modelValue", null);
</script>
