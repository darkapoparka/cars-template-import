<script lang="ts">
	import { resolve } from '$app/paths';
	import { ArrowRight, MapPin, Navigation, PhoneCall, ScanLine, X } from '@lucide/svelte';
	import type {
		AuxeroSellCarFormData,
		AuxeroSellCarMobileCopy,
		AuxeroSellCarMobileStep
	} from '$lib/auxero/sell-your-car';
	import { daynightContact } from '$lib/data/daynight';
	import { Drawer } from 'vaul-svelte';
	import MobileAppbar from '$lib/components/layout/MobileAppbar.svelte';
	import MobileServiceEntry from '$lib/components/services/MobileServiceEntry.svelte';
	import SellCarWizard from './SellCarWizard.svelte';

	let {
		copy,
		form,
		steps
	}: {
		copy: AuxeroSellCarMobileCopy;
		form: AuxeroSellCarFormData;
		steps: AuxeroSellCarMobileStep[];
	} = $props();

	let wizardOpen = $state(false);
	let manualEntry = $state(false);
	let wizardSession = $state(0);
	const vinField = $derived(form.fields.find((field) => field.name === 'vin')!);
	/* Editable copies let the hero VIN carry into the detailed valuation. */
	// svelte-ignore state_referenced_locally
	let fieldValues = $state(
		Object.fromEntries(form.fields.map((field) => [field.name, field.value ?? ''])) as Record<
			string,
			string
		>
	);

	const mapHref = `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(
		daynightContact.addressLabel
	)}`;
	const hrefAttributes = (href: string) => ({
		href: href.startsWith('/') ? resolve(href as '/') : href
	});

	const openWizard = (manual: boolean) => {
		manualEntry = manual;
		wizardSession += 1;
		wizardOpen = true;
	};
	const handleVinSubmit = (event: SubmitEvent) => {
		event.preventDefault();
		openWizard(false);
	};
	const setLocationSheetOpen = (open: boolean) => {
		const toggle = document.getElementById(
			'sell-mobile-location-toggle'
		) as HTMLInputElement | null;
		if (toggle && toggle.checked !== open) toggle.click();
	};
</script>

