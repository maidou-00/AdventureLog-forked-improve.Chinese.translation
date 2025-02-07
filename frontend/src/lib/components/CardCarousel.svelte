<script lang="ts">
	import type { Adventure } from '$lib/types';
	import ImageDisplayModal from './ImageDisplayModal.svelte';
	import { t } from 'svelte-i18n';
	import * as Carousel from '$lib/components/ui/carousel/index.js';

	export let adventures: Adventure[] = [];

	// When an image is clicked, store its data for the modal.
	let selectedImage: { image: string; adventure: Adventure; is_primary: boolean } | null = null;

	// Flatten and map adventures to get all images along with their adventure data.
	$: adventure_images = adventures.flatMap((adventure) =>
		adventure.images.map((image) => ({
			image: image.image,
			adventure,
			is_primary: image.is_primary
		}))
	);

	// Sort so that primary images appear first.
	$: {
		adventure_images.sort((a, b) => {
			if (a.is_primary && !b.is_primary) return -1;
			else if (!a.is_primary && b.is_primary) return 1;
			else return 0;
		});
	}
</script>

{#if selectedImage}
	<ImageDisplayModal
		adventure={selectedImage.adventure}
		image={selectedImage.image}
		on:close={() => (selectedImage = null)}
	/>
{/if}

<figure>
	{#if adventure_images && adventure_images.length > 0}
		<Carousel.Root>
			<Carousel.Content>
				{#each adventure_images as imageData, index (index)}
					<Carousel.Item>
						<a on:click={() => (selectedImage = imageData)} class="cursor-pointer">
							<img
								src={imageData.image}
								alt={imageData.adventure.name}
								class="w-full h-48 object-cover"
							/>
						</a>
					</Carousel.Item>
				{/each}
			</Carousel.Content>
			<Carousel.Previous />
			<Carousel.Next />
		</Carousel.Root>
	{:else}
		<!-- Fallback figure with a gradient if no images are found -->
		<div class="w-full h-48 bg-gradient-to-r from-success via-base to-primary relative">
			<div
				class="absolute bottom-0 left-0 px-2 py-1 text-md font-medium bg-neutral rounded-tr-lg shadow-md"
			>
				{$t('adventures.no_image_found')}
			</div>
		</div>
	{/if}
</figure>
