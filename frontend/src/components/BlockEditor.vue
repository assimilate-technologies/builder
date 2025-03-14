<template>
	<div
		class="editor pointer-events-none fixed z-[18] box-content select-none ring-2 ring-inset"
		ref="editor"
		:selected="isBlockSelected"
		@click.stop="handleClick"
		@dblclick="handleDoubleClick"
		@mousedown.prevent="handleMove"
		@drop.prevent.stop="handleDrop"
		:data-block-id="block.blockId"
		:class="getStyleClasses">
		<PaddingHandler
			:data-block-id="block.blockId"
			v-show="showPaddingHandler"
			:target-block="block"
			:target="target"
			:on-update="updateTracker"
			:disable-handlers="false"
			:breakpoint="breakpoint" />
		<MarginHandler
			v-show="showMarginHandler"
			:target-block="block"
			:target="target"
			:on-update="updateTracker"
			:disable-handlers="false"
			:breakpoint="breakpoint" />
		<BorderRadiusHandler
			:data-block-id="block.blockId"
			v-if="showBorderRadiusHandler"
			:target-block="block"
			:target="target" />
		<BoxResizer v-if="showResizer" :targetBlock="block" @resizing="resizing = $event" :target="target" />
	</div>
</template>
<script setup lang="ts">
import Block from "@/utils/block";
import { addPxToNumber } from "@/utils/helpers";
import { Ref, computed, inject, nextTick, onMounted, ref, watch, watchEffect } from "vue";

import blockController from "@/utils/blockController";
import useStore from "../store";
import setGuides from "../utils/guidesTracker";
import trackTarget from "../utils/trackTarget";
import BorderRadiusHandler from "./BorderRadiusHandler.vue";
import BoxResizer from "./BoxResizer.vue";
import MarginHandler from "./MarginHandler.vue";
import PaddingHandler from "./PaddingHandler.vue";

const canvasProps = inject("canvasProps") as CanvasProps;

const showResizer = computed(() => {
	return (
		!props.block.isRoot() &&
		!props.editable &&
		!store.isDragging &&
		isBlockSelected.value &&
		!blockController.multipleBlocksSelected() &&
		!props.block.getParentBlock()?.isGrid() &&
		!(props.block.isHTML() && !props.block.isSVG() && !props.block.isIframe())
	);
});

const props = defineProps({
	block: {
		type: Block,
		required: true,
	},
	breakpoint: {
		type: String,
		default: "desktop",
	},
	target: {
		type: [HTMLElement, SVGElement],
		required: true,
	},
	editable: {
		type: Boolean,
		default: false,
	},
	isSelected: {
		type: Boolean,
		default: false,
	},
});
const store = useStore();
const editor = ref(null) as unknown as Ref<HTMLElement>;
const updateTracker = ref(() => {});
const resizing = ref(false);
const guides = setGuides(props.target, canvasProps);
const moving = ref(false);
const preventCLick = ref(false);

const showPaddingHandler = computed(() => {
	return (
		isBlockSelected.value &&
		!resizing.value &&
		!store.isDragging &&
		!props.editable &&
		!blockController.multipleBlocksSelected() &&
		!props.block.isSVG() &&
		(!props.block.isText() || (props.block.isLink() && props.block.hasChildren()))
	);
});

const showMarginHandler = computed(() => {
	return (
		isBlockSelected.value &&
		!props.block.isRoot() &&
		!store.isDragging &&
		!resizing.value &&
		!props.editable &&
		!blockController.multipleBlocksSelected() &&
		(!props.block.isText() || (props.block.isLink() && props.block.hasChildren()))
	);
});

const showBorderRadiusHandler = computed(() => {
	return (
		isBlockSelected.value &&
		!props.block.isRoot() &&
		!props.block.isText() &&
		!props.block.isHTML() &&
		!props.block.isSVG() &&
		!props.editable &&
		!resizing.value &&
		!store.isDragging &&
		!blockController.multipleBlocksSelected()
	);
});

watchEffect(() => {
	props.block.getStyle("top");
	props.block.getStyle("left");
	props.block.getStyle("bottom");
	props.block.getStyle("right");
	props.block.getStyle("position");
	props.block.rawStyles;
	const parentBlock = props.block.getParentBlock();
	parentBlock?.getStyle("display");
	parentBlock?.getStyle("justifyContent");
	parentBlock?.getStyle("alignItems");
	parentBlock?.getStyle("flexDirection");
	parentBlock?.getStyle("paddingTop");
	parentBlock?.getStyle("paddingBottom");
	parentBlock?.getStyle("paddingLeft");
	parentBlock?.getStyle("paddingRight");
	parentBlock?.getStyle("margin");
	parentBlock?.getChildIndex(props.block);
	store.builderLayout.leftPanelWidth;
	store.builderLayout.rightPanelWidth;
	store.showRightPanel;
	store.showLeftPanel;
	store.activeBreakpoint;
	store.dropTarget.placeholder;
	canvasProps.breakpoints.map((bp) => bp.visible);
	nextTick(() => {
		updateTracker.value();
	});
});

