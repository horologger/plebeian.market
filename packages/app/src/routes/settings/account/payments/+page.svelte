<script lang="ts">
	import { goto } from '$app/navigation'
	import { page } from '$app/stores'
	import PaymentDetailEdit from '$lib/components/settings/paymentDetailCreateEdit.svelte'
	import Button from '$lib/components/ui/button/button.svelte'
	import { privatePaymentsQuery } from '$lib/fetch/payments.queries'

	import type { PageData } from './$types'

	export let data: PageData

	const linkDetails = data.menuItems
		.find((item) => item.value === 'account-settings')
		?.links.find((item) => item.href === $page.url.pathname)
</script>

<div class="flex flex-col gap-4">
	<div>
		<div class=" flex items-center gap-1">
			<Button variant="ghost" size="icon" on:click={() => goto('/settings/account/')}>
				<span class="cursor-pointer i-tdesign-arrow-left w-6 h-6" />
			</Button>
			<section>
				<h3 class="text-lg font-bold">{linkDetails?.title}</h3>
				<p class="text-gray-600">{linkDetails?.description}</p>
			</section>
		</div>
	</div>
	{#if $privatePaymentsQuery.isLoading}
		<p>Loading...</p>
	{:else if $privatePaymentsQuery.data?.length}
		{#each $privatePaymentsQuery.data ?? [] as paymentDetail (paymentDetail.id)}
			<PaymentDetailEdit {paymentDetail} />
		{/each}
		<PaymentDetailEdit />
	{:else if !$privatePaymentsQuery.data?.length}
		<PaymentDetailEdit showGuidance={true} />
	{/if}
</div>
