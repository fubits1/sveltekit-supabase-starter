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

	let filter; // TODO: move filter logic into query logic

	$: options = queryResult
		/** redudant */
		// ?.map((entry) => {
		// 	return { id: entry.id, name: entry.name };
		// })
		.sort((a, b) => a.name.localeCompare(b.name));

	$: selection = filter
		? queryResult.filter((row) => {
				return row.id == filter;
		  })
		: ' - ';

	$: keys = Object.keys(queryResult[0]);
</script>

<main>
	<!-- {#if $user}
		<p>You are signed in as {$user.email}</p>
		<button on:click={() => auth.signOut()}>Sign out</button>
	{:else}
		<nav>
			<a href="/sign-in">Sign in</a>
		</nav>
	{/if} -->
	{#if queryResult}
		<form>
			<select bind:value={filter}>
				<option value="">Select a selection</option>
				{#each options as name}
					<option value={name.id}>{name.name}</option>
				{/each}
			</select>
		</form>

		<p>There are {queryResult.length} entries.</p>
		{#if filter}
			<p>filter - row.id: {filter}</p>
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
</main>

<style>
	main {
		max-width: 720px;
		margin: 0 auto;
	}

	table {
		min-width: 100%;
	}
</style>
