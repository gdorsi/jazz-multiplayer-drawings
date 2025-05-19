<script>
	import Draw from './Draw.svelte';
	import Grid from './Grid.svelte';
	import { painting } from './store.svelte';
	import { page } from '$app/state';
	import { afterNavigate } from '$app/navigation';

	afterNavigate(() => {
		const id = page.url.searchParams.get('id');
		if (id) {
			painting.id = id;
		} else {
			painting.id = null;
		}
	});
</script>

{#if painting.id}
	<Draw id={painting.id} />
{:else}
	<Grid />
{/if}
