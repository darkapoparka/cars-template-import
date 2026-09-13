<script lang="ts">
	import type { HomeFiveVehicleCardData } from '$lib/auxero/home-five';
	import { page } from '$app/state';
	import { templateInquiryCopy } from '$lib/data/template-settings';
	import { importCriteriaFromParams } from '$lib/data/import-criteria';
	import type { AuxeroServiceFormData } from '$lib/auxero/services';
	import { importRequestMobileCopy, importRequestSteps } from '$lib/auxero/services';
	import MobileAppbar from '$lib/components/layout/MobileAppbar.svelte';
	import MobileServiceEntry from '$lib/components/services/MobileServiceEntry.svelte';
	import { ArrowRight, Link2 } from '@lucide/svelte';
	import { Drawer } from 'vaul-svelte';
	import ImportRequestWizard from './ImportRequestWizard.svelte';

	type ImportIntent = 'listing' | 'source';

	let {
		form,
		serviceVehicles
	}: { form: AuxeroServiceFormData; serviceVehicles: HomeFiveVehicleCardData[] } = $props();

	// The route supplies the initial listing value once; the user owns it after hydration.
	// svelte-ignore state_referenced_locally
	let vehicle = $state(form.vehicleField.value ?? '');
	let entryMode = $state<ImportIntent>('listing');
	let wizardOpen = $state(false);
	let wizardIntent = $state<ImportIntent>('listing');
	let wizardSession = $state(0);
	const criteria = $derived(importCriteriaFromParams(page.url.searchParams));

	const openWizard = (intent: ImportIntent) => {
		wizardIntent = intent;
		wizardSession += 1;
		wizardOpen = true;
	};

	const handleVehicleSubmit = (event: SubmitEvent) => {
		event.preventDefault();
		openWizard('listing');
	};
</script>

