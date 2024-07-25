<script>
	// import { _client_canister_actor } from './+page.js';
	import { loginII, logout, isAuthenticated, principalId } from './auth.js';
	import copy_icon from '$lib/images/copy_icon.png';
	import './index.scss';

	let principal = '';

	principalId.subscribe((value) => {
		principal = value;
	});

	let loggedIn = false;

	function handleLogin() {
		loginII();
	}

	function handleLogout() {
		logout();
	}
	isAuthenticated.subscribe((value) => {
		loggedIn = value;
	});

	async function copyValue() {
		await navigator.clipboard.writeText(principal);
		alert('ID скопирован: ' + principal);
	}
</script>

<svelte:head>
	<title>Attention DAO Client</title>
	<meta name="description" content="Attention DAO demo app" />
</svelte:head>

<main>
	<div class="main-container">
		{#if loggedIn}
			<h2>Your Attention DAO id is:</h2>
			<span class="user-principal-container">
				<span class="principal-value">{principal}</span>
				<button class="copy-button" on:click={copyValue}>
					<img id="copy-icon" src={copy_icon} alt="Copy ID" />
				</button>
			</span>
			<h2>Save your this id and Internet Identity number for later use. <br /></h2>
			<br />
			<button class="logout" on:click={handleLogout}> Logout</button>
		{:else}
			<button class="login" on:click={handleLogin}> Please Login with Internet Identity</button>
		{/if}
	</div>
</main>
