<script>
	// @ts-nocheck
	// @ts-ignore
	import plist from 'plist/dist/plist.js';

	let input = '';
	let output = '';
	let ghosttyOutput = '';
	let files = [];

	$: {
		try {
			output = plist.parse(input);
			parseOutputToGhosttyConfig();
			console.log(output);
		} catch (e) {
			console.log(e);
			console.log('An error occurred while parsing the plist');
		}
	}

	function parseOutputToGhosttyConfig() {
		// Find all keys beginning with "Ansi"
		const ansiKeys = Object.keys(output).filter((key) => key.startsWith('Ansi'));

		// For each color keys
		ansiKeys.forEach((key) => {
			// Get the color value
			const color = parseItermColorToHexValue(output[key]);
			// Get the color name
			const colorName = key.replace('Ansi ', '').replace(' Color', '');
			// Add the color to the ghostty config
			ghosttyOutput += `palette=${colorName}=${color}\n`;
		});

    
	}

	function parseItermColorToHexValue(input) {
		let r, g, b;

		r = Math.trunc(input['Red Component'] * 255);
		g = Math.trunc(input['Green Component'] * 255);
		b = Math.trunc(input['Blue Component'] * 255);

		return rgbToHex(r, g, b);
	}

	function rgbToHex(r, g, b) {
		return '#' + componentToHex(r) + componentToHex(g) + componentToHex(b);
	}

	function componentToHex(c) {
		var hex = c.toString(16);
		return hex.length == 1 ? '0' + hex : hex;
	}

	function readFile(file) {
		const reader = new FileReader();
		reader.onload = (e) => {
			input = e.target.result;
		};
		reader.readAsText(file);
	}
</script>

<div class="text-[28px] font-bold tracking-tight mt-10 mb-10">
  iTerm Theme → Ghostty Config Converter
</div>

<div class="flex flex-col gap-y-2 mt-8">
  <div class="flex gap-x-2 text-[16px]">
    <div class="font-semibold">Step 1.</div>
    <div class="text-neutral-700">Import an iTerm Color theme</div>
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
  <div class="bg-neutral-100 p-4 rounded-2xl flex font-mono text-[13px]">
    <textarea readonly class="flex-auto h-[200px] bg-transparent">{ghosttyOutput}</textarea>
  </div>
</div>