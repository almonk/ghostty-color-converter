<script lang="ts">
	// @ts-ignore
	import plist from 'plist/dist/plist.js';

	let input: string | ArrayBuffer | null | undefined = '';
	let output = '';
	let files: [] = [];
	let ghosttyOutputColors: any[] = [];

	$: {
		try {
			if (files.length > 0) {
				parseToGhosttyConfig(plist.parse(input));
			}
		} catch (e) {
			output = "Couldn't parse plist";
		}
	}

	$: ghosttyOutputColors = output.split('\n');

	function parseToGhosttyConfig(input: string) {
		// Find all keys beginning with "Ansi"
		const ansiKeys = Object.keys(input).filter((key) => key.startsWith('Ansi'));

		output = '';

		// For each color keys2
		ansiKeys.forEach((key) => {
			// Get the color value
			const color = parseItermColorToHexValue(input[key]);
			// Get the color name
			const colorName = key.replace('Ansi ', '').replace(' Color', '');
			// Add the color to the ghostty config
			output += `palette=${colorName}=${color}\n`;
		});

		// Find other one off keys
		Object.keys(input).forEach((key) => {
			switch (key) {
				case 'Selection Color':
					output += `selection-background=${parseItermColorToHexValue(input[key])}\n`;
					break;
				case 'Selected Text Color':
					output += `selection-foreground=${parseItermColorToHexValue(input[key])}\n`;
					break;
				case 'Cursor Color':
					output += `cursor-color=${parseItermColorToHexValue(input[key])}\n`;
					break;
				case 'Background Color':
					output += `background=${parseItermColorToHexValue(input[key])}\n`;
					break;
				case 'Foreground Color':
					output += `foreground=${parseItermColorToHexValue(input[key])}\n`;
					break;
				default:
					break;
			}
		});
	}

	function parseItermColorToHexValue(input: any) {
		let r, g, b;

		r = Math.trunc(input['Red Component'] * 255);
		g = Math.trunc(input['Green Component'] * 255);
		b = Math.trunc(input['Blue Component'] * 255);

		return rgbToHex(r, g, b);
	}

	function rgbToHex(r: number, g: number, b: number) {
		return '#' + componentToHex(r) + componentToHex(g) + componentToHex(b);
	}

	function componentToHex(c: number) {
		var hex = c.toString(16);
		return hex.length == 1 ? '0' + hex : hex;
	}

	function readFile(file: Blob) {
		const reader = new FileReader();
		reader.onload = (e) => {
			input = e.target?.result;
		};
		reader.readAsText(file);
	}
</script>

<div class="text-[28px] font-bold tracking-tight mt-10 mb-10">
	iTerm Theme ??? Ghostty Config Converter
</div>

<div class="flex flex-col gap-y-2 mt-8">
	<div class="flex gap-x-2 text-[16px]">
		<div class="font-semibold">Step 1.</div>
		<div class="text-neutral-700">Choose an iTerm Color theme</div>
	</div>
	<div class="bg-neutral-100 p-4 rounded-2xl">
		<input type="file" bind:files on:change={() => readFile(files[0])} />
	</div>
</div>

<div class="flex flex-col gap-y-2 mt-8">
	<div class="flex gap-x-2 text-[16px]">
		<div class="font-semibold">Step 2.</div>
		<div class="text-neutral-700">Copy the output to your Ghostty config</div>
	</div>
	<div class="bg-neutral-100 p-4 rounded-2xl flex font-mono text-[13px] leading-5">
		<textarea rows="16" readonly class="flex-auto bg-transparent">{output}</textarea>
	</div>
</div>

<div class="flex gap-2 mt-4 mb-10">
	{#each ghosttyOutputColors as color}
		<sl-tooltip content={`${color}`}>
			<div
				class="w-4 h-4 border flex-shrink-0 border-black border-opacity-5 rounded-full"
				style={`background-color: #${color.split('#')[1]}`}
			/>
		</sl-tooltip>
	{/each}
</div>
