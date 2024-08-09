<script>
	import { onMount } from 'svelte';
	import { Principal } from '@dfinity/principal';
	import { client_canister, client_canister_actor, CLIENT_CANISTER_ID } from '../auth.js';
	import ReactionDisplay from './ReactionDisplay.svelte';

	let publishing = false;
	let result = '';
	let parsedContent = '';
	let parsedReactions = [];

	// const client_canister = 'mmt3g-qiaaa-aaaal-qi6ra-cai';

	function convertToICRC16(data) {
		if (data === null || data === undefined) {
			console.warn('Received null or undefined data in convertToICRC16');
			return { Text: '' };
		}

		if (typeof data !== 'object') {
			return { Text: String(data) };
		}

		switch (data.type) {
			case 'Text':
				return { Text: String(data.value) };
			case 'Nat':
				return { Nat: BigInt(data.value) };
			case 'Int':
				return { Int: BigInt(data.value) };
			case 'Bool':
				return { Bool: Boolean(data.value) };
			case 'Principal':
				return { Text: String(data.value) };
			case 'Map':
				return {
					Map: Object.entries(data.value).map(([key, val]) => [key, convertToICRC16(val)])
				};
			case 'Array':
				if (!Array.isArray(data.value)) {
					console.warn('Array value is not an array:', data.value);
					return { Array: [] };
				}
				return { Array: data.value.map(convertToICRC16) };
			default:
				console.warn('Unknown data type:', data.type);
				return { Text: JSON.stringify(data) };
		}
	}

	const testData = {
		id: 150,
		prevId: 0,
		timestamp: 1625097600000000,
		namespace: 'test',
		data: {
			type: 'Map',
			value: {
				content: { type: 'Text', value: 'Test message with reaction' },
				expectedReactions: {
					type: 'Array',
					value: [
						{
							type: 'Map',
							value: {
								namespace: { type: 'Text', value: 'feedback.test' },
								template: { type: 'Text', value: 'TextFeedback' },
								price: { type: 'Nat', value: '10' },
								recipient: { type: 'Principal', value: 'mmt3g-qiaaa-aaaal-qi6ra-cai' }
							}
						},
						{
							type: 'Map',
							value: {
								namespace: { type: 'Text', value: 'rating.test' },
								template: { type: 'Text', value: 'LikeDislike' },
								price: { type: 'Nat', value: '1' },
								recipient: { type: 'Principal', value: 'mmt3g-qiaaa-aaaal-qi6ra-cai' }
							}
						},
						{
							type: 'Map',
							value: {
								namespace: { type: 'Text', value: 'registration.test' },
								template: {
									type: 'Map',
									value: {
										type: { type: 'Text', value: 'RegistrationForm' },
										fields: {
											type: 'Array',
											value: [
												{
													type: 'Map',
													value: {
														name: { type: 'Text', value: 'Full Name' },
														fieldType: { type: 'Text', value: 'Text' },
														required: { type: 'Bool', value: 'true' }
													}
												},
												{
													type: 'Map',
													value: {
														name: { type: 'Text', value: 'Age' },
														fieldType: { type: 'Text', value: 'Number' },
														required: { type: 'Bool', value: 'true' }
													}
												},
												{
													type: 'Map',
													value: {
														name: { type: 'Text', value: 'Subscribe to hackathon' },
														fieldType: { type: 'Text', value: 'Boolean' },
														required: { type: 'Bool', value: 'false' }
													}
												}
											]
										}
									}
								},
								price: { type: 'Nat', value: '20' },
								recipient: { type: 'Principal', value: 'mmt3g-qiaaa-aaaal-qi6ra-cai' }
							}
						},
						{
							type: 'Map',
							value: {
								namespace: { type: 'Text', value: 'custom.test' },
								template: {
									type: 'Map',
									value: {
										type: { type: 'Text', value: 'Custom' },
										details: { type: 'Text', value: 'Custom reaction template' }
									}
								},
								price: { type: 'Nat', value: '50' },
								recipient: { type: 'Principal', value: 'mmt3g-qiaaa-aaaal-qi6ra-cai' }
							}
						}
					]
				}
			}
		},
		headers: [{ fieldName: 'Content-Type', fieldType: 'Text', fieldValue: 'application/json' }]
	};

	function bigIntReplacer(key, value) {
		if (typeof value === 'bigint') {
			return value.toString() + 'n';
		}
		return value;
	}

	function parsePublicationResult(data) {
		if (typeof data === 'object' && data !== null) {
			if ('Map' in data) {
				const map = new Map(data.Map);
				parsedContent = map.get('content')?.Text || '';
				const expectedReactions = map.get('expectedReactions')?.Array || [];
				parsedReactions = expectedReactions
					.map((reaction) => {
						if ('Map' in reaction) {
							const reactionMap = new Map(reaction.Map);
							return {
								namespace: reactionMap.get('namespace')?.Text || '',
								template:
									reactionMap.get('template')?.Text || reactionMap.get('template')?.Map || '',
								price: reactionMap.get('price')?.Nat || 0,
								recipient: reactionMap.get('recipient')?.Principal || ''
							};
						}
						return null;
					})
					.filter((r) => r !== null);
			}
		}
	}

	function handleReaction(event) {
		console.log('Reaction selected:', event.detail);
		// Here you can implement logic to create a new publication based on the selected reaction
	}

	async function sendTestPublication() {
		publishing = true;
		result = '';
		try {
			const event = {
				id: Number(testData.id),
				prevId: [Number(testData.prevId)],
				timestamp: BigInt(testData.timestamp),
				namespace: testData.namespace,
				source: Principal.fromText(CLIENT_CANISTER_ID),
				data: convertToICRC16(testData.data),
				headers:
					testData.headers.length > 0
						? [
								testData.headers.map((h) => [
									h.fieldName,
									convertToICRC16({ type: h.fieldType, value: h.fieldValue })
								])
							]
						: []
			};
			console.log(
				'Prepared event:',
				JSON.stringify(event, (key, value) =>
					typeof value === 'bigint' ? value.toString() : value
				)
			);

			let actor = client_canister_actor;
			if (!client_canister_actor) {
				console.log('Creating new client_canister actor');
				actor = await client_canister();
			}
			console.log('Actor ready, calling publish method from ', CLIENT_CANISTER_ID);
			const publishResult = await actor.publish(event);
			console.log('Publish result:', publishResult);
			result = `Publication sent successfully. Result: ${JSON.stringify(publishResult, bigIntReplacer)}`;
		} catch (error) {
			console.error('Error in sendTestPublication:', error);
			result = `Error sending publication: ${error.message}`;
		} finally {
			publishing = false;
		}
	}