const isBlockSelected = computed(() => {
	return props.isSelected && props.breakpoint === store.activeBreakpoint;
});

const getStyleClasses = computed(() => {
	const classes = [];
	if (movable.value && !props.block.isRoot()) {
		classes.push("cursor-grab");
	}
	if (props.block.isExtendedFromComponent()) {
		classes.push("ring-purple-400");
	} else {
		classes.push("ring-blue-400");
	}
	if (
		isBlockSelected.value &&
		!props.editable &&
		!props.block.isRoot() &&
		!props.block.isRepeater() &&
		!store.isDragging
	) {
		// make editor interactive
		classes.push("pointer-events-auto");
		// Place the block on the top of the stack
		classes.push("!z-[19]");
	}
	return classes;
});

watch(
	() => store.activeCanvas?.block,
	() => {
		nextTick(() => {
			updateTracker.value();
		});
	},
);

const movable = computed(() => {
	return props.block.isMovable();
});

onMounted(() => {
	updateTracker.value = trackTarget(props.target, editor.value, canvasProps);
});

const handleClick = (ev: MouseEvent) => {
	if (props.editable) return;
	if (preventCLick.value) {
		preventCLick.value = false;
		return;
	}

	if (props.block.isText() || props.block.isButton() || props.block.isLink()) {
		store.editableBlock = props.block;
	}

	const editorWrapper = editor.value;
	editorWrapper.classList.add("pointer-events-none");
	let element = document.elementFromPoint(ev.x, ev.y) as HTMLElement;
	if (element.classList.contains("editor")) {
		element.classList.remove("pointer-events-auto");
		element.classList.add("pointer-events-none");
		element = document.elementFromPoint(ev.x, ev.y) as HTMLElement;
	}
	if (element.classList.contains("__builder_component__")) {
		element.dispatchEvent(new MouseEvent("click", ev));
	}
};

// dispatch drop event to the target block
const handleDrop = (ev: DragEvent) => {
	if (props.editable) return;
	const dropEvent = new DragEvent("drop", ev);
	props.target.dispatchEvent(dropEvent);
};

const handleDoubleClick = (ev: MouseEvent) => {
	if (props.block.isHTML()) {
		store.editHTML(props.block);
		return;
	}
	if (props.editable) return;
	if (props.block.isText() || props.block.isButton() || props.block.isLink()) {
		store.editableBlock = props.block;
	}
};

const handleMove = (ev: MouseEvent) => {
	if (store.mode === "text") {
		store.editableBlock = props.block;
	}
	if (!movable.value || props.block.isRoot()) return;
	const pauseId = store.activeCanvas?.history?.pause();
	const target = ev.target as HTMLElement;
	const startX = ev.clientX;
	const startY = ev.clientY;
	const startLeft = props.target.offsetLeft || 0;
	const startTop = props.target.offsetTop || 0;

	moving.value = true;
	guides.showX();

	// to disable cursor jitter
	const docCursor = document.body.style.cursor;
	document.body.style.cursor = "grabbing";
	target.style.cursor = "grabbing";

	const mousemove = async (mouseMoveEvent: MouseEvent) => {
		const scale = canvasProps.scale;
		const movementX = (mouseMoveEvent.clientX - startX) / scale;
		const movementY = (mouseMoveEvent.clientY - startY) / scale;
		let finalLeft = startLeft + movementX;
		let finalTop = startTop + movementY;
		props.block.setStyle("left", addPxToNumber(finalLeft));
		props.block.setStyle("top", addPxToNumber(finalTop));
		await nextTick();
		const { leftOffset, rightOffset } = guides.getPositionOffset();
		if (leftOffset !== 0) {
			props.block.setStyle("left", addPxToNumber(finalLeft + leftOffset));
		}
		if (rightOffset !== 0) {
			props.block.setStyle("left", addPxToNumber(finalLeft + rightOffset));
		}

		mouseMoveEvent.preventDefault();
		preventCLick.value = true;
	};
	document.addEventListener("mousemove", mousemove);
	document.addEventListener(
		"mouseup",
		(mouseUpEvent) => {
			moving.value = false;
			document.body.style.cursor = docCursor;
			target.style.cursor = "grab";
			document.removeEventListener("mousemove", mousemove);
			mouseUpEvent.preventDefault();
			guides.hideX();
			store.activeCanvas?.history?.resume(pauseId, true);
		},
		{ once: true },
	);
};

defineExpose({
	element: editor,
});
</script>
