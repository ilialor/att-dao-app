<script>
	import {
		loginII,
		logout,
		isAuthenticated,
		principalId,
		broadcaster,
		broadcaster_canister_actor
	} from './auth.js';
	import copy_icon from '$lib/images/copy_icon.png';
	import './index.scss';
	import { onMount } from 'svelte';
	import { writable } from 'svelte/store';

	let principal = '';
	let messagesMapStore = writable(new Map());
	let isLoading = false;
	let loggedIn = false;

	principalId.subscribe((value) => {
		principal = value;
	});

	isAuthenticated.subscribe((value) => {
		loggedIn = value;
	});

	function handleLogin() {
		loginII();
	}

	function handleLogout() {
		logout();
	}

	async function copyValue() {
		await navigator.clipboard.writeText(principal);
		alert('ID скопирован: ' + principal);
	}

	const TARGET_PRINCIPAL = 'mmt3g-qiaaa-aaaal-qi6ra-cai';

	$: notifications = $messagesMapStore.get(TARGET_PRINCIPAL) || [];

	onMount(() => {
		console.log('onMount start');
		handleNotifications();
	});

	async function handleNotifications() {
		isLoading = true;
		let actor = broadcaster_canister_actor;
		try {
			if (!broadcaster_canister_actor) {
				// @ts-ignore
				actor = await broadcaster();
			}
			console.log('Getting notifications for user: ', TARGET_PRINCIPAL);
			// @ts-ignore
			let result = await actor.getNotificationsByUser(TARGET_PRINCIPAL);
			console.log('Notifications: ', result);
			if (Array.isArray(result[1])) {
				messagesMapStore.update((map) => {
					return new Map(map.set(TARGET_PRINCIPAL, result[1]));
				});
			} else {
				console.error('Unexpected result format:', result);
			}
		} catch (error) {
			console.error('Error when getting notifications: ', error);
		} finally {
			isLoading = false;
		}
	}

	// @ts-ignore
	function formatTimestamp(timestamp) {
		return new Date(Number(timestamp)).toLocaleString();
	}

	// @ts-ignore
	function renderICRC16(data) {
		if (typeof data === 'object' && data !== null) {
			if (Array.isArray(data)) {
				return `[${data.map(renderICRC16).join(', ')}]`;
			} else if (data instanceof Uint8Array) {
				return `Blob(${data.length} bytes)`;
			} else if (data.constructor === Object) {
				// @ts-ignore
				return JSON.stringify(data, (key, value) =>
					typeof value === 'bigint' ? value.toString() : value
				);
			}
		}
		return typeof data === 'bigint' ? data.toString() : String(data);
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
			<button class="notification-button" on:click={handleNotifications} disabled={isLoading}>
				{isLoading ? 'Loading...' : 'Notifications'}
			</button>
			<button class="logout" on:click={handleLogout}> Logout</button>
		{:else}
			<button class="login" on:click={handleLogin}> Login with Internet Identity</button>
		{/if}
	</div>
	<br />
	<div class="notifications-container">
		<h1>Notifications for Principal</h1>
		<p class="principal-id">{TARGET_PRINCIPAL}</p>
		{#if notifications.length > 0}
			<p class="notification-count">
				Total notifications: {notifications.length}
			</p>
			<div class="card-list">
				{#each notifications as notification}
					<div class="card">
						<h2 class="card-title">Event ID: {notification.eventId}</h2>
						<p>
							<strong>Timestamp:</strong>
							{formatTimestamp(notification.timestamp)}
						</p>
						<p><strong>Namespace:</strong> {notification.namespace}</p>
						<details>
							<summary>More details</summary>
							<p><strong>ID:</strong> {notification.id}</p>
							<p>
								<strong>Source:</strong>
								{notification.source.toText()}
							</p>
							<p>
								<strong>Pre-Event ID:</strong>
								{notification.preEventId ? notification.preEventId : 'None'}
							</p>
							<p>
								<strong>Filter:</strong>
								{notification.filter ? notification.filter : 'None'}
							</p>
							<div class="data-section">
								<h4>Data:</h4>
								<pre>{renderICRC16(notification.data)}</pre>
							</div>
							{#if notification.headers}
								<div class="headers-section">
									<h4>Headers:</h4>
									<pre>{renderICRC16(notification.headers)}</pre>
								</div>
							{/if}
						</details>
					</div>
				{/each}
			</div>
		{:else}
			<p>No notifications available for this principal</p>
		{/if}
	</div>
</main>
