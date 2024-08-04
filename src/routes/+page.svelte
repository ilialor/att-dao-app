<script>
	import { onMount } from 'svelte';
	import { writable, get } from 'svelte/store';
	import { isLoading, handleNotifications } from '$lib/notification-store';
	import Notification from './components/Notification.svelte';
	import CustomTypography from './components/CustomTypography.svelte';
	import { isAuthenticated, principalId, broadcaster, broadcaster_canister_actor } from './auth.js';

	let principal = '';
	let messagesMapStore = writable(new Map());
	let loggedIn = false;

	principalId.subscribe((value) => {
		principal = value;
	});

	isAuthenticated.subscribe((value) => {
		loggedIn = value;
	});

	const TARGET_PRINCIPAL = 'mmt3g-qiaaa-aaaal-qi6ra-cai';

	onMount(() => {
		console.log('onMount start');
		handleNotificationsImpl();
	});

	async function handleNotificationsImpl() {
		isLoading.set(true);
		let actor = broadcaster_canister_actor;
		try {
			if (!broadcaster_canister_actor) {
				actor = await broadcaster();
			}
			console.log('Getting notifications for user: ', TARGET_PRINCIPAL);
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
			isLoading.set(false);
		}
	}

	handleNotifications.set(handleNotificationsImpl);

	$: notifications = ($messagesMapStore.get(TARGET_PRINCIPAL) || []).map((notification, index) => ({
		...notification,
		uniqueId: `${notification.id || ''}-${index}`
	}));

	function handleReaction(event) {
		const { notificationId, reaction } = event.detail;
		console.log(`Reaction for notification ${notificationId}:`, reaction);
		// Здесь вы можете реализовать логику для обработки реакции,
		// например, отправку новой публикации на основе выбранной реакции
	}
</script>

<div class="notifications-container">
	<CustomTypography variant="h4">Notifications for Principal</CustomTypography>
	<CustomTypography variant="body1" class="principal-id">{TARGET_PRINCIPAL}</CustomTypography>

	{#if $isLoading}
		<CustomTypography variant="body1">Loading notifications...</CustomTypography>
	{:else if notifications.length > 0}
		<CustomTypography variant="body1" class="notification-count">
			Total notifications: {notifications.length}
		</CustomTypography>
		<div class="card-list">
			{#each notifications as notification (notification.uniqueId)}
				<Notification {notification} on:reaction={handleReaction} />
			{/each}
		</div>
	{:else}
		<CustomTypography variant="body1"
			>No notifications available for this principal</CustomTypography
		>
	{/if}
</div>

<style>
	.notifications-container {
		max-width: 800px;
		margin: 0 auto;
		padding: 20px;
	}

	.card-list {
		display: flex;
		flex-direction: column;
		gap: 20px;
	}
</style>
