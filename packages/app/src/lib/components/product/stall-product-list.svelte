<script lang="ts">
	import type { RichStall } from '$lib/server/stalls.service'
	import { createProductsByFilterQuery } from '$lib/fetch/products.queries'

	import Spinner from '../assets/spinner.svelte'
	import Separator from '../ui/separator/separator.svelte'
	import ImgPlaceHolder from './imgPlaceHolder.svelte'
	import SatPriceLoader from './sat-price-loader.svelte'

	export let stall: Partial<RichStall>

	$: productsByStall = createProductsByFilterQuery({
		stallId: stall.id,
	})
</script>

<div>
	{#if $productsByStall.isLoading}
		<Spinner />
	{:else if $productsByStall?.data?.products.length}
		{#each $productsByStall?.data?.products as product}
			<a class="flex flex-row justify-between my-4 gap-2" href={`/products/${product.id}`}>
				{#if product?.images?.length}
					{@const mainImage = product.images?.sort((a, b) => a.imageOrder - b.imageOrder)}
					<img class="contain h-[60px] aspect-square object-cover" src={mainImage[0].imageUrl} alt="" />
				{:else}
					<ImgPlaceHolder imageType={'mini'} />
				{/if}
				<div class="flex flex-col flex-grow justify-between">
					<div>{product.name}</div>
					<div class="max-w-[300px] whitespace-nowrap overflow-hidden text-ellipsis">{product.description}</div>
				</div>
				<div class="flex flex-col justify-between text-right">
					<SatPriceLoader {product} />
					<div>({product.currency} {product.price})</div>
				</div>
			</a>
			<Separator />
		{/each}
	{/if}
</div>
