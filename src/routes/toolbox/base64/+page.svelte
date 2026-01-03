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

<div class="mb-4 text-4xl italic">Base64 encoder/decoder</div>

<div class="mb-2 text-3xl">Decoder</div>

<div class="flex flex-col gap-4 mb-4">
	<div class="flex w-sm flex-col gap-0.5">
		<label for="filetype"> File type </label>
		<select id="filetype" bind:value={tipoConversion} class="border">
			<option value={null} disabled selected>Please select...</option>
			{#each catConvertionTypes as option}
				<option value={option.value}>{option.name}</option>
			{/each}
		</select>
	</div>

	<div class="grid h-100 grid-cols-2 place-items-stretch gap-4">
		<div class="flex flex-col gap-0.5">
			<label for="entrada">Input:</label>
			<textarea id="entrada" bind:value={rawInput} class="flex-1 resize-none border"></textarea>
		</div>

		<div class="flex flex-col gap-0.5">
			<div>
				Output:
				{#if filetypeRecognized !== null}
					<span class="italic">
						({convertionTypeName(filetypeRecognized)} recognized)
					</span>
				{/if}
			</div>
			<div class="flex flex-1 border border-gray-200">
				{#if tipoConversion === null}
					<div class="p-2">Selecciona el tipo de conversión</div>
				{:else if !debouncedValue}
					<div class="p-2">Aquí verás el resultado</div>
				{:else}
					<div class="flex w-full flex-1 items-center justify-center">
						{#if tipoConversion === ConvertionType.PLAIN_TEXT || filetypeRecognized === ConvertionType.PLAIN_TEXT}
							<div class="h-full w-full border border-gray-200 p-2 wrap-anywhere">
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
	</div>
</div>

<div class="mb-2 text-3xl">Encoder</div>

// TODO: Con calmita xd