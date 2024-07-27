<script>
	import {
		loginII,
		logout,
		isAuthenticated,
		principalId,
		dao_backend,
		client_canister_actor
	} from '../auth.js';
	import copy_icon from '$lib/images/copy_icon.png';
	import '../index.scss';

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

	let isLoading = false;

	async function dao() {
		isLoading = true;
		let actor = client_canister_actor;
		try {
			if (!client_canister_actor) {
				// @ts-ignore
				actor = await dao_backend();
			}
			// @ts-ignore
			let members = await actor.listMembers();
			console.log('Members: ', members);
		} catch (error) {
			console.error('Error in DAO function:', error);
		} finally {
			isLoading = false;
		}
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
				<br />
				<button class="dao-button" on:click={dao} disabled={isLoading}>
					{isLoading ? 'Loading...' : 'DAO Members'}
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
