<script lang="ts">
	import { tick } from 'svelte';
	import { SvelteURLSearchParams } from 'svelte/reactivity';
	import DesktopFilterPicker from './DesktopFilterPicker.svelte';
	import '$lib/styles/desktop-filters.css';
	import { page } from '$app/state';
	import SlidersHorizontal from '@lucide/svelte/icons/sliders-horizontal';
	import X from '@lucide/svelte/icons/x';
	import LayoutGrid from '@lucide/svelte/icons/layout-grid';
	import type {
		AuxeroInventoryDesktopData,
		AuxeroInventoryFilter
	} from '$lib/server/inventory-options';
	import InventoryFilter from './InventoryFilter.svelte';
	import InventoryCompactField from './InventoryCompactField.svelte';
	import Search from '@lucide/svelte/icons/search';
	import {
		inventoryFilterParam,
		parseInventoryQuery,
		serializeInventoryQuery
	} from '$lib/domain/inventory-query';
	import Action from '$lib/components/common/Action.svelte';
	import Modal from '$lib/components/common/Modal.svelte';
	import { linkHref } from '$lib/utils/links';
	let { desktop, english = false }: { desktop: AuxeroInventoryDesktopData; english?: boolean } =
		$props();
	let allOpen = $state(false);
	let draft = $state<Record<string, string[]>>({});
	let keyword = $state('');
	let activeFilter = $state<AuxeroInventoryFilter | null>(null);
	let returnTrigger: HTMLButtonElement | undefined;
	let picker = $state<DesktopFilterPicker>();
	let keywordInput = $state<HTMLInputElement>();
	let filterForm = $state<HTMLFormElement>();
	let resultCount = $state<number | null>(null);
	let counting = $state(false);
	const draftFilters = $derived.by(() => {
		const make = desktop.filters.find((filter) => filter.name === 'brand');
		const brands = make ? (draft[make.id] ?? []) : [];
		return desktop.filters.map((filter) =>
			filter.modelCatalog
				? {
						...filter,
						options: filter.modelCatalog
							.filter(
								(option) =>
									!brands.length ||
									option.brands.some((brand) =>
										brands.some((selected) => selected.toLowerCase() === brand.toLowerCase())
									)
							)
							.map((option) => ({
								...option,
								label:
									brands.length === 1
										? option.value
										: `${option.brands.join(' / ')} ${option.value}`
							}))
							.sort((a, b) => a.label.localeCompare(b.label))
					}
				: filter
		);
	});
	const currentFilter = $derived(draftFilters.find((filter) => filter.id === activeFilter?.id));
	const orderedFilters = $derived(
		[...draftFilters].sort((a, b) => {
			const order = [
				'brand',
				'model',
				'priceTo',
				'mileageTo',
				'bodyType',
				'transmission',
				'fuel',
				'feature'
			];
			return order.indexOf(a.name) - order.indexOf(b.name);
		})
	);
	function updateSelection(filter: AuxeroInventoryFilter, values: string[]) {
		draft[filter.id] = values;
		if (filter.name === 'brand') {
			const model = draftFilters.find((item) => item.name === 'model');
			if (model)
				draft[model.id] = (draft[model.id] ?? []).filter((value) =>
					model.options.some((option) => option.value === value)
				);
		}
	}
	$effect(() => {
		if (!allOpen) return;
		const params = new SvelteURLSearchParams(passthrough);
		if (keyword.trim()) params.set('keyword', keyword.trim());
		for (const filter of desktop.filters) {
			for (const value of draft[filter.id] ?? [])
				params.append(inventoryFilterParam(filter.name), value);
		}
		const controller = new AbortController();
		counting = true;
		resultCount = null;
		const timer = setTimeout(async () => {
			try {
				const response = await fetch(`${linkHref('/api/inventory/count')}?${params}`, {
					signal: controller.signal
				});
				if (!response.ok) throw new Error('Inventory count unavailable');
				const data = await response.json();
				if (!controller.signal.aborted && Number.isInteger(data.count)) resultCount = data.count;
			} catch {
				// Filtering still works through the form when the preview is unavailable.
			} finally {
				if (!controller.signal.aborted) counting = false;
			}
		}, 180);
		return () => {
			clearTimeout(timer);
			controller.abort();
		};
	});
	async function applyFilters() {
		activeFilter = null;
		await tick();
		filterForm?.requestSubmit();
	}
	async function chooseFilter(filter: AuxeroInventoryFilter, trigger: HTMLButtonElement) {
		returnTrigger = trigger;
		activeFilter = filter;
		await tick();
		picker?.focusSearch();
	}
	async function backToFilters() {
		activeFilter = null;
		await tick();
		returnTrigger?.focus({ preventScroll: true });
	}
	function openFilters() {
		activeFilter = null;
		draft = Object.fromEntries(
			desktop.filters.map((filter) => [filter.id, [...filter.selectedValues]])
		);
		keyword = page.url.searchParams.get('keyword') ?? '';
		allOpen = true;
	}
	let viewMenu: HTMLDetailsElement;
	function dismissViewMenu(event: PointerEvent | FocusEvent) {
		if (event.target instanceof Node && !viewMenu?.contains(event.target)) {
			if (viewMenu) viewMenu.open = false;
		}
	}
	function handleViewKey(event: KeyboardEvent) {
		if (event.key === 'Escape' && viewMenu?.open) {
			viewMenu.open = false;
			viewMenu.querySelector('summary')?.focus();
		}
	}
	const formId = $props.id();
	const quickFilters = $derived(
		[
			['brand'],
			['q', 'model'],
			['maxPrice', 'priceTo'],
			['minYear', 'yearFrom', 'mileageTo', 'maxMileage'],
			['fuel'],
			['body', 'bodyType']
		]
			.map((names) => desktop.filters.find((filter) => names.includes(filter.name)))
			.filter((filter) => filter !== undefined)
	);
	const fieldNames = $derived(
		new Set(desktop.filters.map((filter) => inventoryFilterParam(filter.name)))
	);
	const passthrough = $derived(
		[
			...serializeInventoryQuery(parseInventoryQuery(page.url.searchParams), page.url.searchParams)
		].filter(([name]) => !fieldNames.has(name) && name !== 'keyword' && name !== 'page')
	);
