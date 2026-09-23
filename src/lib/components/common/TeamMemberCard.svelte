<script lang="ts">
	import { assetHref } from '$lib/utils/assets';
	import type { AuxeroAgentCard } from '$lib/auxero/agents';
	import SocialLinks from './SocialLinks.svelte';
	let { person }: { person: Pick<AuxeroAgentCard, 'name' | 'title' | 'image' | 'socials'> } =
		$props();
	const socials = $derived(
		person.socials
			.filter((link) => link.icon === 'brands/instagram.svg')
			.map((link) => ({ platform: 'instagram' as const, label: link.label, href: link.href }))
	);
</script>

<article class="team-card">
	<img
		class="team-card__portrait"
		src={assetHref(person.image)}
		alt={person.name}
		width="600"
		height="700"
		loading="lazy"
	/>
	<div class="team-card__body">
		<h3>{person.name}</h3>
		<p>{person.title}</p>
		{#if socials.length}<div class="team-card__socials"><SocialLinks links={socials} /></div>{/if}
	</div>
</article>

<style>
	.team-card {
		min-width: 0;
		overflow: hidden;
		border-radius: var(--bc-radius-panel);
		background: var(--bc-surface);
	}
	.team-card__portrait {
		display: block;
		width: 100%;
		height: 360px;
		object-fit: cover;
		object-position: top;
	}
	.team-card__body {
		display: grid;
		justify-items: center;
		gap: var(--bc-space-3);
		padding: var(--bc-space-5);
		text-align: center;
	}
	h3 {
		margin: 0 0 var(--bc-space-2);
		font: var(--bc-weight-heading) var(--bc-text-h4)/1.3 var(--bc-font-heading);
	}
	p {
		margin: 0;
		color: var(--bc-copy);
		font-size: var(--bc-text-body);
	}
	.team-card__socials {
		display: contents;
	}
	@media (min-width: 768px) {
		.team-card {
			overflow: visible;
			border-radius: 0;
			background: transparent;
		}
		.team-card__portrait {
			height: auto;
			aspect-ratio: 4 / 3;
			border-radius: var(--bc-radius-card);
		}
		.team-card__body {
			grid-template-columns: minmax(0, 1fr) auto;
			justify-items: start;
			align-items: start;
			gap: var(--bc-space-2) var(--bc-space-4);
			padding: var(--bc-space-5) 0 0;
			text-align: left;
		}
		h3 {
			grid-column: 1 / -1;
			margin: 0;
		}
		p {
			grid-column: 1;
			grid-row: 2;
			font-size: var(--bc-text-body-lg);
			line-height: var(--bc-leading-body-lg);
		}
		.team-card__socials {
			display: block;
			grid-column: 2;
			grid-row: 2;
		}
	}
</style>
