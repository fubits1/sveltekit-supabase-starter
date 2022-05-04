<script>
	import data from '$lib/data/data.json';
</script>

{#each data as entry}
	<p>{entry.category} | {entry.value}</p>
{/each}
