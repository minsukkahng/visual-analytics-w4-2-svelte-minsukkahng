<script>
	import { onMount } from "svelte";

	export const myName = "REPLACE_WITH_YOUR_NAME";

	let data;
	onMount(async () => {
		const fetched = await fetch("/static/census2000.json");
		let rawData = (await fetched.json()).demographic_groups;
		console.log(rawData);

		data = rawData;

	});
</script>

<main>
	<h1>In-Class Acitivity by {myName}!</h1>

	<svg id="visualization">
		{#if data !== undefined}
			{#each data as record, index}
				<g transform="translate(0, {index * 10})">
					<text>{record.people}</text>
				</g>
			{/each}
		{/if}
	</svg>

</main>

<style>
	h1 {
		font-size: 1.5rem;
	}
	svg#visualization {
		width: 600px;
		height: 500px;
	}
	text {
		font-size: 0.6rem;
	}

</style>
