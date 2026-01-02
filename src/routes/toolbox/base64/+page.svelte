<script lang="ts">
	enum TipoConversion {
		PDF = 1,
		JPG
	}
	const catTiposConversion = [
		{ value: TipoConversion.PDF, name: 'PDF' },
		{ value: TipoConversion.JPG, name: 'JPG' }
	];

	let rawInput = $state('');
	let tipoConversion = $state('');
	let debouncedValue = $state<string | null>(null);

	$effect(() => {
		let newValue = rawInput;
		const timer = setTimeout(() => {
			debouncedValue = newValue;
		}, 500);

		return () => clearTimeout(timer);
	});
</script>

<div class="flex h-full gap-4 p-4">
	<div class="flex flex-1 flex-col">
		<label for="entrada">Entrada:</label>
		<textarea id="entrada" bind:value={rawInput} class="flex-1 resize-none border"
		></textarea>
	</div>
	<div class="flex w-40 flex-col items-center justify-center gap-2">
		<select bind:value={tipoConversion} class="w-full border">
			<option value="" disabled selected hidden>Tipo</option>
			{#each catTiposConversion as option}
				<option value={option.value}>{option.name}</option>
			{/each}
		</select>
	</div>
	<div class="flex flex-1 flex-col">
		{#if !tipoConversion}
			<div class="text-xl">Selecciona el tipo de conversión</div>
		{:else if !debouncedValue}
			<div class="text-xl">Aquí verás el resultado</div>
		{:else}
			<div>Salida:</div>
			<embed
				src={`data:application/pdf;base64,${debouncedValue}`}
				type="application/pdf"
				class="flex-1"
			/>
		{/if}
	</div>
</div>
