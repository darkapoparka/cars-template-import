<script lang="ts">
	import { onMount } from 'svelte';
	import Search from '@lucide/svelte/icons/search';
	import { assetHref } from '$lib/utils/assets';
	import { inventoryFilterParam } from '$lib/domain/inventory-query';
	import type { AuxeroInventoryFilter } from '$lib/server/inventory-options';
	let {
		filter,
		english = false,
		selection = $bindable<string[]>([]),
		serialize = false
	}: {
		filter: AuxeroInventoryFilter;
		english?: boolean;
		selection?: string[];
		serialize?: boolean;
	} = $props();
	const id = $props.id();
	let query = $state('');
	let input = $state<HTMLInputElement>();
	const matching = $derived(
		filter.options.filter((option) =>
			option.label.toLocaleLowerCase().includes(query.trim().toLocaleLowerCase())
		)
	);
	onMount(() => input?.focus({ preventScroll: true }));
	function toggle(value: string) {
		selection =
			filter.mode === 'single'
				? [value]
				: selection.includes(value)
					? selection.filter((item) => item !== value)
					: [...selection, value];
	}
</script>

<div class="desktop-picker">
	<div class="desktop-picker__search">
		<label class="filter-control">
			{#if !filter.numericInput}<Search size={20} aria-hidden="true" />{/if}
			<span class="sr-only"
				>{filter.numericInput
					? `${filter.numericInput.label} (${filter.numericInput.unit})`
					: (english ? 'Search in ' : 'Търси в ') + filter.label}</span
			>
			{#if filter.numericInput}
				<input
					bind:this={input}
					type="number"
					min="1"
					step="1"
					inputmode="numeric"
					value={selection[0] ?? ''}
					placeholder={filter.numericInput.label}
					oninput={(event) => {
						const value = event.currentTarget.valueAsNumber;
						selection = Number.isFinite(value) ? [String(value)] : [];
					}}
				/>
				<span class="desktop-picker__unit" aria-hidden="true">{filter.numericInput.unit}</span>
			{:else}
				<input
					bind:this={input}
					type="search"
					bind:value={query}
					placeholder={(english ? 'Search in ' : 'Търси в ') + filter.label}
					autocomplete="off"
				/>
			{/if}
		</label>
	</div>
	{#if serialize}
		{#each selection as value (value)}<input
				type="hidden"
				name={inventoryFilterParam(filter.name)}
				{value}
			/>{/each}
	{/if}
	<div class="desktop-picker__options">
		{#each matching as option (option.value)}
			<label class="desktop-picker__option">
				{#if option.image}<img src={assetHref(option.image)} alt="" width="36" height="28" />{/if}
				<span>{option.label}</span>
				<input
					type={filter.mode === 'single' ? 'radio' : 'checkbox'}
					name={id + '-choice'}
					form={id + '-options'}
					checked={selection.includes(option.value)}
					onchange={() => toggle(option.value)}
				/>
			</label>
		{:else}<p role="status">{english ? 'No matches' : 'Няма съвпадения'}</p>{/each}
	</div>
</div>

<style>
	.desktop-picker__search {
		position: sticky;
		top: 0;
		z-index: 1;
		padding: var(--bc-space-1) var(--bc-space-1) var(--bc-space-4);
		background: var(--bc-surface-raised);
	}
	.desktop-picker__options {
		display: grid;
		gap: var(--bc-space-2);
		padding: var(--bc-space-1);
	}
	.desktop-picker__option {
		display: flex;
		align-items: center;
		gap: var(--bc-space-3);
		min-height: var(--bc-control-height-hero);
		padding: var(--bc-space-3) var(--bc-space-4);
		border: 1px solid var(--bc-border);
		border-radius: var(--bc-radius-md);
		color: var(--bc-ink);
		font-size: var(--bc-text-control);
		cursor: pointer;
	}
	.desktop-picker__option span {
		flex: 1;
	}
	.desktop-picker__option:hover {
		background: var(--bc-surface);
	}
	.desktop-picker__option:has(:checked) {
		border-color: var(--bc-ink);
		background: var(--bc-surface);
		font-weight: var(--bc-weight-heading);
	}
	.desktop-picker__option:has(:focus-visible) {
		outline: 2px solid var(--bc-focus);
		outline-offset: 2px;
	}
	.desktop-picker__option input {
		margin: 0;
		width: var(--bc-text-control);
		height: var(--bc-text-control);
		flex-shrink: 0;
		accent-color: var(--bc-ink);
	}
	.desktop-picker__option img {
		object-fit: contain;
		flex-shrink: 0;
	}
	.desktop-picker__unit {
		color: var(--bc-muted);
		font-size: var(--bc-text-label);
	}
</style>
