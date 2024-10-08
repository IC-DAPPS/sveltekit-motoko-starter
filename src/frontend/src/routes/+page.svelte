<script lang="ts">
	import AuthButton from '$lib/components/AuthButton/AuthButton.svelte';
	import { authStore } from '$lib/stores/auth.store';
	import { AuthClient } from '@dfinity/auth-client';
	import { createActor } from '../../../declarations/backend';
	import { onMount } from 'svelte';

	let input = '';
	let disabled = false;
	let greeting = '';

	const handleOnSubmit = async () => {
		disabled = true;

		try {
			// Canister IDs are automatically expanded to .env config - see vite.config.ts
			const canisterId = import.meta.env.VITE_BACKEND_CANISTER_ID;

			// We pass the host instead of using a proxy to support NodeJS >= v17 (ViteJS issue: https://github.com/vitejs/vite/issues/4794)
			const host = import.meta.env.VITE_HOST;

			// Create an actor to interact with the IC for a particular canister ID
			const actor = createActor(canisterId, { agentOptions: { host } });

			// Call the IC
			greeting = await actor.greet(input);
		} catch (err: unknown) {
			console.error(err);
		}

		disabled = false;
	};

	// Safari double clicks issues with the AuthButton component can be fixed temporarily with below hack
	// onMount(async () => {
	// 	try {
	// 		if (/apple/i.test(navigator?.vendor)) {
	// 			await authStore.signIn({ domain: 'internetcomputer.org' });
	// 		}
	// 	} catch (error) {
	// 		console.error('Sign in error:', error);
	// 	}
	// });
</script>

<main>
	<img src="logo2.svg" alt="DFINITY logo" />
	<br />
	<br />
	<AuthButton />
	{#if $authStore.identity}
		<h3>Hello, "{$authStore.identity.getPrincipal().toText()}"</h3>
	{/if}

	<form on:submit|preventDefault={handleOnSubmit}>
		<label for="name">Enter your name: &nbsp;</label>
		<input id="name" alt="Name" type="text" bind:value={input} {disabled} />
		<button type="submit">Click Me!</button>
	</form>

	<section id="greeting">
		{greeting}
	</section>
</main>

<style lang="scss">
	img {
		max-width: 50vw;
		max-height: 25vw;
		display: block;
		margin: auto;
	}

	form {
		display: flex;
		justify-content: center;
		gap: 0.5em;
		flex-flow: row wrap;
		max-width: 40vw;
		margin: auto;
		align-items: baseline;
		font-family: sans-serif;
		font-size: 1.5rem;
	}

	button[type='submit'] {
		padding: 5px 20px;
		margin: 10px auto;
		float: right;
	}

	#greeting {
		margin: 10px auto;
		padding: 10px 60px;
		border: 1px solid #222;
	}

	#greeting:empty {
		display: none;
	}
</style>
