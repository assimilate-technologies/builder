<template>
	<div>
		<router-view v-slot="{ Component }">
			<keep-alive>
				<component :is="Component" />
			</keep-alive>
		</router-view>
		<UseDark attribute="data-theme"></UseDark>
		<Toaster :theme="isDark ? 'dark' : 'light'" richColors closeButton />
		<Dialogs></Dialogs>
	</div>
</template>
<script setup lang="ts">
import { UseDark } from "@vueuse/components";
import { useDark, useTitle } from "@vueuse/core";
import { Dialogs } from "frappe-ui";
import { computed, provide } from "vue";
import { useRoute } from "vue-router";
import { Toaster } from "vue-sonner";
import { sessionUser } from "./router";
import useStore from "./store";

const store = useStore();
const route = useRoute();

provide("sessionUser", sessionUser);

const title = computed(() => {
	return store.activePage && route.name !== "home"
		? `${store.activePage.page_title || "Untitled"} | Builder`
		: "Frappe Builder";
});

useTitle(title);

const isDark = useDark({
	attribute: "data-theme",
});
</script>
<style>
[id^="headlessui-dialog"] {
	@apply z-50;
}

[id^="headlessui-dialog-panel"] {
	@apply bg-surface-gray-1;
	@apply dark:border-outline-gray-1;
	@apply dark:border;
}

/* TODO: Remove this when the issue is fixed */
[id^="headlessui-dialog-panel"] > div > div > div > div.mb-6.flex.items-center.justify-between > button {
	@apply bg-surface-gray-1;
	@apply hover:bg-surface-gray-3;
	@apply stroke-ink-gray-8;
	@apply hover:stroke-ink-gray-9;
	> svg {
		@apply stroke-[0.2px];
		@apply h-[14px];
	}
}

[id^="headlessui-dialog-panel"] > div,
[id^="headlessui-dialog-panel"] .space-y-4 > p {
	@apply bg-surface-white;
	@apply text-ink-gray-8;
}

[id^="headlessui-dialog-panel"] header h3 {
	@apply dark:text-white;
}

[id^="headlessui-dialog-panel"] label > span {
	@apply dark:text-gray-50;
}

[id^="headlessui-dialog"] [data-dialog] {
	@apply dark:bg-black-overlay-800;
}

[id^="headlessui-menu-items"],
[id^="headlessui-combobox-options"] {
	@apply bg-surface-white;
	@apply dark:bg-surface-gray-1;
	@apply text-ink-gray-7;

	@apply overflow-y-auto;
	-ms-overflow-style: none; /* IE and Edge */
	scrollbar-width: none;
	@apply max-w-60;
	max-height: min(60vh, 18rem);
}
[id^="headlessui-menu-items"] [id^="headlessui-menu-item"] > span {
	@apply truncate;
}
.divide-gray-100 > :not([hidden]) ~ :not([hidden]) {
	@apply dark:border-gray-700;
}
[id^="headlessui-menu-items"] &::webkit-scrollbar {
	display: none;
}

[id^="headlessui-menu-items"] button,
[id^="headlessui-combobox-options"] li {
	@apply dark:text-gray-200;
	@apply dark:hover:bg-gray-700;
	@apply dark:rounded;
	@apply break-all;
}

[data-headlessui-state~="active"] li {
	@apply dark:bg-gray-600;
	@apply dark:text-gray-200;
}

[data-headlessui-state="selected"] li {
	@apply dark:bg-gray-700;
	@apply dark:text-gray-200;
}

[id^="headlessui-menu-items"] button svg {
	@apply dark:text-gray-200;
}

[data-sonner-toaster] {
	font-family: "InterVar";
}

[data-sonner-toast][data-styled="true"] {
	@apply bg-surface-white;
	@apply dark:border-gray-800;
	@apply !text-base;
}

[id^="headlessui-menu-items"] {
	@apply min-w-28;
	@apply rounded-md;
}
[id^="headlessui-menu-item"] {
	@apply text-base;
}
[id^="headlessui-menu-item"] button {
	@apply rounded;
}
[id^="headlessui-menu-item"] svg {
	@apply size-3;
}
</style>
