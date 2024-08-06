<script>
	import { createEventDispatcher } from 'svelte';
	import LikeDislikeReaction from './LikeDislikeReaction.svelte';
	import TextFeedbackReaction from './TextFeedbackReaction.svelte';
	import RegistrationFormReaction from './RegistrationFormReaction.svelte';

	export let reactions = [];
	export let disabled = false;
	const dispatch = createEventDispatcher();

	let selectedReaction = null;

	function handleReactionClick(reaction) {
		selectedReaction = reaction;
	}

	function handleReactionSubmit(event) {
		dispatch('reaction', event.detail);
		selectedReaction = null;
	}

	function handleReaction(reaction) {
		if (!disabled) {
			dispatch('reaction', reaction);
		}
	}

	function parseReactionTemplate(template) {
		console.log('Parsing reaction template:', template);
		if (typeof template === 'string') {
			return template;
		} else if (template instanceof Map) {
			return template.get('type') || 'Unknown Template';
		} else if (typeof template === 'object' && template !== null) {
			if (template.Map) {
				const templateMap = new Map(template.Map);
				return templateMap.get('type') || 'Unknown Template';
			}
			return template.type || 'Unknown Template';
		}
		return 'Unknown Template';
	}

	function parseMapTemplate(map) {
		if (map.has('type')) {
			return map.get('type');
		} else if (map.has('Text')) {
			return map.get('Text');
		}
		for (let [key, value] of map.entries()) {
			if (typeof value === 'string') {
				return value;
			} else if (typeof value === 'object') {
				let result = parseObjectTemplate(value);
				if (result !== 'Unknown Template') {
					return result;
				}
			}
		}
		return 'Unknown Template';
	}

	function parseObjectTemplate(obj) {
		if (obj.type) {
			return obj.type;
		} else if (obj.Text) {
			return obj.Text;
		} else if (obj.Map) {
			return parseMapTemplate(new Map(obj.Map));
		}
		for (let key in obj) {
			if (typeof obj[key] === 'string') {
				return obj[key];
			} else if (typeof obj[key] === 'object') {
				let result = parseObjectTemplate(obj[key]);
				if (result !== 'Unknown Template') {
					return result;
				}
			}
		}
		return 'Unknown Template';
	}
</script>

<div class="reactions">
	{#each reactions as reaction}
		<button class="reaction-button" on:click={() => handleReactionClick(reaction)} {disabled}>
			<span class="template">{parseReactionTemplate(reaction.template)}</span>
			<span class="price">{reaction.price} FOCUS</span>
		</button>
	{/each}
</div>

{#if selectedReaction}
	<div class="reaction-component">
		{#if parseReactionTemplate(selectedReaction.template) === 'LikeDislike'}
			<LikeDislikeReaction
				reaction={selectedReaction}
				{disabled}
				on:reaction={handleReactionSubmit}
			/>
		{:else if parseReactionTemplate(selectedReaction.template) === 'TextFeedback' || parseReactionTemplate(selectedReaction.template) === 'Custom'}
			<TextFeedbackReaction
				reaction={selectedReaction}
				{disabled}
				on:reaction={handleReactionSubmit}
			/>
		{:else if parseReactionTemplate(selectedReaction.template) === 'RegistrationForm'}
			<RegistrationFormReaction
				reaction={selectedReaction}
				{disabled}
				on:reaction={handleReactionSubmit}
			/>
		{/if}
	</div>
{/if}

<style>
	.reactions {
		display: flex;
		flex-wrap: wrap;
		gap: 10px;
		margin-top: 10px;
	}
	.reaction-button {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 10px;
		border: 1px solid #ccc;
		border-radius: 5px;
		background-color: #f0f0f0;
		cursor: pointer;
		transition: background-color 0.3s;
	}
	.reaction-button:hover {
		background-color: #e0e0e0;
	}
	.template {
		font-weight: bold;
	}
	.price {
		font-size: 0.8em;
		color: #666;
	}
	.reaction-button:disabled {
		opacity: 0.5;
		cursor: not-allowed;
	}
	.reaction-component {
		margin-top: 10px;
	}
</style>