</script>

<svelte:document
	onpointerdown={dismissViewMenu}
	onfocusin={dismissViewMenu}
	onkeydown={handleViewKey}
/>

<div class="inventory-toolbar">
	<div class="inventory-toolbar__row site-container">
		<Action
			variant="strong"
			class="inventory-toolbar__all"
			aria-haspopup="dialog"
			aria-expanded={allOpen}
			onclick={openFilters}
			><SlidersHorizontal size={18} aria-hidden="true" />{english
				? 'All filters'
				: 'Всички филтри'}</Action
		>
		<div class="inventory-toolbar__filters">
			{#each quickFilters as filter (filter.id)}<InventoryFilter {filter} {english} />{/each}
		</div>
		<form action={linkHref('/inventory')} class="inventory-toolbar__sort">
			{#each [...page.url.searchParams].filter(([name]) => name !== 'sort') as [name, value], i (i)}<input
					type="hidden"
					{name}
					{value}
				/>{/each}
			<label
				><span class="sr-only">{desktop.sortLabel}</span><select
					name="sort"
					value={desktop.sortOptions.find((option) => option.active)?.value ??
						desktop.sortOptions[0]?.value}
					onchange={(event) => event.currentTarget.form?.requestSubmit()}
					>{#each desktop.sortOptions as option (option.value)}<option value={option.value}
							>{option.label}</option
						>{/each}</select
				></label
			>
			<noscript><button type="submit">{english ? 'Sort' : 'Подреди'}</button></noscript>
		</form>
		<details class="inventory-view" bind:this={viewMenu}>
			<summary><LayoutGrid size={18} aria-hidden="true" />{desktop.viewLabel}</summary>
			<nav aria-label={desktop.viewLabel}>
				{#each desktop.viewOptions as option (option.view)}<a
						href={linkHref(option.href)}
						aria-current={option.active ? 'true' : undefined}>{option.label}</a
					>{/each}
			</nav>
		</details>
	</div>
	{#if desktop.activeFilters}<div class="site-container inventory-toolbar__active">
			{#each desktop.activeFilters.chips as chip (chip.href)}<a href={linkHref(chip.href)}
					>{chip.label}<X size={14} aria-hidden="true" /></a
				>{/each}<a href={linkHref(desktop.activeFilters.clearHref)}
				>{desktop.activeFilters.clearLabel}</a
			>
		</div>{/if}
</div>
<Modal
	bind:open={allOpen}
	variant="filter"
	onBack={activeFilter ? backToFilters : undefined}
	backLabel={english ? 'All filters' : 'Всички филтри'}
	onOpenAutoFocus={(event) => {
		event.preventDefault();
		keywordInput?.focus({ preventScroll: true });
	}}
	title={activeFilter?.label ?? (english ? 'Find a car' : 'Търсене на автомобили')}
	onEscapeKeydown={(event) => {
		if (activeFilter) {
			event.preventDefault();
			void backToFilters();
		}
	}}
	wide
	class={[
		'inventory-filters-dialog desktop-filter-dialog',
		activeFilter && 'inventory-filters-dialog--options'
	]
		.filter(Boolean)
		.join(' ')}
>
	{#if activeFilter && currentFilter}
		{#key activeFilter.id}<DesktopFilterPicker
				bind:this={picker}
				filter={currentFilter}
				{english}
				bind:selection={
					() => draft[currentFilter.id] ?? [], (values) => updateSelection(currentFilter, values)
				}
			/>{/key}
	{/if}

	<form
		style:display={activeFilter ? 'none' : undefined}
		class="inventory-all__form"
		bind:this={filterForm}
		id={formId}
		action={linkHref('/inventory')}
		onsubmit={() => (allOpen = false)}
	>
		{#each passthrough as [name, value], i (i)}<input type="hidden" {name} {value} />{/each}
		<div class="inventory-all__search filter-control">
			<Search size={20} aria-hidden="true" />
			<label class="sr-only" for={formId + '-keyword'}
				>{english ? 'Make, model or keyword' : 'Марка, модел или ключова дума'}</label
			>
			<input
				bind:this={keywordInput}
				id={formId + '-keyword'}
				type="search"
				name="keyword"
				bind:value={keyword}
				placeholder={english ? 'Make, model or keyword' : 'Марка, модел или ключова дума'}
			/>
		</div>
		<div class="inventory-all">
			{#each orderedFilters as filter (filter.id)}<InventoryCompactField
					{filter}
					onchoose={(trigger) => chooseFilter(filter, trigger)}
					bind:selection={() => draft[filter.id] ?? [], (values) => updateSelection(filter, values)}
				/>{/each}
		</div>
	</form>
	{#snippet footer()}
		<div class="inventory-all__actions">
			<span class="sr-only" aria-live="polite"
				>{counting
					? ''
					: resultCount === null
						? ''
						: english
							? `${resultCount} matching cars`
							: `${resultCount} намерени автомобила`}</span
			>
			{#if !activeFilter && (keyword || Object.values(draft).some((values) => values.length))}
				<Action
					variant="quiet"
					onclick={() => {
						keyword = '';
						draft = Object.fromEntries(desktop.filters.map((filter) => [filter.id, []]));
					}}>{desktop.sidebar.actions.clearLabel}</Action
				>
			{/if}
			<Action class="inventory-all__apply" onclick={applyFilters} aria-busy={counting}
				>{english ? 'Show cars' : 'Покажи автомобили'}<span
					class="inventory-all__count"
					aria-hidden="true">{counting ? '…' : (resultCount ?? '')}</span
				></Action
			>
		</div>
		{#if resultCount === 0}<p class="inventory-all__empty">
				{english
					? 'No cars match these filters. Try a different make or a higher budget.'
					: 'Няма автомобили с тези филтри. Опитайте друга марка или по-висок бюджет.'}
			</p>{/if}
	{/snippet}
</Modal>

<style>
	.inventory-toolbar {
		background: var(--bc-surface-raised);
		border-bottom: 1px solid var(--bc-border);
		padding-block: var(--bc-space-3);
	}
	.inventory-toolbar__row {
		display: grid;
		grid-template-columns: auto minmax(0, 1fr) auto auto;
		align-items: center;
		gap: var(--bc-space-3);
	}
	.inventory-toolbar__filters {
		display: flex;
		flex-wrap: wrap;
		min-width: 0;
		gap: var(--bc-space-2);
	}
	.inventory-toolbar__sort {
		margin: 0;
	}
	select,
	summary {
		border: 1px solid var(--bc-border);
		border-radius: var(--bc-radius-pill);
		min-height: var(--bc-route-pill-height);
		padding: 0 var(--bc-space-3);
		background: var(--bc-surface-raised);
		color: var(--bc-ink);
		font-size: var(--bc-text-filter);
	}
	.inventory-view {
		position: relative;
	}
	summary {
		display: flex;
		align-items: center;
	}
	.inventory-view nav {
		position: absolute;
		right: 0;
		top: calc(100% + 8px);
		z-index: 10;
		min-width: 160px;
		padding: var(--bc-space-2);
		border: 1px solid var(--bc-border);
		border-radius: var(--bc-radius-control);
		background: var(--bc-surface-raised);
		box-shadow: var(--bc-shadow-panel);
		display: grid;
	}
	.inventory-view a {
		padding: var(--bc-space-2);
		text-decoration: none;
		color: var(--bc-ink);
	}
	.inventory-view a:hover {
		background: var(--bc-surface);
	}
	.inventory-toolbar__active {
		display: flex;
		flex-wrap: wrap;
		gap: var(--bc-space-2);
		padding-top: var(--bc-space-3);
	}
	.inventory-toolbar__active a {
		display: inline-flex;
		align-items: center;
		gap: var(--bc-space-2);
		color: var(--bc-copy);
		font-size: var(--bc-text-label);
		text-decoration: none;
		padding: var(--bc-space-1) var(--bc-space-2);
		border-radius: var(--bc-radius-sm);
		background: var(--bc-surface);
	}
	.inventory-all {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: var(--bc-space-3);
		padding-top: var(--bc-space-4);
	}
	:global(.site-dialog.inventory-filters-dialog) {
		width: min(760px, calc(100vw - 2 * var(--bc-space-6)));
	}
	.inventory-all__form {
		padding: var(--bc-space-1);
	}
	.inventory-all__actions {
		display: flex;
		justify-content: space-between;
		gap: var(--bc-space-3);
	}

	.inventory-all__actions :global(.inventory-all__apply) {
		margin-left: auto;
	}
	.inventory-all__count {
		min-width: 2ch;
		text-align: center;
		font-variant-numeric: tabular-nums;
		opacity: 0.85;
	}
	.inventory-all__empty {
		margin: var(--bc-space-3) 0 0;
		color: var(--bc-copy);
		font-size: var(--bc-text-label);
		text-align: right;
	}
	.inventory-toolbar {
		position: sticky;
		top: 0;
		z-index: 80;
		box-shadow: var(--bc-shadow-subtle);
	}
	.inventory-toolbar__row :global(.inventory-toolbar__all) {
		min-height: var(--bc-route-pill-height);
		border-radius: var(--bc-radius-pill);
		white-space: nowrap;
	}
	.inventory-toolbar__filters :global(.site-filter-trigger) {
		flex: 0 0 auto;
	}
	summary {
		gap: var(--bc-space-2);
		list-style: none;
	}
	summary::-webkit-details-marker {
		display: none;
	}
	.inventory-view nav {
		border-radius: var(--bc-radius-panel);
	}
	@media (max-width: 1399px) {
		.inventory-toolbar__row {
			grid-template-columns: 1fr auto auto;
		}
		.inventory-toolbar__filters {
			grid-row: 2;
			grid-column: 1/-1;
		}
		.inventory-toolbar__row :global(.inventory-toolbar__all) {
			justify-self: start;
		}
	}
	@media (min-width: 768px) {
		select,
		summary,
		.inventory-toolbar__row :global(.inventory-toolbar__all) {
			min-height: var(--bc-control-height-standard);
			font-size: var(--bc-text-control);
		}
		.inventory-toolbar__active a {
			min-height: var(--bc-control-height-compact);
			padding-inline: var(--bc-space-3);
		}
		.inventory-view a[aria-current='true'] {
			background: var(--bc-bg-strong);
			border-radius: var(--bc-radius-md);
			font-weight: var(--bc-weight-heading);
		}
	}
	@media (max-width: 599px) {
		.inventory-all {
			grid-template-columns: 1fr;
		}
	}
	@media (min-width: 600px) and (max-width: 899px) {
		.inventory-all {
			grid-template-columns: repeat(2, minmax(0, 1fr));
		}
	}
</style>