<div class="daynight-sell-mobile">
	<input
		id="sell-mobile-location-toggle"
		class="daynight-sell-mobile__sheet-toggle"
		type="checkbox"
		autocomplete="off"
		tabindex="-1"
		aria-hidden="true"
	/>

	<MobileAppbar
		actionsLabel={copy.contactLabel}
		logoAlt={copy.logoAlt}
		surface="dark"
		onMap={() => setLocationSheetOpen(true)}
	/>

	<MobileServiceEntry
		title={copy.title}
		intro="Въведи VIN, добави основните данни и ще се свържем с оценка до 24 ч."
		meta="5 стъпки · Снимки по желание"
		response="Демонстрационна оценка"
		stepsTitle={copy.stepsTitle}
		{steps}
	>
		{#snippet entry()}
			<form onsubmit={handleVinSubmit}>
				<label for="sell-mobile-vin">VIN номер</label>
				<div class="service-input">
					<ScanLine size={21} strokeWidth={2.15} aria-hidden="true" />
					<input
						id="sell-mobile-vin"
						name={vinField.name}
						type={vinField.type}
						placeholder="Въведи VIN номер"
						required={vinField.required}
						autocomplete={vinField.autocomplete}
						bind:value={fieldValues.vin}
					/>
					<button type="submit" aria-label="Продължи с VIN">
						<ArrowRight size={21} strokeWidth={2.35} aria-hidden="true" />
					</button>
				</div>
			</form>
		{/snippet}
		{#snippet alternative()}
			<button class="sell-entry-banner" type="button" onclick={() => openWizard(true)}>
				<img
					src={resolve('/assets/daynight/sell/sell-mobile-banner-v1.webp')}
					alt=""
					aria-hidden="true"
				/>
				<span class="sell-entry-banner__copy">
					<strong>Нямам VIN</strong>
					<small>Избери автомобила ръчно</small>
				</span>
				<span class="sell-entry-banner__go" aria-hidden="true">
					<ArrowRight size={19} strokeWidth={2.35} />
				</span>
			</button>
		{/snippet}
	</MobileServiceEntry>

	<div
		id="sell-mobile-location-sheet"
		class="daynight-sell-mobile-sheet daynight-sell-mobile-sheet--location"
		role="dialog"
		aria-modal="true"
		aria-labelledby="sell-mobile-location-title"
	>
		<button
			type="button"
			class="daynight-sell-mobile-sheet__backdrop"
			aria-label="Затвори локацията"
			onclick={() => setLocationSheetOpen(false)}
		></button>
		<div class="daynight-sell-mobile-sheet__panel">
			<span class="daynight-sell-mobile-sheet__handle" aria-hidden="true"></span>
			<header class="daynight-sell-mobile-sheet__header">
				<div>
					<p>Day Night Auto шоурум</p>
					<h2 id="sell-mobile-location-title">{daynightContact.addressLabel}</h2>
				</div>
				<button type="button" aria-label="Затвори" onclick={() => setLocationSheetOpen(false)}>
					<X size={20} strokeWidth={2.3} aria-hidden="true" />
				</button>
			</header>
			<div class="daynight-sell-mobile__map-preview" aria-hidden="true">
				<span class="road road-a"></span>
				<span class="road road-b"></span>
				<span class="road road-c"></span>
				<span class="pin"><MapPin size={24} strokeWidth={2.4} /></span>
				<span class="badge">Day Night Auto</span>
			</div>
			<div class="daynight-sell-mobile__location-copy">
				<span>{daynightContact.appointmentNote}</span>
				<strong>{daynightContact.addressLabel}</strong>
				<p>Огледите са с уговорка. Обади се преди посещение, за да подготвим човека и времето.</p>
			</div>
			<div class="daynight-sell-mobile-sheet__actions">
				<a {...hrefAttributes(mapHref)} target="_blank" rel="noreferrer">
					<Navigation size={18} strokeWidth={2.25} aria-hidden="true" />
					Отвори карта
				</a>
				<a {...hrefAttributes(daynightContact.primaryPhoneHref)}>
					<PhoneCall size={18} strokeWidth={2.25} aria-hidden="true" />
					Обади се
				</a>
			</div>
		</div>
	</div>

	<Drawer.Root bind:open={wizardOpen} direction="bottom" fixed={true}>
		<Drawer.Overlay class="daynight-sell-wizard-drawer__backdrop">
			<span>Затвори</span>
		</Drawer.Overlay>
		<Drawer.Content class="daynight-sell-wizard-drawer__sheet">
			<Drawer.Title class="daynight-sell-wizard-drawer__title">Оценка на автомобила</Drawer.Title>
			{#key wizardSession}
				<SellCarWizard
					initial={{
						make: '',
						model: '',
						year: '',
						mileage: fieldValues.mileage,
						phone: fieldValues.phone,
						price: fieldValues.price,
						vin: manualEntry ? '' : fieldValues.vin
					}}
					{manualEntry}
					onclose={() => (wizardOpen = false)}
				/>
			{/key}
		</Drawer.Content>
	</Drawer.Root>
</div>

<style>
	.daynight-sell-mobile {
		position: relative;
		min-height: calc(100dvh - var(--bc-mobile-nav-height) - env(safe-area-inset-bottom));
		background: var(--bc-bg-strong);
		color: var(--bc-ink);
	}

	.daynight-sell-mobile :global(.mobile-service-entry) {
		min-height: 0;
	}

	.daynight-sell-mobile :global(a) {
		text-decoration: none !important;
	}

	.daynight-sell-mobile :global(a) {
		text-decoration: none !important;
	}

	.daynight-sell-mobile :global(.mobile-service-entry__browse) {
		position: relative;
		z-index: 2;
		margin-top: -10px;
		border-radius: 24px 24px 0 0;
		background: #eef1f4;
		padding: 26px var(--bc-mobile-gutter) var(--bc-space-5);
		box-shadow: 0 -1px 0 rgba(255, 255, 255, 0.14);
	}
	.daynight-sell-mobile :global(.mobile-service-entry__browse)::before {
		position: absolute;
		top: 9px;
		left: 50%;
		width: 38px;
		height: 4px;
		border-radius: 999px;
		background: #c3cad2;
		content: '';
		transform: translateX(-50%);
	}
	.daynight-sell-mobile :global(.mobile-service-entry__alternative) {
		margin: 0;
	}
	.daynight-sell-mobile :global(.mobile-service-entry__alternative .sell-entry-banner) {
		position: relative;
		display: block;
		min-height: 154px;
		overflow: hidden;
		border: 0;
		border-radius: 16px;
		background: #0b0d10;
		padding: 0;
		box-shadow: none;
	}
	.daynight-sell-mobile :global(.sell-entry-banner > img) {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		object-fit: cover;
		object-position: 62% center;
		opacity: 0.92;
	}
	.daynight-sell-mobile :global(.sell-entry-banner)::after {
		position: absolute;
		inset: 0;
		background: linear-gradient(
			90deg,
			rgba(5, 7, 10, 0.94) 0%,
			rgba(5, 7, 10, 0.64) 45%,
			rgba(5, 7, 10, 0.08) 78%
		);
		content: '';
	}
	.daynight-sell-mobile :global(.sell-entry-banner__copy) {
		position: absolute;
		z-index: 2;
		top: 50%;
		left: 18px;
		display: grid;
		max-width: 48%;
		gap: 5px;
		transform: translateY(-50%);
		color: #fff;
	}
	.daynight-sell-mobile :global(.sell-entry-banner__copy strong) {
		color: #fff;
		font-size: 21px;
		font-weight: 750;
		line-height: 1.1;
	}
	.daynight-sell-mobile :global(.sell-entry-banner__copy small) {
		color: rgba(255, 255, 255, 0.78);
		font-size: 13px;
		font-weight: 500;
		line-height: 1.3;
	}
	.daynight-sell-mobile :global(.sell-entry-banner__go) {
		position: absolute;
		z-index: 3;
		right: 14px;
		bottom: 14px;
		display: grid;
		width: 40px;
		height: 40px;
		place-items: center;
		border-radius: 50%;
		background: #fff;
		color: var(--bc-ink);
	}

	.daynight-sell-mobile__sheet-toggle {
		position: fixed;
		width: 1px;
		height: 1px;
		overflow: hidden;
		opacity: 0;
		pointer-events: none;
	}

	.daynight-sell-mobile-sheet__header p {
		margin: 0;
		color: var(--bc-accent);
		font-size: var(--bc-mobile-meta);
		font-weight: 600;
		letter-spacing: 0.04em;
		line-height: var(--bc-mobile-meta-leading);
		text-transform: uppercase;
	}

	:global(.daynight-sell-wizard-drawer__backdrop) {
		position: fixed;
		inset: 0;
		z-index: 1200;
		border: 0;
		background: transparent;
		cursor: pointer;
		padding: 0;
	}

	:global(.daynight-sell-wizard-drawer__backdrop span),
	:global(.daynight-sell-wizard-drawer__title) {
		position: absolute;
		width: 1px;
		height: 1px;
		overflow: hidden;
		clip: rect(0 0 0 0);
		white-space: nowrap;
	}

	:global(.daynight-sell-wizard-drawer__sheet) {
		position: fixed;
		inset: 0;
		z-index: 1201;
		display: block;
		width: 100%;
		height: 100dvh;
		max-height: none;
		overflow: hidden;
		overscroll-behavior: contain;
		border-radius: 0;
		background: var(--bc-bg-strong);
		box-shadow: none;
		outline: 0;
		padding: 0;
		scrollbar-width: none;
	}

	:global(.daynight-sell-wizard-drawer__sheet .sell-flow) {
		height: 100%;
		min-height: 0;
	}

	:global(.daynight-sell-wizard-drawer__sheet::-webkit-scrollbar) {
		display: none;
	}

	/* Vaul extends bottom sheets with a 200% pseudo-element for drag bounce.
	   A full-screen document sheet must not expose that empty scroll range. */
	:global(.daynight-sell-wizard-drawer__sheet)::after {
		display: none;
	}

	:global(.daynight-sell-wizard-drawer__handle) {
		position: absolute;
		top: 7px;
		left: 50%;
		z-index: 5;
		display: block !important;
		width: 40px;
		height: 4px;
		border-radius: 999px;
		background: rgba(255, 255, 255, 0.34);
		transform: translateX(-50%);
		opacity: 1;
	}

	.daynight-sell-mobile-sheet {
		position: fixed;
		inset: 0;
		z-index: 1000;
		visibility: hidden;
		pointer-events: none;
	}

	#sell-mobile-location-toggle:checked ~ #sell-mobile-location-sheet {
		visibility: visible;
		pointer-events: auto;
	}

	.daynight-sell-mobile-sheet__backdrop {
		position: absolute;
		inset: 0;
		border: 0;
		background: rgba(28, 28, 28, 0.36);
		cursor: pointer;
		opacity: 0;
		padding: 0;
		transition: opacity 180ms ease;
	}

	#sell-mobile-location-toggle:checked
		~ #sell-mobile-location-sheet
		.daynight-sell-mobile-sheet__backdrop {
		opacity: 1;
	}

	.daynight-sell-mobile-sheet__panel {
		position: absolute;
		right: 0;
		/* Lifted above the on-screen keyboard on iOS; --bc-kb-inset stays 0 elsewhere. */
		bottom: var(--bc-kb-inset, 0px);
		left: 0;
		display: grid;
		gap: var(--bc-space-3);
		max-height: min(calc(88dvh - var(--bc-kb-inset, 0px)), 720px);
		overflow-y: auto;
		border: 0;
		border-top: 1px solid var(--bc-border);
		border-radius: var(--bc-radius-panel) var(--bc-radius-panel) 0 0;
		background: var(--bc-bg-strong);
		box-shadow: var(--bc-shadow-panel);
		padding: var(--bc-space-2) var(--bc-mobile-gutter)
			calc(var(--bc-space-4) + env(safe-area-inset-bottom));
		transform: translateY(100%);
		transition: transform 240ms cubic-bezier(0.22, 1, 0.36, 1);
	}

	#sell-mobile-location-toggle:checked
		~ #sell-mobile-location-sheet
		.daynight-sell-mobile-sheet__panel {
		transform: translateY(0);
	}

	.daynight-sell-mobile-sheet__handle {
		display: block;
		width: 42px;
		height: 5px;
		justify-self: center;
		border-radius: var(--bc-radius-pill);
		background: var(--bc-border);
	}

	.daynight-sell-mobile-sheet__header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		gap: var(--bc-space-3);
	}

	.daynight-sell-mobile-sheet__header div {
		display: grid;
		gap: 2px;
		min-width: 0;
	}

	.daynight-sell-mobile-sheet__header p {
		color: var(--bc-muted);
	}

	.daynight-sell-mobile-sheet__header h2 {
		margin: 0;
		color: var(--bc-ink);
		font-size: var(--bc-mobile-section-title);
		font-weight: 700;
		letter-spacing: 0;
		line-height: var(--bc-mobile-section-title-leading);
	}

	.daynight-sell-mobile-sheet__header button {
		display: flex;
		width: var(--bc-control-height-standard);
		height: var(--bc-control-height-standard);
		flex: 0 0 var(--bc-control-height-standard);
		align-items: center;
		justify-content: center;
		border: 0;
		border-radius: var(--bc-radius-pill);
		background: var(--bc-surface-hover);
		color: var(--bc-ink);
		cursor: pointer;
		padding: 0;
	}

	.daynight-sell-mobile__map-preview {
		position: relative;
		height: 132px;
		overflow: hidden;
		border-radius: var(--bc-radius-card);
		background:
			linear-gradient(135deg, rgba(216, 221, 227, 0.72), rgba(248, 250, 252, 0.92)),
			var(--bc-bg-strong);
	}

	.daynight-sell-mobile__map-preview::before,
	.daynight-sell-mobile__map-preview::after {
		position: absolute;
		inset: 18px;
		border: 1px solid rgba(28, 28, 28, 0.08);
		border-radius: 18px;
		content: '';
	}

	.daynight-sell-mobile__map-preview::after {
		inset: 42px -22px auto 44px;
		height: 48px;
		border-right: 0;
		border-left: 0;
		transform: rotate(-7deg);
	}

	.daynight-sell-mobile__map-preview .road {
		position: absolute;
		border-radius: var(--bc-radius-pill);
		background: rgba(28, 28, 28, 0.1);
	}

	.daynight-sell-mobile__map-preview .road-a {
		top: 29px;
		left: -20px;
		width: 72%;
		height: 8px;
		transform: rotate(12deg);
	}

	.daynight-sell-mobile__map-preview .road-b {
		right: 18px;
		bottom: 31px;
		width: 58%;
		height: 8px;
		transform: rotate(-19deg);
	}

	.daynight-sell-mobile__map-preview .road-c {
		top: 15px;
		left: 47%;
		width: 8px;
		height: 116px;
		transform: rotate(21deg);
	}

	.daynight-sell-mobile__map-preview .pin {
		position: absolute;
		top: 45px;
		left: 50%;
		display: flex;
		width: 48px;
		height: 48px;
		align-items: center;
		justify-content: center;
		border: 4px solid var(--bc-white);
		border-radius: var(--bc-radius-pill);
		background: var(--bc-accent);
		color: var(--bc-white);
		color: var(--bc-ink);
		transform: translateX(-50%);
	}

	.daynight-sell-mobile__map-preview .badge {
		position: absolute;
		right: 15px;
		bottom: 14px;
		border-radius: var(--bc-radius-pill);
		background: var(--bc-ink);
		color: var(--bc-white);
		font-size: var(--bc-mobile-meta);
		font-weight: 700;
		line-height: var(--bc-mobile-meta-leading);
		padding: 7px 10px;
	}

	.daynight-sell-mobile__location-copy {
		display: grid;
		gap: 4px;
		border-radius: var(--bc-radius-card);
		background: var(--bc-white);
		padding: var(--bc-space-3) var(--bc-mobile-gutter);
	}

	.daynight-sell-mobile__location-copy span,
	.daynight-sell-mobile__location-copy strong,
	.daynight-sell-mobile__location-copy p {
		margin: 0;
		letter-spacing: 0;
	}

	.daynight-sell-mobile__location-copy span {
		color: var(--bc-muted);
		font-size: var(--bc-mobile-meta);
		font-weight: 700;
		line-height: var(--bc-mobile-meta-leading);
		text-transform: uppercase;
	}

	.daynight-sell-mobile__location-copy strong {
		color: var(--bc-ink);
		font-size: var(--bc-mobile-card-title);
		font-weight: 700;
		line-height: var(--bc-mobile-card-title-leading);
	}

	.daynight-sell-mobile__location-copy p {
		color: var(--bc-copy);
		font-size: var(--bc-mobile-body);
		font-weight: 500;
		line-height: var(--bc-mobile-label-leading);
	}

	.daynight-sell-mobile-sheet__actions {
		display: grid;
		grid-template-columns: repeat(2, minmax(0, 1fr));
		gap: var(--bc-space-3);
	}

	.daynight-sell-mobile-sheet__actions a {
		display: flex;
		min-height: var(--bc-control-height-primary);
		align-items: center;
		justify-content: center;
		gap: var(--bc-space-2);
		border-radius: var(--bc-radius-control);
		background: var(--bc-white);
		color: var(--bc-ink);
		font-size: var(--bc-mobile-body);
		font-weight: 700;
		line-height: var(--bc-mobile-label-leading);
	}

	.daynight-sell-mobile-sheet__actions a:first-child {
		background: var(--bc-accent);
		color: var(--bc-white);
	}
</style>
