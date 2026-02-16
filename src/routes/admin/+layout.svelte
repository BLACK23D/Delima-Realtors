<script>
	import { onMount } from 'svelte';
	import { adminAuth } from '$lib/stores/adminAuth';

	let isLoggedIn = false;

	onMount(() => {
		adminAuth.subscribe((auth) => {
			isLoggedIn = auth.isLoggedIn;
		});
	});
</script>

{#if isLoggedIn}
	<slot />
{/if}
