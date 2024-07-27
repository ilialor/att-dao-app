<script>
	import {
		loginII,
		logout,
		isAuthenticated,
		principalId,
		broadcaster,
		broadcaster_canister_actor
	} from './auth.js';
	// import { Principal } from '@dfinity/principal';
	import copy_icon from '$lib/images/copy_icon.png';
	import './index.scss';

	let principal = '';

	let isLoading = false;

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
	async function getNotifications() {
		isLoading = true;
		let actor = broadcaster_canister_actor;
		try {
			if (!broadcaster_canister_actor) {
				// @ts-ignore
				actor = await broadcaster();
			}
			// @ts-ignore
			console.log('Getting notifications for user: ', principal);
			// @ts-ignore
			let notifications = await actor.getReceivedMessages(); // TODO getNotificationsByUser(principal);
			console.log('Notifications: ', notifications);
		} catch (error) {
			console.log('Error when getting notifications: ', error);
		} finally {
			isLoading = false;
		}
	}
</script>

<svelte:head>
	<title>Attention Client</title>
	<meta name="description" content="Attention DAO demo app" />
</svelte:head>

<main>
	<div class="main-container">
		{#if loggedIn}
			<h2>Your Attention id is:</h2>
			<span class="user-principal-container">
				<span class="principal-value">{principal}</span>
				<button class="copy-button" on:click={copyValue}>
					<img id="copy-icon" src={copy_icon} alt="Copy ID" />
				</button>
				<br />
			</span>
			<h2>Save your this id and Internet Identity number for later use. <br /></h2>
			<br />
			<button class="notification-button" on:click={getNotifications} disabled={isLoading}>
				{isLoading ? 'Loading...' : 'Notifications'}
			</button>
			<button class="logout" on:click={handleLogout}> Logout</button>
		{:else}
			<button class="login" on:click={handleLogin}> Login with Internet Identity</button>
		{/if}
	</div>
</main>