<div class="daynight-import-mobile">
	<MobileAppbar surface="dark" />

	<MobileServiceEntry
		{serviceVehicles}
		showTitle={false}
		title={importRequestMobileCopy.title}
		intro={importRequestMobileCopy.intro}
		meta="3 стъпки · Без ангажимент"
		response={templateInquiryCopy.response}
		steps={importRequestSteps.map((step, index) =>
			index === 0 ? { ...step, text: 'Изпращаш линк към обявата или директно VIN номера.' } : step
		)}
	>
		{#snippet modes()}
			<div class="import-mode-tabs" role="tablist" aria-label="Начин за заявка">
				<button
					type="button"
					class:active={entryMode === 'listing'}
					role="tab"
					aria-selected={entryMode === 'listing'}
					onclick={() => (entryMode = 'listing')}>LINK / VIN</button
				>
				<button
					type="button"
					class:active={entryMode === 'source'}
					role="tab"
					aria-selected={entryMode === 'source'}
					onclick={() => (entryMode = 'source')}>Нямам линк</button
				>
			</div>
		{/snippet}
		{#snippet entry()}
			{#if entryMode === 'listing'}
				<form onsubmit={handleVehicleSubmit}>
					<label for="import-mobile-vehicle">Линк към обява или VIN</label>
					<div class="service-input">
						<Link2 size={21} strokeWidth={2.15} aria-hidden="true" />
						<input
							id="import-mobile-vehicle"
							name={form.vehicleField.name}
							type={form.vehicleField.type}
							placeholder="Линк към обява или VIN"
							required
							bind:value={vehicle}
						/>
						<button type="submit" aria-label="Провери автомобила">
							<ArrowRight size={21} strokeWidth={2.35} aria-hidden="true" />
						</button>
					</div>
				</form>
			{:else}
				<button class="import-manual-entry" type="button" onclick={() => openWizard('source')}>
					<span class="import-manual-entry__copy">
						<strong>Опиши автомобила</strong>
						<small>Марка, модел, бюджет</small>
					</span>
					<span class="import-manual-entry__go" aria-hidden="true">
						<ArrowRight size={20} strokeWidth={2.35} />
					</span>
				</button>
			{/if}
		{/snippet}
	</MobileServiceEntry>

	<Drawer.Root bind:open={wizardOpen} direction="bottom" fixed={true}>
		<Drawer.Overlay class="daynight-import-wizard-drawer__backdrop">
			<span>Затвори</span>
		</Drawer.Overlay>
		<Drawer.Content class="daynight-import-wizard-drawer__sheet">
			<Drawer.Title class="daynight-import-wizard-drawer__title">Заявка за внос</Drawer.Title>
			{#key wizardSession}
				<ImportRequestWizard
					initialCriteria={criteria}
					initialIntent={wizardIntent}
					initialVehicle={vehicle}
					onclose={() => (wizardOpen = false)}
				/>
			{/key}
		</Drawer.Content>
	</Drawer.Root>
</div>

<style>
	.daynight-import-mobile {
		position: relative;
		min-height: calc(100dvh - var(--bc-mobile-nav-height) - env(safe-area-inset-bottom));
		background: var(--bc-bg-strong);
		color: var(--bc-ink);
	}

	.daynight-import-mobile :global(a) {
		text-decoration: none !important;
	}

	.import-mode-tabs {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: 0;
		min-height: 0;
		border: 0;
		border-bottom: 1px solid rgb(255 255 255 / 0.2);
		border-radius: 0;
		background: transparent;
		box-shadow: none;
		padding: 0;
	}

	.import-mode-tabs button {
		position: relative;
		display: flex;
		width: 100%;
		height: 44px;
		min-height: 44px;
		align-items: center;
		justify-content: center;
		border: 0;
		border-radius: 0;
		background: transparent;
		color: rgb(255 255 255 / 0.72);
		font-family:
			'Geist Variable',
			'Geist',
			'Noto Sans',
			ui-sans-serif,
			system-ui,
			-apple-system,
			BlinkMacSystemFont,
			'Segoe UI',
			Arial,
			sans-serif;
		font-size: 19px;
		font-weight: 600;
		letter-spacing: 0;
		line-height: 24px;
		text-align: center;
		cursor: pointer;
		user-select: none;
		-webkit-user-select: none;
		padding: 0;
	}

	.import-mode-tabs button.active {
		background: transparent;
		box-shadow: none;
		color: var(--bc-white);
		font-weight: 700;
	}

	.import-mode-tabs button.active::after {
		position: absolute;
		inset: auto 0 -1px;
		height: 2px;
		background: var(--bc-white);
		content: '';
	}

	.import-mode-tabs button:focus-visible {
		outline: 2px solid rgb(255 255 255 / 0.72);
		outline-offset: -3px;
	}

	.daynight-import-mobile :global(.mobile-service-entry__field .import-manual-entry) {
		display: grid;
		grid-template-columns: minmax(0, 1fr) 48px;
		width: 100%;
		height: auto;
		min-height: 56px;
		align-items: center;
		gap: 10px;
		border: 0;
		border-radius: var(--bc-radius-pill);
		background: var(--bc-surface-soft);
		color: var(--bc-ink);
		padding: 4px 4px 4px 17px;
		cursor: pointer;
		text-align: left;
	}

	.import-manual-entry__copy {
		display: grid;
		gap: 2px;
	}

	.import-manual-entry__copy strong {
		font-size: 15px;
		font-weight: 700;
		line-height: 1.2;
	}

	.import-manual-entry__copy small {
		color: var(--bc-muted);
		font-size: 12px;
		font-weight: 500;
		line-height: 1.2;
	}

	.import-manual-entry__go {
		display: grid;
		width: 48px;
		height: 48px;
		place-items: center;
		border-radius: 50%;
		background: var(--bc-accent);
		color: var(--bc-white);
	}

	:global(.daynight-import-wizard-drawer__backdrop) {
		position: fixed;
		inset: 0;
		z-index: 1200;
		border: 0;
		background: transparent;
		cursor: pointer;
		padding: 0;
	}

	:global(.daynight-import-wizard-drawer__backdrop span),
	:global(.daynight-import-wizard-drawer__title) {
		position: absolute;
		width: 1px;
		height: 1px;
		overflow: hidden;
		clip: rect(0 0 0 0);
		white-space: nowrap;
	}

	:global(.daynight-import-wizard-drawer__sheet) {
		position: fixed;
		inset: 0;
		z-index: 1201;
		display: block;
		height: 100dvh;
		overflow: hidden;
		overscroll-behavior: contain;
		border-radius: 0;
		background: var(--bc-bg-strong);
		outline: 0;
		padding: 0;
		scrollbar-width: none;
	}

	:global(.daynight-import-wizard-drawer__sheet::-webkit-scrollbar) {
		display: none;
	}

	:global(.daynight-import-wizard-drawer__sheet)::after {
		display: none;
	}

	:global(.daynight-import-wizard-drawer__handle) {
		display: none !important;
		position: relative;
		display: block;
		width: 56px;
		height: 22px;
		justify-self: center;
		border-radius: 0;
		background: transparent;
		opacity: 1;
	}

	:global(.daynight-import-wizard-drawer__handle)::after {
		position: absolute;
		top: 50%;
		left: 50%;
		width: 42px;
		height: 4px;
		transform: translate(-50%, -50%);
		border-radius: var(--bc-radius-pill);
		background: var(--bc-border);
		content: '';
	}
</style>
