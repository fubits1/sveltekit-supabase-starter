<script context="module">
	export const prerender = false;
	import * as db from '$lib/db';

	export async function load() {
		const queryResult = await db.queryResult.all();

		return {
			props: { queryResult }
		};
	}
</script>

<script>
	// import { user, auth } from '$lib/db';

	import { fade } from 'svelte/transition';

	export let queryResult;

	let filterCont;
	let filterState; // TODO: move filter logic into query logic

	$: options = queryResult
		/** redudant */
		// ?.map((entry) => {
		// 	return { id: entry.id, name: entry.name };
		// })
		.filter((entry) => {
			return entry.continent === filterCont;
		})
		.sort((a, b) => a.name.localeCompare(b.name));

	$: selection = filterState
		? options.filter((row) => {
				return row.id == filterState;
		  })
		: ' - ';

	$: keys = Object.keys(queryResult[0]);

	$: continents = [...new Set(queryResult.map((row) => row.continent))]
		// .filter((continent) => continent != null) // null implies NA
		.sort();
	// .map((val) => (val === null ? '-- NA --' : val));

	// LayerCake example
	import { LayerCake, Svg, Html } from 'layercake';
	import { scaleOrdinal, scaleBand } from 'd3-scale';

	import ForceLayout from '$lib/components/CirclePackForce.svelte';

	// let data = queryResult;
	import data from '$lib/data/data.json';
	// import { fade } from 'svelte/transition';

	console.log(data);
	const xKey = 'category';
	const rKey = 'value';
	const zKey = 'category';

	let groupBy = 'true';

	const seriesNameSet = new Set();
	const seriesColors = ['#f0c', '#0cf', '#fc0', '#f00', '#0f0', '#00f', '#f0f', '#000', '#fff'];

	data.forEach((d) => {
		seriesNameSet.add(d[zKey]);
	});

	// /* --------------------------------------------
	//  * Convert this to an array so we can use it in our scales
	//  */
	const seriesNames = [...seriesNameSet].sort().filter((x) => x != null);
	console.dir(seriesNames);

	let manyBodyStrength = 3;
	let xStrength = 0.1;
</script>

<section>
	<!-- {#if $user}
		<p>You are signed in as {$user.email}</p>
		<button on:click={() => auth.signOut()}>Sign out</button>
	{:else}
		<nav>
			<a href="/sign-in">Sign in</a>
		</nav>
	{/if} -->
	{#if queryResult}
		<blockquote transition:fade>{continents}</blockquote>
		<form transition:fade>
			<select bind:value={filterCont} transition:fade>
				<option value="">Select a Continent</option>
				{#each continents as name, index}
					<option value={name}>{name || '- NA -'}</option>
				{/each}
			</select>
			{#if filterCont || filterCont === null}
				<select bind:value={filterState} transition:fade>
					<option value="">Select a Country</option>
					{#each options as name}
						<option value={name.id}>{name.name}</option>
					{/each}
				</select>
			{/if}
		</form>

		<p>There are {options.length} entries for {filterCont}.</p>
		{#if filterState}
			<p transition:fade>filter - row.id: {filterState}</p>
			<hr />
			<table>
				<thead>
					<tr>
						{#each keys as label}
							<th transition:fade>{label}</th>
						{/each}
					</tr>
				</thead>
				<tbody>
					{#each selection as row}
						<tr>
							{#each keys as label}
								<td transition:fade>{row[label]}</td>
							{/each}
						</tr>
					{/each}
				</tbody>
			</table>
			<p>Keys: {Object.keys(queryResult[0])}</p>
		{/if}
	{/if}

	<hr />

	{#if data}
		{data.length}
		<div class="input-container">
			<label><input type="radio" bind:group={groupBy} value="true" />Group by category</label>
			<label><input type="radio" bind:group={groupBy} value="false" />Clump together</label>
		</div>

		<div class="chart-container" transition:fade>
			<LayerCake
				{data}
				x={xKey}
				r={rKey}
				z={zKey}
				xScale={scaleBand()}
				xDomain={seriesNames}
				rRange={[3, 12]}
				zScale={scaleOrdinal()}
				zDomain={seriesNames}
				zRange={seriesColors}
			>
				<Svg>
					<ForceLayout
						{manyBodyStrength}
						{xStrength}
						groupBy={JSON.parse(groupBy)}
						nodeStroke="#000"
					/>
				</Svg>
			</LayerCake>
		</div>
	{/if}
</section>

<style>
	main {
		max-width: 720px;
		margin: 0 auto;
	}

	table {
		min-width: 100%;
	}

	/*
    The wrapper div needs to have an explicit width and height in CSS.
    It can also be a flexbox child or CSS grid element.
    The point being it needs dimensions since the <LayerCake> element will
    expand to fill it.
  */
	.chart-container {
		width: 90%;
		height: auto;
		aspect-ratio: 1;
		max-height: 80vh;
	}
</style>
