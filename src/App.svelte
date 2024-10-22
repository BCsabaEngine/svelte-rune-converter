<script lang="ts">
	import './app.postcss';

	import { Label, Navbar, NavBrand, Textarea, Toggle } from 'flowbite-svelte';

	const defaultInput = `export let type: InputType = 'text';
export let title: string;
export let maxLength = 0;
export let regexp: RegExp = /.*/;
export let mandatory = false;
export let inProgress = false;

export let value: string | null;
export let validity: ValidityItem | undefined = { isError: false };
`;

	const pattern = /^export\s+let\s+([^\s:]+)\s*(\s*:\s*([^=]+))?(\s*=\s*(.+))?\s*$/;
	const getType = (value: string): string => {
		value = value.trim().toLowerCase();
		if (
			(value.startsWith('"') && value.endsWith('"')) ||
			(value.startsWith("'") && value.endsWith("'"))
		)
			return 'string';
		if (value === 'false' || value === 'true') return 'boolean';
		if (!Number.isNaN(value)) return 'number';
		return 'unknown';
	};
	const converterFunction = (): string => {
		const matches = input
			.split(';')
			.map((element) => element.trim())
			.filter(Boolean)
			.map((element) => element.match(pattern))
			.filter(Boolean);

		const names = matches.map((p) => p![1] + (p![4] ?? ''));
		const types = matches.map((p) => p![1] + (p![4] ? '?' : '') + (p![2] ?? `: ${getType(p![5])}`));
		if (useChildren) {
			names.push('children');
			types.push('children: unknown');
		}
		return `let {\n  ${names.join(',\n  ')}\n} = $props<{\n  ${types.join(';\n  ')};\n}>();`;
	};

	let input = $state(defaultInput);
	let useChildren = $state(false);
	let output = $derived.by(converterFunction);
</script>

<Navbar>
	<NavBrand href="/">
		<img src="banner.png" class="me-3 h-6 sm:h-9" alt="Svelte Rune Converter" />
		<span class="self-center whitespace-nowrap text-xl font-semibold dark:text-white"
			>Rune Converter</span
		>
	</NavBrand>
	<a href="https://github.com/BCsabaEngine/svelte-rune-converter" target="_blank"
		>This is an open source project<img
			class="inline-flex ml-2"
			width="24"
			src="https://github.githubassets.com/favicons/favicon.png"
			alt="github"
		/></a
	>
</Navbar>

<h1 class="text-2xl text-center mb-8">
	This tool is suitable for converting the properties of a Svelte4 component to the new Svelte5 rune
	type syntax.
</h1>
<div class="grid grid-cols-2 gap-4 m-4 h-[65vh]">
	<div>
		<div class="flex gap-6">
			<Label class="mb-2">Svelte 4 component props (paste code here, use semicolons)</Label>
			<Toggle class="-mt-2" bind:checked={useChildren}>Include children (slot)</Toggle>
		</div>
		<Textarea class="h-full font-mono overflow-auto" bind:value={input} />
	</div>
	<div>
		<Label class="mb-2">Svelte 5 $props rune (copy result from here and use a prettier)</Label>
		<Textarea class="h-full font-mono overflow-auto bg-slate-100" readonly value={output} />
	</div>
</div>
