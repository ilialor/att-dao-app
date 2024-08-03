<script>
	import { _client_canister_actor } from './+page.js';
	import { loginII, logout, isAuthenticated } from '../auth.js';
	import { Principal } from '@dfinity/principal';
	import '../index.scss';

	let client_canister = 'mmt3g-qiaaa-aaaal-qi6ra-cai';
	let loggedIn = false;
	let prevId = null;
	let includePrevId = false;
	let includeHeaders = false;
	let id = 0;
	let namespace = 'event.hub.balance';
	let timestamp = Date.now();
	let dataType = 'Text';
	let dataValue = '';
	let headers = [];
	let mapEntries = [];

	function handleLogin() {
		loginII();
	}

	function handleLogout() {
		logout();
	}

	isAuthenticated.subscribe((value) => {
		loggedIn = value;
	});

	function addHeader() {
		headers = [...headers, { fieldName: '', fieldType: 'Text', fieldValue: '' }];
	}

	function removeHeader(index) {
		headers = headers.filter((_, i) => i !== index);
	}

	function updateHeader(index, field, value) {
		headers = headers.map((header, i) => (i === index ? { ...header, [field]: value } : header));
	}

	function addMapEntry() {
		mapEntries = [...mapEntries, { key: '', value: '', type: 'Text' }];
	}

	function removeMapEntry(index) {
		mapEntries = mapEntries.filter((_, i) => i !== index);
	}

	function updateMapEntry(index, field, value) {
		mapEntries = mapEntries.map((entry, i) => (i === index ? { ...entry, [field]: value } : entry));
	}

	function convertToICRC16(type, value) {
		switch (type) {
			case 'Text':
				return { Text: value };
			case 'Nat':
				return { Nat: parseInt(value) };
			case 'Int':
				return { Int: parseInt(value) };
			case 'Bool':
				return { Bool: value === 'true' };
			case 'Map':
				return {
					Map: mapEntries.map((entry) => [entry.key, convertToICRC16(entry.type, entry.value)])
				};
			// Add more cases for other ICRC16 types as needed
			default:
				return { Text: value };
		}
	}

	async function handleSubmit() {
		const event = {
			id: Number(id),
			prevId: includePrevId ? [Number(prevId)] : [],
			timestamp: BigInt(timestamp),
			namespace: namespace,
			source: Principal.fromText(client_canister),
			data: convertToICRC16(dataType, dataValue),
			headers: includeHeaders
				? [headers.map((h) => [h.fieldName, convertToICRC16(h.fieldType, h.fieldValue)])]
				: []
		};
		try {
			const result = await _client_canister_actor.publish(event);
			if ('ok' in result) {
				console.log('Event published successfully. IDs:', result.ok);
				// Handle successful publication (e.g., show a success message)
			} else {
				console.error('Error publishing event:', result.err);
				// Handle error (e.g., show an error message to the user)
			}
		} catch (error) {
			console.error('Error calling publish method:', error);
			// Handle unexpected errors
		}
	}
</script>

<main style="margin-top: 4rem;">
	{#if loggedIn}
		<div>
			<div class="event-form">
				<div class="input-group">
					<label for="id">ID</label>
					<input type="number" id="id" bind:value={id} />
				</div>
				<div class="checkbox-group">
					<label for="includePrevId">Add Previous ID?</label>
					<input id="includePrevId" type="checkbox" bind:checked={includePrevId} />
				</div>
				{#if includePrevId}
					<div class="input-group">
						<label for="prevId">Previous ID</label>
						<input id="prevId" type="number" bind:value={prevId} />
					</div>
				{/if}
				<div class="input-group">
					<label for="timestamp">Timestamp</label>
					<input id="timestamp" type="number" bind:value={timestamp} required />
				</div>
				<div class="input-group">
					<label for="namespace">Namespace</label>
					<input type="text" id="namespace" bind:value={namespace} />
				</div>
				<div class="input-group">
					<label for="dataType">Data Type</label>
					<select id="dataType" bind:value={dataType}>
						<option value="Text">Text</option>
						<option value="Nat">Nat</option>
						<option value="Int">Int</option>
						<option value="Bool">Bool</option>
						<option value="Map">Map</option>
						<!-- Add more options as needed -->
					</select>
				</div>

				{#if dataType !== 'Map'}
					<div class="input-group">
						<label for="dataValue">Data Value</label>
						<input type="text" id="dataValue" bind:value={dataValue} />
					</div>
				{:else}
					<div class="map-entries">
						<button on:click={addMapEntry}>Add Map Entry</button>
						{#each mapEntries as entry, index}
							<div class="map-entry">
								<input
									type="text"
									placeholder="Key"
									value={entry.key}
									on:input={(e) => updateMapEntry(index, 'key', e.target.value)}
								/>
								<select
									value={entry.type}
									on:change={(e) => updateMapEntry(index, 'type', e.target.value)}
								>
									<option value="Text">Text</option>
									<option value="Nat">Nat</option>
									<option value="Int">Int</option>
									<option value="Bool">Bool</option>
								</select>
								<input
									type="text"
									placeholder="Value"
									value={entry.value}
									on:input={(e) => updateMapEntry(index, 'value', e.target.value)}
								/>
								<button on:click={() => removeMapEntry(index)}>Remove</button>
							</div>
						{/each}
					</div>
				{/if}

				<!-- <div class="input-group">
					<label for="dataValue">Data Value</label>
					<input type="text" id="dataValue" bind:value={dataValue} />
				</div> -->
				<div class="checkbox-group">
					<label for="includeHeaders">Add Headers?</label>
					<input id="includeHeaders" type="checkbox" bind:checked={includeHeaders} />
				</div>
				{#if includeHeaders}
					<button on:click={addHeader}>Add Header</button>
					{#each headers as header, index}
						<div class="input-group header-group">
							<div class="field-group">
								<label for={`fieldName${index}`}>Header Name</label>
								<input
									id={`fieldName${index}`}
									type="text"
									value={header.fieldName}
									on:input={(e) => updateHeader(index, 'fieldName', e.target.value)}
								/>
							</div>
							<div class="field-group">
								<label for={`fieldType${index}`}>Header Data Type</label>
								<select
									id={`fieldType${index}`}
									value={header.fieldType}
									on:change={(e) => updateHeader(index, 'fieldType', e.target.value)}
								>
									<option value="Text">Text</option>
									<option value="Nat">Nat</option>
									<option value="Int">Int</option>
									<option value="Bool">Bool</option>
								</select>
							</div>
							<div class="field-group">
								<label for={`fieldValue${index}`}>Header Value</label>
								<input
									id={`fieldValue${index}`}
									type="text"
									value={header.fieldValue}
									on:input={(e) => updateHeader(index, 'fieldValue', e.target.value)}
								/>
							</div>
							<button on:click={() => removeHeader(index)}>Remove</button>
						</div>
					{/each}
				{/if}
				<button on:click={handleSubmit}>Publish</button>
			</div>
			<button class="logout" on:click={handleLogout}> Logout</button>
		</div>
	{:else}
		<button class="login" on:click={handleLogin}> Login with Internet Identity</button>
	{/if}
</main>
