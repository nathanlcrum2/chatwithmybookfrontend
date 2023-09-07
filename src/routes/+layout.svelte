<script lang="ts">
	import Footer from '$lib/Footer.svelte';
	import '../app.postcss';
	import { AppShell } from '@skeletonlabs/skeleton';
</script>

<!-- App Shell -->
<AppShell>
	<!-- Page Route Content -->
	<slot />
	<Footer/>
</AppShell>
