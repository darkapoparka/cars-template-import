<script lang="ts">
	import Modal from '$lib/components/common/Modal.svelte';
	import Action from '$lib/components/common/Action.svelte';
	import { assetHref } from '$lib/utils/assets';
	import ChevronRight from '@lucide/svelte/icons/chevron-right';
	import type { AuxeroInventoryFilter } from '$lib/server/inventory-options';
	import { inventoryFilterParam } from '$lib/domain/inventory-query';
	let {
		filter,
		english = false,
		selection = $bindable<string[]>([])
	}: {
		filter: AuxeroInventoryFilter;
		english?: boolean;
		selection?: string[];
	} = $props();
	const id = $props.id();
	let query = $state('');
	let open = $state(false);
	let searchInput = $state<HTMLInputElement>();
	const matching = $derived(
		filter.options.filter((option) =>
			option.label.toLocaleLowerCase().includes(query.trim().toLocaleLowerCase())
		)
	);
	const summary = $derived(
		selection
			.map((value) => filter.options.find((option) => option.value === value)?.label ?? value)
			.join(', ')
	);
	function toggle(value: string) {
		selection =
			filter.mode === 'single'
				? selection.includes(value)
					? []
					: [value]
				: selection.includes(value)
					? selection.filter((item) => item !== value)
					: [...selection, value];
	}
</script>

<div
	class={[
		'compact-field',
		filter.name === 'feature' && filter.options.length > 1 && 'compact-field--features'
	]}
