<script lang="ts">
	import { createEventDispatcher, onDestroy, onMount } from "svelte";
	import { cubicOut } from "svelte/easing";
	import { fade, fly } from "svelte/transition";
	import Portal from "./Portal.svelte";
	import { browser } from "$app/environment";
	import CarbonClose from "~icons/carbon/close";

	interface Props {
		width?: string;
		closeButton?: boolean;
		children?: import("svelte").Snippet;
	}

	let { width = "max-w-sm", children, closeButton = false }: Props = $props();

	let backdropEl: HTMLDivElement | undefined = $state();
	let modalEl: HTMLDivElement | undefined = $state();

	const dispatch = createEventDispatcher<{ close: void }>();

	function handleKeydown(event: KeyboardEvent) {
		// close on ESC
		if (event.key === "Escape") {
			event.preventDefault();
			dispatch("close");
		}
	}

	function handleBackdropClick(event: MouseEvent) {
		if (window?.getSelection()?.toString()) {
			return;
		}
		if (event.target === backdropEl) {
			dispatch("close");
		}
	}

	onMount(() => {
		document.getElementById("app")?.setAttribute("inert", "true");
		modalEl?.focus();
	});

	onDestroy(() => {
		if (!browser) return;
		document.getElementById("app")?.removeAttribute("inert");
	});
</script>

<Portal>
	<div
		role="presentation"
		tabindex="-1"
		bind:this={backdropEl}
		onclick={(e) => {
			e.stopPropagation();
			handleBackdropClick(e);
		}}
		transition:fade|local={{ easing: cubicOut, duration: 300 }}
		class="fixed inset-0 z-40 flex items-center justify-center bg-black/80 backdrop-blur-sm dark:bg-black/50"
	>
		<div
			role="dialog"
			tabindex="-1"
			bind:this={modalEl}
			onkeydown={handleKeydown}
			in:fly={{ y: 100 }}
			class={[
				"relative mx-auto max-h-[95dvh] max-w-[90dvw] overflow-y-auto overflow-x-hidden rounded-2xl bg-white shadow-2xl outline-none",
				width,
			]}
		>
			{#if closeButton}
				<button class="absolute right-4 top-4 z-50" onclick={() => dispatch("close")}>
					<CarbonClose class="size-6 text-gray-700" />
				</button>
			{/if}
			{@render children?.()}
		</div>
	</div>
</Portal>
