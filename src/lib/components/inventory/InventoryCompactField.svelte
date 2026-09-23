<script lang="ts">
	import ChevronRight from '@lucide/svelte/icons/chevron-right';
	import type { AuxeroInventoryFilter } from '$lib/server/inventory-options';
	import { inventoryFilterParam } from '$lib/domain/inventory-query';
	let {
		filter,
		selection = $bindable<string[]>([]),
		onchoose
	}: {
		filter: AuxeroInventoryFilter;
		selection?: string[];
		onchoose: (trigger: HTMLButtonElement) => void;
	} = $props();
	const id = $props.id();
	const selectedLabels = $derived(
		selection.map(
			(value) => filter.options.find((option) => option.value === value)?.label ?? value
		)
	);
	const summary = $derived(selectedLabels.join(', '));
	const visibleSummary = $derived(
		selectedLabels.length > 1 ? `${selectedLabels[0]} +${selectedLabels.length - 1}` : summary
	);
</script>

<div class="compact-field">
	{#each selection as value (value)}<input
			type="hidden"
			name={inventoryFilterParam(filter.name)}
			{value}
		/>{/each}
	{#if filter.numericInput}
		<label class="compact-field__number filter-control" for={id}>
			<span class="compact-field__text">
				<span class="compact-field__label">{filter.numericInput.label}</span>
				<input
					{id}
					type="number"
					aria-label={filter.numericInput.label + ' (' + filter.numericInput.unit + ')'}
					min="1"
					step="1"
					inputmode="numeric"
					placeholder="—"
					value={selection[0] ?? ''}
					oninput={(event) => {
						const value = event.currentTarget.valueAsNumber;
						selection = Number.isFinite(value) ? [String(value)] : [];
					}}
				/>
			</span>
			<span class="compact-field__unit">{filter.numericInput.unit}</span>
		</label>
	{:else}
		<button
			type="button"
			class="compact-field__trigger filter-control"
			aria-labelledby={id + '-label ' + id + '-value'}
			title={summary || undefined}
			class:compact-field__trigger--selected={selection.length > 0}
			onclick={(event) => onchoose(event.currentTarget)}
		>
			<span class="compact-field__text"
				><span id={id + '-label'} class="compact-field__label">{filter.label}</span><span
					class="compact-field__value"
					class:sr-only={!summary}
					aria-hidden="true">{visibleSummary}</span
				></span
			><ChevronRight size={16} aria-hidden="true" />
			<span class="sr-only" id={id + '-value'}>{summary || filter.allLabel}</span>
		</button>
	{/if}
</div>

<style>
	.compact-field {
		display: grid;
		gap: var(--bc-space-2);
		min-width: 0;
		align-content: start;
	}
	.compact-field :global(.filter-control) {
		min-height: var(--bc-control-height-primary);
	}
	.compact-field__label {
		color: var(--bc-ink);
		font-size: var(--bc-text-control);
		font-weight: var(--bc-weight-control);
	}
	.compact-field :global(.compact-field__trigger) {
		justify-content: space-between;
		text-align: left;
	}
	.compact-field__text {
		display: flex;
		align-items: center;
		line-height: var(--bc-leading-label);
		gap: var(--bc-space-3);
		min-width: 0;
		flex: 1;
	}
	.compact-field__value {
		margin-left: auto;
		min-width: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: var(--bc-text-control);
		color: var(--bc-ink);
		font-weight: var(--bc-weight-control);
	}
	.compact-field :global(.compact-field__trigger > svg) {
		flex: none;
	}
	.compact-field__number input {
		text-align: right;
	}
	.compact-field__number {
		cursor: text;
	}
	.compact-field__number input::placeholder {
		color: var(--bc-copy);
		opacity: 1;
	}
	.compact-field__label {
		flex-shrink: 0;
	}
	.compact-field__unit {
		color: var(--bc-muted);
		font-size: var(--bc-text-label);
	}
	.compact-field__trigger--selected .compact-field__label {
		color: var(--bc-copy);
	}
</style>