</script>

<div class="test-publication">
	<h2>Test Publication</h2>
	<pre>{JSON.stringify(testData, null, 2)}</pre>
	<button on:click={sendTestPublication} disabled={publishing}>
		{publishing ? 'Sending...' : 'Send Test'}
	</button>
	{#if result}
		<div class="result">
			<h3>Result:</h3>
			<pre>{result}</pre>
		</div>
	{/if}
	{#if parsedContent}
		<div class="parsed-content">
			<h3>Content:</h3>
			<p>{parsedContent}</p>
		</div>
	{/if}
	{#if parsedReactions.length > 0}
		<div class="parsed-reactions">
			<h3>Available Reactions:</h3>
			<ReactionDisplay reactions={parsedReactions} on:reaction={handleReaction} />
		</div>
	{/if}
</div>

<style>
	.test-publication {
		max-width: 800px;
		margin: 0 auto;
		padding: 20px;
		font-family: Arial, sans-serif;
	}
	pre {
		background-color: #f4f4f4;
		padding: 10px;
		border-radius: 5px;
		overflow-x: auto;
	}
	button {
		background-color: #4caf50;
		border: none;
		color: white;
		padding: 15px 32px;
		text-align: center;
		text-decoration: none;
		display: inline-block;
		font-size: 16px;
		margin: 4px 2px;
		cursor: pointer;
		border-radius: 4px;
	}
	button:disabled {
		background-color: #cccccc;
		cursor: not-allowed;
	}
	.result,
	.parsed-content,
	.parsed-reactions {
		margin-top: 20px;
	}
</style>
