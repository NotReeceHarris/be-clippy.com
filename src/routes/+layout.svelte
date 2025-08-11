<script lang="ts">
	import '../app.css';
	
	let { children } = $props();
</script>

<svelte:head>
	<title>Be Like Clippy</title>
	<meta name="description" content="Join the Be Like Clippy movement to make technology more user-friendly and transparent." />
	<meta property="og:title" content="Be Like Clippy" />
	<meta property="og:description" content="Join the Be Like Clippy movement to make technology more user-friendly and transparent." />
	<meta property="og:image" content="/clippys/clippy.png" />
	<meta property="og:url" content="https://be-clippy.com" />

	<link rel="icon" type="image/x-icon" href="/favicon.png" />
</svelte:head>

{@render children?.()}
