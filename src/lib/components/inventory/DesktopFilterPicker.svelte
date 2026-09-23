<script lang="ts">
	import Search from '@lucide/svelte/icons/search';
	import InventoryFilterChoice from './InventoryFilterChoice.svelte';
	import type { AuxeroInventoryFilter } from '$lib/server/inventory-options';
	let {
		filter,
		english = false,
		selection = $bindable<string[]>([])
	}: { filter: AuxeroInventoryFilter; english?: boolean; selection?: string[] } = $props();
	const id = $props.id();
	let query = $state('');
	let input = $state<HTMLInputElement>();
	let root = $state<HTMLDivElement>();
	const searchable = $derived(
		['brand', 'model'].includes(filter.name) || filter.options.length > 8
	);
	const matching = $derived(
		filter.options.filter((option) =>
			option.label.toLocaleLowerCase().includes(query.trim().toLocaleLowerCase())
		)
	);
	export function focusSearch() {
		(input ?? root?.querySelector<HTMLInputElement>('input'))?.focus({ preventScroll: true });
	}
	function toggle(value: string) {
		selection =
			filter.mode === 'single'
				? [value]
				: selection.includes(value)
					? selection.filter((item) => item !== value)
					: [...selection, value];
	}
</script>

<div
	class="desktop-picker"
	class:desktop-picker--models={filter.name === 'model'}
	class:desktop-picker--brands={filter.name === 'brand'}
	bind:this={root}
>
	{#if searchable}<div class="desktop-picker__search">
			<label class="filter-control"
				><Search size={20} aria-hidden="true" /><span class="sr-only"
					>{(english ? 'Search in ' : 'Търси в ') + filter.label}</span
				><input
					bind:this={input}
					type="search"
					bind:value={query}
					placeholder={(english ? 'Search in ' : 'Търси в ') + filter.label}
					autocomplete="off"
					onkeydown={(event) => {
						if (event.key === 'Enter') event.preventDefault();
					}}
				/></label
			>
		</div>{/if}
	<div class="desktop-picker__options">
		{#each matching as option (option.value)}<InventoryFilterChoice
				label={option.label}
				image={option.image}
				mode={filter.mode}
				name={id + '-choice'}
				form={id + '-options'}
				checked={selection.includes(option.value)}
				onchange={() => toggle(option.value)}
			/>{:else}<p role="status">{english ? 'No matches' : 'Няма съвпадения'}</p>{/each}
	</div>
</div>

<style>
	.desktop-picker--models .desktop-picker__options {
		grid-template-columns: repeat(3, minmax(0, 1fr));
	}
	.desktop-picker__search {
		position: sticky;
		top: 0;
		z-index: 1;
		padding-bottom: var(--bc-space-4);
		background: var(--bc-surface-raised);
	}
	.desktop-picker__options {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		align-content: start;
		gap: var(--bc-space-2);
		padding-block: var(--bc-space-1);
	}
	@media (max-width: 1199px) {
		.desktop-picker--models .desktop-picker__options {
			grid-template-columns: repeat(2, minmax(0, 1fr));
		}
	}
	@media (min-width: 1200px) {
		.desktop-picker--brands .desktop-picker__options {
			grid-template-columns: repeat(3, minmax(0, 1fr));
		}
	}
	@media (max-width: 899px) {
		.desktop-picker__options,
		.desktop-picker--models .desktop-picker__options {
			grid-template-columns: minmax(0, 1fr);
		}
	}
</style>
