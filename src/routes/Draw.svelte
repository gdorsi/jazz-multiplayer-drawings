<script lang="ts">
	import { page } from '$app/state';
	import { Counter } from '$lib/schema';
	import { CoState } from 'jazz-svelte';
	import Paper from './Paper.svelte';

	let { id } = $props();

	let painting = $derived(
		new CoState(Counter, id, {
			resolve: {
				painting: {
					$each: {
						segments: {
							$each: true
						}
					}
				}
			}
		})
	);
</script>

<Paper {painting} />
