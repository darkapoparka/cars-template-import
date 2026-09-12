<script lang="ts">
	import { resolve } from '$app/paths';
	import { Calendar, Cog, Fuel, Gauge } from '@lucide/svelte';

	type MobileVehicleCardData = {
		slug: string;
		image: string;
		title: string;
		brand: string;
		priceLabel: string;
		monthlyLabel: string;
		mileageLabel: string;
		year: string | number;
		fuel: string;
		transmission: string;
		tag?: string;
	};

	let {
		card,
		image = card.image,
		fallbackImage = '/assets/images/card/card-48.jpg'
	}: { card: MobileVehicleCardData; image?: string; fallbackImage?: string } = $props();

	const useFallbackImage = (event: Event) => {
		const element = event.currentTarget as HTMLImageElement;
		if (!element.src.endsWith(fallbackImage)) element.src = fallbackImage;
	};
</script>

<article class="mobile-vehicle-card">
	<a class="mobile-vehicle-card__image" href={resolve('/inventory/[slug]', { slug: card.slug })}>
		<img
			src={image}
			alt={card.title}
			width="660"
			height="440"
			loading="lazy"
			decoding="async"
			onerror={useFallbackImage}
		/>
		{#if card.tag}<span>{card.tag}</span>{/if}
	</a>
	<div class="mobile-vehicle-card__body">
		<p>{card.brand}</p>
		<h2><a href={resolve('/inventory/[slug]', { slug: card.slug })}>{card.title}</a></h2>
		<strong>{card.priceLabel}</strong>
		<small>{card.monthlyLabel}</small>
		<ul>
			<li><Gauge size={14} strokeWidth={2} aria-hidden="true" />{card.mileageLabel}</li>
			<li><Calendar size={14} strokeWidth={2} aria-hidden="true" />{card.year}</li>
			<li><Fuel size={14} strokeWidth={2} aria-hidden="true" />{card.fuel}</li>
			<li><Cog size={14} strokeWidth={2} aria-hidden="true" />{card.transmission}</li>
		</ul>
	</div>
</article>

<style>
	.mobile-vehicle-card {
		display: grid;
		width: 100%;
		min-width: 0;
		grid-template-columns: clamp(96px, 32vw, 132px) minmax(0, 1fr);
		min-height: 154px;
		overflow: hidden;
		border-radius: var(--bc-radius-lg);
		background: var(--bc-white);
	}

	.mobile-vehicle-card__image {
		position: relative;
		display: block;
		min-height: 154px;
		overflow: hidden;
		text-decoration: none !important;
	}

	.mobile-vehicle-card__image img {
		display: block;
		width: 100%;
		height: 100%;
		min-height: 154px;
		object-fit: cover;
	}

	.mobile-vehicle-card__image span {
		position: absolute;
		top: var(--bc-space-2);
		left: var(--bc-space-2);
		min-height: 24px;
		border-radius: var(--bc-radius-pill);
		background: var(--bc-accent);
		padding: 0 var(--bc-space-2);
		color: var(--bc-white);
		font-size: 10px;
		font-weight: 700;
		line-height: 24px;
		text-transform: uppercase;
	}

	.mobile-vehicle-card__body {
		display: grid;
		min-width: 0;
		align-content: start;
		padding: var(--bc-space-2) var(--bc-space-2) var(--bc-space-2) var(--bc-space-3);
	}

	.mobile-vehicle-card__body p {
		margin: 0 0 2px;
		color: var(--bc-muted);
		font-size: var(--bc-mobile-stat);
		font-weight: 600;
		line-height: var(--bc-mobile-stat-leading);
		text-transform: uppercase;
	}

	.mobile-vehicle-card__body h2 {
		min-width: 0;
		margin: 0 0 var(--bc-space-1);
		color: var(--bc-ink);
		font-size: var(--bc-mobile-card-title);
		font-weight: 650;
		line-height: var(--bc-mobile-card-title-leading);
		overflow-wrap: anywhere;
	}

	.mobile-vehicle-card__body h2 a {
		display: flex;
		min-width: 0;
		min-height: var(--bc-control-height-standard);
		align-items: center;
		margin-block: -11px;
		padding-block: 11px;
		color: inherit;
		text-decoration: none !important;
	}

	.mobile-vehicle-card__body strong {
		color: var(--bc-accent);
		font-size: 20px;
		font-weight: 700;
		line-height: var(--bc-space-6);
	}

	.mobile-vehicle-card__body small {
		margin-bottom: var(--bc-space-2);
		color: var(--bc-muted);
		font-size: var(--bc-mobile-stat);
		font-weight: 600;
		line-height: var(--bc-mobile-stat-leading);
	}

	.mobile-vehicle-card__body ul {
		display: grid;
		grid-template-columns: minmax(max-content, 1fr) minmax(0, 1fr);
		gap: var(--bc-space-2);
		margin: 0;
		padding: 0;
		list-style: none;
	}

	.mobile-vehicle-card__body li {
		display: flex;
		min-width: 0;
		min-height: var(--bc-mobile-stat-height);
		align-items: center;
		gap: var(--bc-space-1);
		border-radius: var(--bc-radius-md);
		background: var(--bc-surface-soft);
		padding: var(--bc-space-1) var(--bc-space-2);
		color: var(--bc-copy);
		font-size: var(--bc-mobile-stat);
		font-weight: 600;
		line-height: var(--bc-mobile-stat-leading);
		white-space: nowrap;
		font-variant-numeric: tabular-nums;
	}

	.mobile-vehicle-card__body li :global(svg) {
		flex: 0 0 auto;
	}

	@media (max-width: 399px) {
		.mobile-vehicle-card {
			grid-template-columns: clamp(96px, calc(100vw - 251px), 124px) minmax(0, 1fr);
		}
	}

	@media (max-width: 359px) {
		.mobile-vehicle-card__body li :global(svg) {
			display: none;
		}
	}
</style>
