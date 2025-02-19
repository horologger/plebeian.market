<script lang="ts">
	import { goto, invalidateAll } from '$app/navigation'
	import { page } from '$app/stores'
	import { Button } from '$lib/components/ui/button'
	import * as Collapsible from '$lib/components/ui/collapsible'
	import { Input } from '$lib/components/ui/input'
	import { addForbiddenWordMutation, deleteForbiddenWordMutation } from '$lib/fetch/settingsMeta.mutations.js'
	import { appSettingsMetaQuery } from '$lib/fetch/settingsMeta.queries.js'
	import { toast } from 'svelte-sonner'

	let newWord = ''
	export let data
	const linkDetails = data.menuItems.find((item) => item.value === 'app-settings')?.links.find((item) => item.href === $page.url.pathname)

	let isCollapsibleOpen = false

	$: blacklistedWordsQuery = appSettingsMetaQuery('word_blacklist')

	async function handleDeleteWord(wordId: string) {
		await $deleteForbiddenWordMutation.mutateAsync(wordId)
		toast.success('Word removed from blacklist successfully!')
	}

	async function handleAddWord() {
		if (newWord.trim()) {
			try {
				await $addForbiddenWordMutation.mutateAsync(newWord.trim())

				toast.success('Word added to blacklist successfully!')
				newWord = ''
				await invalidateAll()
				isCollapsibleOpen = false
			} catch (error) {
				console.error('Failed to add word to blacklist', error)
				toast.error('Failed to add word to blacklist')
			}
		}
	}
</script>

<div class="pb-4 space-y-2 max-w-2xl">
	<div>
		<div class=" flex items-center gap-1">
			<Button variant="ghost" size="icon" on:click={() => goto('/settings/app')}>
				<span class="cursor-pointer i-tdesign-arrow-left w-6 h-6" />
			</Button>
			<section>
				<h3 class="text-lg font-bold">{linkDetails?.title}</h3>
				<p class="text-gray-600">{linkDetails?.description}</p>
			</section>
		</div>
	</div>
	{#if $blacklistedWordsQuery?.data?.forbiddenWords}
		{#each $blacklistedWordsQuery?.data?.forbiddenWords as word}
			<div class="flex flex-row items-center justify-between border rounded-md p-2">
				<span class="text-sm">{word.valueText}</span>
				<Button type="button" size="icon" variant="destructive" on:click={() => handleDeleteWord(word.id)}>
					<span class="i-mdi-trash-can" />
				</Button>
			</div>
		{/each}
	{/if}

	<Collapsible.Root bind:open={isCollapsibleOpen}>
		<Collapsible.Trigger asChild let:builder>
			<Button builders={[builder]} variant="outline" class="w-full justify-between">
				Add new blacklisted word
				<span class="i-mdi-plus w-6 h-6" />
			</Button>
		</Collapsible.Trigger>
		<Collapsible.Content>
			<div class="mt-4 space-y-4">
				<Input bind:value={newWord} placeholder="Enter new blacklisted word" />
				<div class="flex justify-end space-x-2">
					<Button variant="outline" on:click={() => (newWord = '')}>Cancel</Button>
					<Button variant="primary" on:click={handleAddWord}>Save</Button>
				</div>
			</div>
		</Collapsible.Content>
	</Collapsible.Root>
</div>
