<script lang="ts">
	import '../app.css';
	import 'jazz-inspector-element';
	import { Head, Toaster } from '@fuxui/base';

	let { children } = $props();

	import { JazzProvider } from 'jazz-svelte';
	import { GithubCorner } from '@fuxui/social';
	import { Account } from 'jazz-tools';
	import { dev } from '$app/environment';
</script>

<JazzProvider
	sync={{
		peer: `wss://cloud.jazz.tools/?key=flo.bit.dev@gmail.com`
	}}
	AccountSchema={Account}
>
	<!-- {#if dev}
		<jazz-inspector></jazz-inspector>
	{/if} -->

	{@render children()}
</JazzProvider>

<Toaster />

<Head title="multiplayer drawing" emojiFavicon="🎨" />

<GithubCorner href="https://github.com/flo-bit/jazz-multiplayer-drawings" />
