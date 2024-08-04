<script>
	// import { onMount } from 'svelte';
	import CustomTypography from './CustomTypography.svelte';
	import CustomCard from './CustomCard.svelte';

	export let notification;

	let showDetails = false;

	function toggleDetails() {
		showDetails = !showDetails;
	}

	function formatTimestamp(timestamp) {
		return new Date(Number(timestamp)).toLocaleString();
	}

	function renderICRC16(data) {
		if (data === null || data === undefined) {
			return 'N/A';
		}
		if (typeof data === 'object') {
			if (Array.isArray(data)) {
				return `[${data.map(renderICRC16).join(', ')}]`;
			} else if (data instanceof Uint8Array) {
				return `Blob(${data.length} bytes)`;
			} else if (data.constructor === Object) {
				return JSON.stringify(data, (key, value) =>
					typeof value === 'bigint' ? value.toString() : value
				);
			}
		}
		return typeof data === 'bigint' ? data.toString() : String(data);
	}

	// onMount(() => {
	// 	console.log('Notification mounted:', notification);
	// });
</script>

<CustomCard>
	<CustomTypography variant="h6" class="card-title">
		Event ID: {notification.eventId || 'N/A'}
	</CustomTypography>
	<CustomTypography variant="body2">
		Timestamp: {formatTimestamp(notification.timestamp)}
	</CustomTypography>
	<CustomTypography variant="body2">
		Namespace: {notification.namespace || 'N/A'}
	</CustomTypography>
	<button on:click={toggleDetails}>{showDetails ? 'Hide' : 'Show'} Details</button>
	{#if showDetails}
		<details open>
			<summary>Data</summary>
			<div class="data-section">
				<pre>{renderICRC16(notification.data)}</pre>
			</div>
		</details>
		{#if notification.headers}
			<details>
				<summary>Headers</summary>
				<div class="headers-section">
					<pre>{renderICRC16(notification.headers)}</pre>
				</div>
			</details>
		{/if}
	{/if}
</CustomCard>

<style>
	button {
		background-color: #4caf50;
		border: none;
		color: white;
		padding: 10px 20px;
		text-align: center;
		text-decoration: none;
		display: inline-block;
		font-size: 16px;
		margin: 4px 2px;
		cursor: pointer;
		border-radius: 4px;
	}

	.data-section,
	.headers-section {
		background-color: #f8f9fa;
		border-radius: 4px;
		padding: 10px;
		margin-top: 10px;
	}

	pre {
		white-space: pre-wrap;
		word-break: break-word;
	}
</style>
