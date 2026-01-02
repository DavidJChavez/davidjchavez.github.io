<script lang="ts">
	enum ConvertionType {
		AUTO,
		PLAIN_TEXT,
		PDF,
		JPG,
		PNG,
		GIF,
		ZIP
	}
	const catConvertionTypes = [
		{ value: ConvertionType.AUTO, name: 'Auto' },
		{ value: ConvertionType.PLAIN_TEXT, name: 'Plain text' },
		{ value: ConvertionType.PDF, name: 'PDF' },
		{ value: ConvertionType.JPG, name: 'JPG/JPEG' },
		{ value: ConvertionType.PNG, name: 'PNG' },
		{ value: ConvertionType.GIF, name: 'GIF' },
		{ value: ConvertionType.ZIP, name: 'ZIP' }
	];
	const signatures: Record<string, ConvertionType> = {
		JVBERi0: ConvertionType.PDF,
		'/9j/': ConvertionType.JPG,
		iVBORw0KGgo: ConvertionType.PNG,
		R0lGOD: ConvertionType.GIF,
		UEsDBB: ConvertionType.ZIP
	};

	let rawInput = $state('');
	let tipoConversion = $state<ConvertionType | null>(null);
	let filetypeRecognized = $state<ConvertionType | null>(null);
	let debouncedValue = $state<string | null>(null);

	const calculateConvertionType = (base64: string): ConvertionType => {
		for (let s in signatures) {
			console.log(s);
			if (base64.startsWith(s)) {
				return signatures[s];
			}
		}

		return ConvertionType.PLAIN_TEXT;
	};

	const convertionTypeName = (ct: ConvertionType): string => {
		return catConvertionTypes.find((c) => c.value === ct)?.name ?? 'Unkown type';
	};

	const base64ToUnicode = (base64: string) => {
		const binaryString = atob(base64);
		const binaryStringLen = binaryString.length;
		const bytes = new Uint8Array(binaryStringLen);
		for (let i = 0; i < binaryStringLen; i++) {
			bytes[i] = binaryString.charCodeAt(i);
		}

		return new TextDecoder().decode(bytes);
	};

	$effect(() => {
		let newValue = rawInput;
		let newConvertionType = tipoConversion;
		const timer = setTimeout(() => {
			debouncedValue = newValue;
			if (newConvertionType === ConvertionType.AUTO) {
				filetypeRecognized = calculateConvertionType(newValue);
			} else {
				filetypeRecognized = null;
			}
		}, 500);

		return () => clearTimeout(timer);
	});
</script>

<div class="flex h-full gap-4 p-4">
	<div class="flex flex-1 flex-col">
		<label for="entrada">Input:</label>
		<textarea id="entrada" bind:value={rawInput} class="flex-1 resize-none border"></textarea>
	</div>
	<div class="flex w-40 flex-col items-center justify-center gap-2">
		<label for="filetype">File type</label>
		<select id="filetype" bind:value={tipoConversion} class="w-full border">
			<option value={null} disabled hidden selected></option>
			{#each catConvertionTypes as option}
				<option value={option.value}>{option.name}</option>
			{/each}
		</select>
	</div>
	<div class="flex flex-1 flex-col items-center justify-center">
		{#if tipoConversion === null}
			<div class="text-xl">Selecciona el tipo de conversión</div>
		{:else if !debouncedValue}
			<div class="text-xl">Aquí verás el resultado</div>
		{:else}
			<div>
				Output:
				{#if filetypeRecognized !== null}
					<span class="italic">
						({convertionTypeName(filetypeRecognized)} recognized)
					</span>
				{/if}
			</div>
			<div class="flex w-full flex-1 items-center justify-center">
				{#if tipoConversion === ConvertionType.PLAIN_TEXT || filetypeRecognized === ConvertionType.PLAIN_TEXT}
					<div class="border border-gray-200 p-2 wrap-anywhere w-full h-full">
						{base64ToUnicode(debouncedValue)}
					</div>
				{:else if tipoConversion === ConvertionType.PDF || filetypeRecognized === ConvertionType.PDF}
					<embed
						src={`data:application/pdf;base64,${debouncedValue}`}
						type="application/pdf"
						class="h-full w-full flex-1"
					/>
				{:else if tipoConversion === ConvertionType.JPG || filetypeRecognized === ConvertionType.JPG}
					<img src={`data:image/jpeg;base64,${debouncedValue}`} alt="Base64_Image" />
				{:else if tipoConversion === ConvertionType.PNG || filetypeRecognized === ConvertionType.PNG}
					<img src={`data:image/png;base64,${debouncedValue}`} alt="Base64_Image" />
				{:else if tipoConversion === ConvertionType.GIF || filetypeRecognized === ConvertionType.GIF}{:else if tipoConversion === ConvertionType.ZIP || filetypeRecognized === ConvertionType.ZIP}{/if}
			</div>
		{/if}
	</div>
</div>
