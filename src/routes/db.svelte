<script context="module">
	import * as db from '$lib/db';

	export async function load() {
		const queryResult = await db.queryResult.all();

		return {
			props: { queryResult }
		};
	}
</script>

<script lang="ts">
	// import { user, auth } from '$lib/db';

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
		<blockquote>{continents}</blockquote>
		<form>
			<select bind:value={filterCont}>
				<option value="">Select a Continent</option>
				{#each continents as name, index}
					<option value={name}>{name || '- NA -'}</option>
				{/each}
			</select>
			{#if filterCont || filterCont === null}
				<select bind:value={filterState}>
					<option value="">Select a Country</option>
					{#each options as name}
						<option value={name.id}>{name.name}</option>
					{/each}
				</select>
			{/if}
		</form>

		<p>There are {options.length} entries for {filterCont}.</p>
		{#if filterState}
			<p>filter - row.id: {filterState}</p>
			<hr />
			<table>
				<thead>
					<tr>
						{#each keys as label}
							<th>{label}</th>
						{/each}
					</tr>
				</thead>
				<tbody>
					{#each selection as row}
						<tr>
							{#each keys as label}
								<td>{row[label]}</td>
							{/each}
						</tr>
					{/each}
				</tbody>
			</table>
			<p>Keys: {Object.keys(queryResult[0])}</p>
		{/if}
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
</style>