>
	{#each selection as value (value)}<input
			type="hidden"
			name={inventoryFilterParam(filter.name)}
			{value}
		/>{/each}
	{#if filter.name === 'feature' && filter.options.length === 1}
		<label class="compact-field__feature"
			><input
				type="checkbox"
				checked={selection.includes(filter.options[0].value)}
				onchange={() => toggle(filter.options[0].value)}
			/><span class="compact-field__text"
				><span class="sr-only">{filter.label}</span><span class="compact-field__value"
					>{filter.options[0].label}</span
				></span
			></label
		>
	{:else if filter.name === 'feature'}
		<fieldset class="compact-features">
			<legend>{filter.label}</legend>
			<div>
				{#each filter.options as option (option.value)}<label
						><input
							type="checkbox"
							checked={selection.includes(option.value)}
							onchange={() => toggle(option.value)}
						/><span>{option.label}</span></label
					>{/each}
			</div>
		</fieldset>
	{:else if filter.numericInput}
		<div class="compact-field__number">
			<div class="compact-field__text">
				<label class="compact-field__label" class:sr-only={!selection.length} for={id}
					>{filter.label}</label
				>
				<input
					{id}
					type="number"
					aria-label={filter.numericInput.label + ' (' + filter.numericInput.unit + ')'}
					min="1"
					step="1"
					inputmode="numeric"
					placeholder={filter.numericInput.label}
					value={selection[0] ?? ''}
					list={id + '-presets'}
					oninput={(event) => {
						const value = event.currentTarget.valueAsNumber;
						selection = Number.isFinite(value) ? [String(value)] : [];
					}}
				/>
			</div>
			<span class="compact-field__unit">{filter.numericInput.unit}</span>
		</div>
		<datalist id={id + '-presets'}
			>{#each filter.options as option (option.value)}<option value={option.value}
					>{option.label}</option
				>{/each}</datalist
		>
	{:else}
		<button
			type="button"
			class="compact-field__trigger"
			aria-labelledby={id + '-label ' + id + '-value'}
			aria-haspopup="dialog"
			aria-expanded={open}
			onclick={() => {
				query = '';
				open = true;
			}}
		>
			<span class="compact-field__text"
				><span id={id + '-label'} class="compact-field__label">{filter.label}</span><span
					class="compact-field__value"
					class:sr-only={!summary}
					id={id + '-value'}>{summary || filter.allLabel}</span
				></span
			><ChevronRight size={16} aria-hidden="true" />
		</button>
		<Modal
			bind:open
			class="filter-picker-dialog compact-field__dialog"
			title={filter.label}
			onOpenAutoFocus={(event) => {
				event.preventDefault();
				searchInput?.focus({ preventScroll: true });
			}}
		>
			{#snippet headerContent()}
				<div class="compact-field__search-wrap">
					<input
						bind:this={searchInput}
						class="compact-field__search"
						type="search"
						bind:value={query}
						aria-label={(english ? 'Search in ' : 'Търси в ') + filter.label}
						placeholder={english ? 'Search options' : 'Търси в опциите'}
					/>
				</div>
			{/snippet}
			<div class="compact-field__options">
				{#each matching as option (option.value)}
					<label
						><input
							type={filter.mode === 'single' ? 'radio' : 'checkbox'}
							name={id + '-option'}
							checked={selection.includes(option.value)}
							onchange={() => toggle(option.value)}
						/>{#if option.image}<img
								src={assetHref(option.image)}
								alt=""
								width="36"
								height="28"
							/>{/if}<span>{option.label}</span></label
					>
				{:else}<p role="status">{english ? 'No matches' : 'Няма съвпадения'}</p>{/each}
			</div>
			{#snippet footer()}
				<div class="compact-field__actions">
					<Action variant="secondary" onclick={() => (selection = [])}
						>{english ? 'Clear' : 'Изчисти'}</Action
					>
					<Action onclick={() => (open = false)}>{english ? 'Done' : 'Готово'}</Action>
				</div>
			{/snippet}
		</Modal>
	{/if}
</div>

<style>
	.compact-field > .compact-field__feature {
		display: flex;
		align-items: center;
		gap: var(--bc-space-2);
		min-height: var(--bc-control-height-primary);
		padding: var(--bc-space-2) var(--bc-space-3);
		border: 1px solid var(--bc-border);
		border-radius: var(--bc-radius-md);
		background: var(--bc-surface);
		color: var(--bc-ink);
		font-size: var(--bc-text-filter);
		font-weight: normal;
		cursor: pointer;
	}
	.compact-field__feature input {
		accent-color: var(--bc-accent);
	}
	.compact-field--features {
		grid-column: 1 / -1;
	}
	.compact-features {
		margin: 0;
		min-width: 0;
		padding: var(--bc-space-5);
		border: 0;
		border-radius: var(--bc-radius-card);
		background: var(--bc-surface);
	}
	.compact-features legend {
		float: left;
		width: 100%;
		padding: 0 0 var(--bc-space-4);
		font-weight: var(--bc-weight-heading);
	}
	.compact-features > div {
		clear: both;
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(min(100%, 14rem), 1fr));
		gap: var(--bc-space-2);
	}
	.compact-features label {
		display: flex;
		align-items: center;
		gap: var(--bc-space-2);
		padding: var(--bc-space-3);
		min-height: var(--bc-control-height-standard);
		background: var(--bc-surface-raised);
		border-radius: var(--bc-radius-md);
		font-size: var(--bc-text-label);
		cursor: pointer;
	}
	.compact-features input {
		accent-color: var(--bc-accent);
	}
	.compact-field {
		display: grid;
		gap: var(--bc-space-2);
		min-width: 0;
		align-content: start;
	}
	.compact-field__label {
		color: var(--bc-ink);
		font-size: var(--bc-text-control);
		font-weight: var(--bc-weight-control);
	}
	.compact-field :global(.compact-field__trigger),
	.compact-field__number {
		display: flex;
		align-items: center;
		gap: var(--bc-space-2);
		width: 100%;
		min-height: var(--bc-control-height-primary);
		padding: var(--bc-space-2) var(--bc-space-3);
		border: 1px solid var(--bc-route-pill-border);
		border-radius: var(--bc-radius-md);
		background: var(--bc-surface);
		color: var(--bc-ink);
		font-size: var(--bc-text-filter);
	}
	.compact-field :global(.compact-field__trigger) {
		justify-content: space-between;
		text-align: left;
	}
	.compact-field__text {
		display: flex;
		align-items: center;
		line-height: var(--bc-leading-label);
		gap: var(--bc-space-2);
		min-width: 0;
		flex: 1;
	}
	.compact-field__value {
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: var(--bc-text-label);
		color: var(--bc-copy);
	}
	.compact-field :global(.compact-field__trigger > svg) {
		flex: none;
	}
	.compact-field__number input {
		width: 100%;
		min-width: 0;
		padding: 0;
		border: 0;
		background: transparent;
		color: inherit;
		font: inherit;
		font-size: var(--bc-text-control);
	}
	.compact-field__number input::placeholder {
		color: var(--bc-ink);
		opacity: 1;
	}
	.compact-field__label {
		flex-shrink: 0;
	}
	.compact-field__feature .compact-field__value {
		font-size: var(--bc-text-control);
		color: var(--bc-ink);
	}
	.compact-field__unit {
		color: var(--bc-muted);
		font-size: var(--bc-text-label);
	}
	.compact-field__number:focus-within {
		outline: 2px solid var(--bc-focus);
		outline-offset: 2px;
	}
	.compact-field__number input:focus-visible {
		outline: none !important;
		box-shadow: none !important;
	}
	.compact-field__search {
		width: 100%;
		min-height: var(--bc-control-height-standard);
		padding: var(--bc-space-2) var(--bc-space-3);
		border: 1px solid var(--bc-route-pill-border);
		border-radius: var(--bc-radius-md);
		background: var(--bc-surface);
		color: var(--bc-ink);
		font: inherit;
		font-size: var(--bc-text-control);
	}
	.compact-field__search-wrap {
		padding: var(--bc-space-1) var(--bc-space-1) var(--bc-space-4);
	}
	.compact-field__options img {
		object-fit: contain;
		flex: 0 0 36px;
	}
	.compact-field__options input {
		width: var(--bc-text-control);
		height: var(--bc-text-control);
		margin: 0;
		flex-shrink: 0;
	}
	.compact-field__options {
		min-height: 0;
	}
	.compact-field__options label {
		display: flex;
		align-items: center;
		gap: var(--bc-space-3);
		min-height: var(--bc-control-height-primary);
		padding: var(--bc-space-2);
		font-size: var(--bc-text-control);
		border-radius: var(--bc-radius-md);
		cursor: pointer;
	}
	.compact-field__options label:hover,
	.compact-field__options label:has(:checked) {
		background: var(--bc-surface);
	}
	.compact-field__options input {
		accent-color: var(--bc-accent);
	}
	.compact-field__actions {
		display: flex;
		justify-content: space-between;
	}
</style>
