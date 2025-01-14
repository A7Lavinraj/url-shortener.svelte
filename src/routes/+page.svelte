<script lang="ts">
	import Spinner from '$components/spinner.svelte';
	import _config from '../_config';

	let isLoading: boolean = false;
	let shortenedURLStack: { short_name: string; short_url: string }[] = [];
	let selectedIndex = 0;
	let timeoutId: NodeJS.Timeout | null;
	let copyStatus: boolean = false;

	async function submitHandler(
		event: SubmitEvent & {
			currentTarget: EventTarget & HTMLFormElement;
		}
	) {
		event.preventDefault();
		const form = event.currentTarget;
		const entries = Object.fromEntries(new FormData(form));
		isLoading = true;

		try {
			const response = await fetch(_config.BASE_URL, {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(entries)
			});

			const parsed = await response.json();
			shortenedURLStack = [
				{
					short_name: entries['shorten-name'] as string,
					short_url: `${_config.BASE_URL}/${parsed.short_url}`
				},
				...shortenedURLStack
			];
		} catch (err) {
			console.error(err);
		}

		isLoading = false;
	}

	function copyURLToClipboard() {
		if (timeoutId) {
			clearTimeout(timeoutId);
		}

		navigator.clipboard.writeText(shortenedURLStack[selectedIndex].short_url);
		copyStatus = true;
		timeoutId = setTimeout(() => {
			copyStatus = false;
		}, 2000);
	}
</script>

<main class="grid h-screen grid-cols-2 grid-rows-2 gap-2 p-4">
	<aside class="row-span-2 overflow-scroll rounded border border-neutral-600 bg-neutral-800/30 p-2">
		<h2 class="w-full text-center font-semibold">Recents</h2>
		<div class="mt-4">
			<ul class="flex flex-col gap-2">
				{#each shortenedURLStack as url, index}
					{#if index === selectedIndex}
						<li>
							<button
								type="button"
								onclick={() => (selectedIndex = index)}
								class="h-10 w-full rounded border border-blue-400 bg-blue-900/30 px-4 text-left"
							>
								{url.short_name}
							</button>
						</li>
					{:else}
						<li>
							<button
								type="button"
								onclick={() => (selectedIndex = index)}
								class="h-10 w-full rounded border-neutral-400 px-4 text-left text-neutral-600 hover:border hover:bg-neutral-800/30"
							>
								{url.short_name}
							</button>
						</li>
					{/if}
				{/each}
			</ul>
		</div>
	</aside>
	<form
		onsubmit={submitHandler}
		class="flex h-full w-full flex-col gap-4 overflow-scroll rounded border border-neutral-600 bg-neutral-800/30 p-4"
	>
		<div class="flex flex-col gap-4">
			<label for="shorten-name">Shorten name</label>
			<input
				type="text"
				name="shorten-name"
				id="shorten-name"
				class="h-10 rounded bg-neutral-700 px-2"
			/>
		</div>

		<div class="flex flex-col gap-4">
			<label for="original-url">URL to shorten</label>
			<input
				type="url"
				name="original-url"
				id="original-url"
				required
				class="h-10 rounded bg-neutral-700 px-2"
			/>
		</div>

		<button
			type="submit"
			class="flex min-h-10 items-center justify-center gap-4 rounded bg-indigo-600"
		>
			{#if isLoading}
				<Spinner />
			{:else}
				<span>Submit</span>
			{/if}
		</button>
	</form>
	<div
		class="flex h-full flex-1 flex-col items-center overflow-scroll border border-neutral-600 bg-neutral-800/30 pt-4 text-xl font-semibold"
	>
		{#if shortenedURLStack.length === 0}
			<h2>Your shortened result show here&excl;</h2>
		{:else}
			<h2>{shortenedURLStack[selectedIndex].short_name}</h2>
			<div
				class="mt-4 flex items-center gap-4 rounded border border-neutral-600 bg-neutral-700/30 p-4"
			>
				<a
					target="_blank"
					href={shortenedURLStack[selectedIndex].short_url}
					class="font-mono hover:text-blue-400"
				>
					{shortenedURLStack[selectedIndex].short_url}
				</a>
				<button type="button" aria-label="copy-button" class="size-6" onclick={copyURLToClipboard}>
					{#if copyStatus}
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" fill="#00ff00">
							<path
								d="M438.6 105.4c12.5 12.5 12.5 32.8 0 45.3l-256 256c-12.5 12.5-32.8 12.5-45.3 0l-128-128c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0L160 338.7 393.4 105.4c12.5-12.5 32.8-12.5 45.3 0z"
							/></svg
						>
					{:else}
						<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" fill="#525252">
							<path
								d="M208 0L332.1 0c12.7 0 24.9 5.1 33.9 14.1l67.9 67.9c9 9 14.1 21.2 14.1 33.9L448 336c0 26.5-21.5 48-48 48l-192 0c-26.5 0-48-21.5-48-48l0-288c0-26.5 21.5-48 48-48zM48 128l80 0 0 64-64 0 0 256 192 0 0-32 64 0 0 48c0 26.5-21.5 48-48 48L48 512c-26.5 0-48-21.5-48-48L0 176c0-26.5 21.5-48 48-48z"
							/>
						</svg>
					{/if}
				</button>
			</div>
		{/if}
	</div>
</main>
