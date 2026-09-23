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
			display: flex;
			flex-direction: column;
			align-items: center;
			padding: var(--bc-space-8) var(--bc-space-5);
			border: 1px solid var(--bc-border);
			background: var(--bc-surface-raised);
			box-shadow: var(--bc-shadow-subtle);
		}
		.team-card__portrait {
			width: 144px;
			height: 144px;
			border-radius: 50%;
		}
		.team-card__body {
			flex: 1;
			width: 100%;
			grid-template-rows: auto 1fr auto;
			gap: var(--bc-space-2);
			padding: var(--bc-space-6) 0 0;
		}
		h3 {
			margin: 0;
		}
		p {
			line-height: var(--bc-leading-body-lg);
			max-width: 26ch;
			text-wrap: balance;
		}
		.team-card__socials {
			display: block;
			margin-top: var(--bc-space-4);
		}
	}
</style>
