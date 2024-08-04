<script>
	import { onMount } from 'svelte';
	import CustomCard from './CustomCard.svelte';
	import CustomTypography from './CustomTypography.svelte';
	import CustomButton from './CustomButton.svelte';
	import CustomBox from './CustomBox.svelte';

	export let notification;

	let isExpanded = false;

	function formatTimestamp(timestamp) {
		return new Date(Number(timestamp)).toLocaleString();
	}

	function renderICRC16(data) {
		if (typeof data === 'object' && data !== null) {
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

	function renderReactionButtons() {
		if (!notification.data || !notification.data.Map) return [];

		const expectedReactions = notification.data.Map.find(([key]) => key === 'expectedReactions');
		if (!expectedReactions || !expectedReactions[1].Array) return [];

		return expectedReactions[1].Array.map((reaction) => {
			const reactionMap = reaction.Map;
			if (!reactionMap) return null;

			const template = reactionMap.find(([key]) => key === 'template')[1];
			const price = reactionMap.find(([key]) => key === 'price')[1];
			const templateName =
				typeof template === 'object'
					? template.Map.find(([key]) => key === 'type')[1].Text
					: template.Text;

			return {
				templateName,
				price: renderICRC16(price),
				reaction
			};
		});
	}

	function handleReaction(reaction) {
		console.log('Reaction:', reaction);
	}

	let reactionButtons;

	onMount(() => {
		reactionButtons = renderReactionButtons();
	});

	function toggleExpanded() {
		isExpanded = !isExpanded;
	}
</script>

<CustomCard className="mb-4">
	<CustomTypography variant="h6">Event ID: {notification.eventId}</CustomTypography>
	<CustomTypography variant="body2">
		Timestamp: {formatTimestamp(notification.timestamp)}
	</CustomTypography>
	<CustomTypography variant="body2">Namespace: {notification.namespace}</CustomTypography>

	<CustomButton onClick={toggleExpanded}>
		{isExpanded ? 'Hide Details' : 'Show Details'}
	</CustomButton>

	{#if isExpanded}
		<CustomBox mt={2}>
			<CustomTypography variant="body2">ID: {notification.id}</CustomTypography>
			<CustomTypography variant="body2">
				Source: {notification.source.toText()}
			</CustomTypography>
			<CustomTypography variant="body2">
				Pre-Event ID: {notification.preEventId ? notification.preEventId : 'None'}
			</CustomTypography>
			<CustomTypography variant="body2">
				Filter: {notification.filter ? notification.filter : 'None'}
			</CustomTypography>
			<CustomBox mt={2}>
				<CustomTypography variant="h6">Content:</CustomTypography>
				<CustomTypography variant="body2">
					{#if notification.data && notification.data.Map}
						{renderICRC16(notification.data.Map.find(([key]) => key === 'content')[1])}
					{:else}
						No content available
					{/if}
				</CustomTypography>
			</CustomBox>
			{#if notification.headers}
				<CustomBox mt={2}>
					<CustomTypography variant="h6">Headers:</CustomTypography>
					<pre>{renderICRC16(notification.headers)}</pre>
				</CustomBox>
			{/if}
			<CustomBox mt={2}>
				<CustomTypography variant="h6">Reactions:</CustomTypography>
				{#each reactionButtons as button}
					<CustomButton onClick={() => handleReaction(button.reaction)} className="mr-2 mb-2">
						{button.templateName}
						<CustomTypography variant="body2">
							Price: {button.price}
						</CustomTypography>
					</CustomButton>
				{/each}
			</CustomBox>
		</CustomBox>
	{/if}
</CustomCard>
