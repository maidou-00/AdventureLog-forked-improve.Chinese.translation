<script lang="ts">
	import { Card, CardContent, CardHeader } from '$lib/components/ui/card';
	import { Badge } from '$lib/components/ui/badge';
	import { MapPinIcon, GlobeIcon, LockIcon, User } from 'lucide-svelte';
	import type { Adventure } from '$lib/types';
	import CardCarousel from './CardCarousel.svelte';
	export let adventure: Adventure;

	export let user: User;

	// Get status color based on adventure status
	const getStatusColor = adventure.is_visited ? 'bg-green-500' : 'bg-blue-500';
</script>

<Card class="w-full max-w-sm overflow-hidden bg-gray-800 outline-none border-none">
	<!-- Adventure Image -->
	<div class="relative aspect-video w-full overflow-hidden">
		<CardCarousel adventures={[adventure]} />
		<!-- Public/Private Badge -->
		<div class="absolute right-2 top-2">
			<Badge variant="secondary" class="shadow-lg">
				{#if adventure.is_public}
					<GlobeIcon class="mr-1 h-3 w-3" />
					Public
				{:else}
					<LockIcon class="mr-1 h-3 w-3" />
					Private
				{/if}
			</Badge>
		</div>
	</div>

	<CardHeader class="space-y-4">
		<!-- Title -->
		<a href={`/adventures/${adventure.id}`} class="text-xl font-semibold">{adventure.name}</a>

		<!-- Location -->
		<div class="flex items-center text-muted-foreground">
			<MapPinIcon class="mr-1 h-4 w-4" />
			<span class="text-sm">{adventure.location}</span>
		</div>
	</CardHeader>

	<CardContent>
		<div class="flex flex-wrap gap-2">
			<!-- Status Badge -->
			<Badge class={getStatusColor}>
				{adventure.is_visited ? '✓ Visited' : '⏳ Planned'}
			</Badge>

			<!-- Category Badge -->
			<Badge variant="outline" class="capitalize">
				{(adventure.category?.icon ?? '') + (adventure.category?.display_name ?? '')}
			</Badge>
		</div>
	</CardContent>
</Card>
