<script lang="ts">
	import { tick } from 'svelte';
	import { SvelteURLSearchParams } from 'svelte/reactivity';
	import DesktopFilterRange from './DesktopFilterRange.svelte';
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
	let cleared = $state(false);
	let draft = $state<Record<string, string[]>>({});
	let keyword = $state('');
	let minimums = $state<Record<string, string>>({});
	let rangePicker = $state<DesktopFilterRange>();
	const minimumParam = (filter: AuxeroInventoryFilter) =>
		filter.name === 'priceTo' ? 'minPrice' : 'minMileage';
	let activeFilter = $state<AuxeroInventoryFilter | null>(null);
	let picker = $state<DesktopFilterPicker>();
	let keywordInput = $state<HTMLInputElement>();
	let filterForm = $state<HTMLFormElement>();
	let filterPanel = $state<HTMLDivElement>();
	$effect(() => {
		// Each category starts at its heading, even after scrolling a long model list.
		void activeFilter?.id;
		if (filterPanel) filterPanel.scrollTop = 0;
	});
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
			if (filter.numericInput && minimums[filter.id])
				params.set(minimumParam(filter), minimums[filter.id]);
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
		if (rangePicker && !rangePicker.validate()) return;
		const invalid = draftFilters.find(
			(filter) =>
				filter.numericInput &&
				([...(draft[filter.id] ?? []), ...(minimums[filter.id] ? [minimums[filter.id]] : [])].some(
					(value) => !Number.isInteger(Number(value)) || Number(value) < 1
				) ||
					Boolean(
						minimums[filter.id] &&
						draft[filter.id]?.[0] &&
						Number(minimums[filter.id]) > Number(draft[filter.id][0])
					))
		);
		if (invalid) {
			activeFilter = invalid;
			await tick();
			rangePicker?.validate();
			return;
		}
		filterForm?.submit();
	}
	function chooseFilter(filter: AuxeroInventoryFilter | null) {
		activeFilter = filter;
	}
	async function navigateFilters(event: KeyboardEvent, index: number) {
		const keys = ['ArrowDown', 'ArrowUp', 'Home', 'End'];
		if (!keys.includes(event.key)) return;
		event.preventDefault();
		const tabs = [null, ...orderedFilters];
		const next =
			event.key === 'Home'
				? 0
				: event.key === 'End'
					? tabs.length - 1
					: (index + (event.key === 'ArrowDown' ? 1 : -1) + tabs.length) % tabs.length;
		activeFilter = tabs[next];
		await tick();
		filterForm?.querySelector<HTMLButtonElement>('[role="tab"][aria-selected="true"]')?.focus();
	}
	function selectionSummary(filter: AuxeroInventoryFilter) {
		const values = draft[filter.id] ?? [];
		if (filter.numericInput) {
			const min = minimums[filter.id];
			const format = (value: string) => Number(value).toLocaleString(english ? 'en' : 'bg');
			if (!min && !values.length) return '';
			const range =
				min && values[0]
					? `${format(min)} – ${format(values[0])}`
					: min
						? `${english ? 'From' : 'От'} ${format(min)}`
						: `${english ? 'Up to' : 'До'} ${format(values[0])}`;
			return `${range} ${filter.numericInput.unit}`;
		}
		return values
			.map((value) => filter.options.find((option) => option.value === value)?.label ?? value)
			.join(', ');
	}
	function appliedRangeSummary(filter: AuxeroInventoryFilter) {
		if (!filter.numericInput) return undefined;
		const min = page.url.searchParams.get(minimumParam(filter));
		if (!min) return undefined;
		const format = (value: string) => Number(value).toLocaleString(english ? 'en' : 'bg');
		return `${filter.selectedValues[0] ? format(min) + ' – ' + format(filter.selectedValues[0]) : (english ? 'From ' : 'От ') + format(min)} ${filter.numericInput.unit}`;
	}
	function openFilters(filter?: AuxeroInventoryFilter) {
		cleared = false;
		activeFilter = filter ?? desktop.filters[0];
		draft = Object.fromEntries(
			desktop.filters.map((filter) => [filter.id, [...filter.selectedValues]])
		);
		keyword =
			page.url.searchParams.get('keyword') ??
			(!desktop.filters.find((filter) => filter.name === 'model')?.selectedValues.length
				? (page.url.searchParams.get('q') ?? '')
				: '');
		minimums = Object.fromEntries(
			desktop.filters
				.filter((filter) => filter.numericInput)
				.map((filter) => [filter.id, page.url.searchParams.get(minimumParam(filter)) ?? ''])
		);
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
			...serializeInventoryQuery(
				{ ...parseInventoryQuery(page.url.searchParams), ...(cleared ? { filters: {} } : {}) },
				page.url.searchParams
			)
		].filter(
			([name]) =>
				!fieldNames.has(name) && !['keyword', 'page', 'minPrice', 'minMileage'].includes(name)
		)
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
			onclick={() => openFilters()}
			><SlidersHorizontal size={18} aria-hidden="true" />{english
				? 'All filters'
				: 'Всички филтри'}</Action
		>
		<div class="inventory-toolbar__filters">
			{#each quickFilters as filter (filter.id)}<InventoryFilter
					{filter}
					summary={appliedRangeSummary(filter)}
					expanded={allOpen && activeFilter?.id === filter.id}
					onopen={() => openFilters(filter)}
				/>{/each}
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
	onOpenAutoFocus={(event) => {
		event.preventDefault();
		(currentFilter?.numericInput ? rangePicker : picker)?.focusSearch();
	}}
	title={english ? 'Find a car' : 'Търсене на автомобили'}
	description={
		english
			? 'Choose a few details to narrow down the cars in stock.'
			: 'Изберете критерии, за да стесните наличните автомобили.'
	}
	wide
	class="inventory-filters-dialog desktop-filter-dialog"
>
	<form
		class="inventory-all__form"
		bind:this={filterForm}
		id={formId}
		action={linkHref('/inventory')}
		onsubmit={(event) => {
			event.preventDefault();
			void applyFilters();
		}}
	>
		{#each passthrough as [name, value], i (i)}<input type="hidden" {name} {value} />{/each}
		<input type="hidden" name="keyword" value={keyword} />
		{#each desktop.filters.filter((filter) => filter.numericInput && minimums[filter.id]) as filter (filter.id)}<input
				type="hidden"
				name={minimumParam(filter)}
				value={minimums[filter.id]}
			/>{/each}
		{#each desktop.filters as filter (filter.id)}{#each draft[filter.id] ?? [] as value (value)}<input
					type="hidden"
					name={inventoryFilterParam(filter.name)}
					{value}
				/>{/each}{/each}
		<div
			class="inventory-all__navigation"
			role="tablist"
			aria-orientation="vertical"
			aria-label={english ? 'Filters' : 'Филтри'}
		>
			<button
				type="button"
				role="tab"
				id={formId + '-keyword-tab'}
				aria-selected={!activeFilter}
				aria-controls={formId + '-panel'}
				tabindex={!activeFilter ? 0 : -1}
				onclick={() => chooseFilter(null)}
				onkeydown={(event) => navigateFilters(event, 0)}
			>
				<span>{english ? 'Search' : 'Търсене'}</span>{#if keyword}<small>{keyword}</small>{/if}
			</button>
			{#each orderedFilters as filter, index (filter.id)}
				<button
					type="button"
					role="tab"
					id={formId + '-' + filter.id}
					aria-selected={activeFilter?.id === filter.id}
					aria-controls={formId + '-panel'}
					tabindex={activeFilter?.id === filter.id ? 0 : -1}
					onclick={() => chooseFilter(filter)}
					onkeydown={(event) => navigateFilters(event, index + 1)}
					title={selectionSummary(filter) || undefined}
				>
					<span
						>{filter.label}{#if draft[filter.id]?.length || minimums[filter.id]}<span
								class="inventory-all__selected"
								aria-hidden="true"
							></span>{/if}</span
					>
					{#if draft[filter.id]?.length || minimums[filter.id]}<small
							>{selectionSummary(filter)}</small
						>{/if}
				</button>
			{/each}
		</div>
		<div
			class="inventory-all__panel"
			bind:this={filterPanel}
			role="tabpanel"
			id={formId + '-panel'}
			aria-labelledby={activeFilter ? formId + '-' + activeFilter.id : formId + '-keyword-tab'}
			tabindex="0"
		>
			<h2>
				{currentFilter?.label ?? (english ? 'Search' : 'Търсене')}
			</h2>
			{#if currentFilter}
				{#key currentFilter.id}{#if currentFilter.numericInput}<DesktopFilterRange
							bind:this={rangePicker}
							filter={currentFilter}
							{english}
							bind:minimum={minimums[currentFilter.id]}
							bind:selection={
								() => draft[currentFilter.id] ?? [],
								(values) => updateSelection(currentFilter, values)
							}
						/>{:else}<DesktopFilterPicker
							bind:this={picker}
							filter={currentFilter}
							{english}
							bind:selection={
								() => draft[currentFilter.id] ?? [],
								(values) => updateSelection(currentFilter, values)
							}
						/>{/if}{/key}
			{:else}
				<label class="inventory-all__search filter-control">
					<Search size={20} aria-hidden="true" /><span class="sr-only"
						>{english ? 'Make, model or keyword' : 'Марка, модел или ключова дума'}</span
					>
					<input
						bind:this={keywordInput}
						type="search"
						bind:value={keyword}
						placeholder={english ? 'Make, model or keyword' : 'Марка, модел или ключова дума'}
					/>
				</label>
			{/if}
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
			{#if (!cleared && desktop.activeFilters) || keyword || Object.values(draft).some((values) => values.length) || Object.values(minimums).some(Boolean)}
				<Action
					variant="quiet"
					onclick={() => {
						keyword = '';
						cleared = true;
						minimums = {};
						draft = Object.fromEntries(desktop.filters.map((filter) => [filter.id, []]));
					}}>{desktop.sidebar.actions.clearLabel}</Action
				>
			{/if}
			{#if resultCount === 0}<p class="inventory-all__empty">
					{english
						? 'No cars match. Adjust or clear your filters.'
						: 'Няма автомобили. Променете или изчистете филтрите.'}
				</p>{/if}
			<Action class="inventory-all__apply" type="submit" form={formId} aria-busy={counting}
				>{english ? 'Show cars' : 'Покажи автомобили'}<span
					class="inventory-all__count"
					aria-hidden="true">{counting ? '…' : (resultCount ?? '')}</span
				></Action
			>
		</div>
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
	:global(.site-dialog.inventory-filters-dialog) {
		width: min(1080px, calc(100vw - 2 * var(--bc-space-6)));
		height: min(600px, calc(100dvh - 2 * var(--bc-space-6)));
	}
	@media (min-width: 900px) {
		:global(.site-dialog.inventory-filters-dialog .site-dialog__header) {
			align-items: center;
			padding-block: var(--bc-space-4);
			border-bottom: 1px solid var(--bc-border);
		}
		:global(.site-dialog.inventory-filters-dialog .site-dialog__footer) {
			border-top: 1px solid var(--bc-border);
			background: var(--bc-surface-raised);
		}
		.inventory-all__navigation {
			padding-inline: var(--bc-space-3);
			border-right: 1px solid var(--bc-border);
			scrollbar-color: var(--bc-border-strong) var(--bc-surface);
			padding-block: var(--bc-space-2);
		}
		.inventory-all__panel {
			scrollbar-color: var(--bc-border-strong) var(--bc-surface-raised);
		}
	}
	:global(.site-dialog.inventory-filters-dialog .site-dialog__body) {
		padding: 0;
		overflow: hidden;
		flex: 1;
	}
	.inventory-all__form {
		display: grid;
		grid-template-columns: 212px minmax(0, 1fr);
		height: 100%;
		min-height: 0;
	}
	.inventory-all__navigation {
		overflow-y: auto;
		padding: var(--bc-space-3);
		background: var(--bc-surface);
		scrollbar-width: thin;
	}
	.inventory-all__navigation button {
		display: block;
		width: 100%;
		border: 0;
		border-radius: var(--bc-radius-md);
		padding: var(--bc-space-2) var(--bc-space-4);
		background: transparent;
		color: var(--bc-ink);
		text-align: left;
		font: inherit;
		font-size: var(--bc-text-control);
		cursor: pointer;
	}
	.inventory-all__navigation button:hover {
		background: var(--bc-bg-strong);
	}
	.inventory-all__navigation button[aria-selected='true'] {
		background: var(--bc-ink);
		color: var(--bc-surface-raised);
		font-weight: var(--bc-weight-heading);
	}
	.inventory-all__navigation button[aria-selected='true'] small {
		color: inherit;
	}
	.inventory-all__navigation button[aria-selected='true'] .inventory-all__selected {
		background: currentColor;
	}
	.inventory-all__navigation button > span {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: var(--bc-space-2);
	}
	.inventory-all__navigation small {
		display: block;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		color: var(--bc-copy);
		font-size: var(--bc-text-label);
		font-weight: var(--bc-weight-control);
		margin-top: var(--bc-space-1);
	}
	.inventory-all__selected {
		width: var(--bc-space-2);
		height: var(--bc-space-2);
		border-radius: var(--bc-radius-pill);
		background: var(--bc-accent);
		flex: none;
	}
	.inventory-all__panel {
		display: flex;
		flex-direction: column;
		min-width: 0;
		min-height: 0;
		padding: var(--bc-space-5) var(--bc-space-6);
		overflow: auto;
		overscroll-behavior: contain;
		scrollbar-gutter: stable;
		scrollbar-width: thin;
	}
	.inventory-all__panel h2 {
		margin: 0 0 var(--bc-space-4);
		font: var(--bc-weight-heading) var(--bc-text-h4)/var(--bc-leading-h4) var(--bc-font-body);
	}
	@media (max-width: 899px) {
		.inventory-all__form {
			grid-template-columns: 184px minmax(0, 1fr);
		}
		.inventory-all__panel {
			padding-inline: var(--bc-space-4);
		}
	}

	.inventory-all__actions {
		display: flex;
		align-items: center;
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
		margin: 0;
		color: var(--bc-copy);
		font-size: var(--bc-text-label);
		text-align: left;
		flex: 1;
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
		:global(.site-dialog.inventory-filters-dialog) {
			width: calc(100vw - 2 * var(--bc-space-4));
			height: min(760px, calc(100dvh - 2 * var(--bc-space-4)));
		}
		.inventory-all__form {
			grid-template-columns: minmax(0, 1fr);
			grid-template-rows: auto minmax(0, 1fr);
		}
		.inventory-all__navigation {
			display: grid;
			grid-template-columns: repeat(3, minmax(0, 1fr));
			gap: var(--bc-space-1);
			max-height: 156px;
			overflow: auto;
			border-right: 0;
			border-bottom: 1px solid var(--bc-border);
			padding: var(--bc-space-2);
		}
		.inventory-all__navigation button {
			padding: var(--bc-space-1) var(--bc-space-2);
			text-align: center;
		}
		.inventory-all__navigation button > span {
			justify-content: center;
		}
		.inventory-all__navigation small {
			display: none;
		}
		.inventory-all__panel {
			padding: var(--bc-space-4);
		}
	}
	@media (min-width: 900px) {
		.inventory-all__navigation button {
			position: relative;
			min-height: var(--bc-control-height-compact);
			border: 1px solid transparent;
			padding-inline: var(--bc-space-3);
			padding-block: var(--bc-space-1);
			transition:
				background-color 140ms ease,
				border-color 140ms ease;
		}
		.inventory-all__navigation button:hover {
			background: var(--bc-bg-strong);
		}
		.inventory-all__navigation button[aria-selected='true'] {
			border-color: color-mix(in srgb, var(--bc-accent) 24%, var(--bc-border));
			background: var(--bc-surface-raised);
			color: var(--bc-ink);
			box-shadow: var(--bc-shadow-subtle);
		}
		.inventory-all__navigation button[aria-selected='true']::before {
			position: absolute;
			inset-block: 9px;
			left: -1px;
			width: 3px;
			border-radius: var(--bc-radius-pill);
			background: var(--bc-accent);
			content: '';
		}
		.inventory-all__navigation button[aria-selected='true'] small {
			color: var(--bc-copy);
		}
		.inventory-all__navigation button[aria-selected='true'] .inventory-all__selected {
			background: var(--bc-accent);
		}
		.inventory-all__navigation button:focus-visible {
			outline: 2px solid var(--bc-focus);
			outline-offset: 2px;
		}
	}
</style>
