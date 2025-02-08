<script lang="ts">
	// External & Library Imports
	import { createEventDispatcher } from 'svelte';
	import { goto } from '$app/navigation';
	import { t } from 'svelte-i18n';
	import { addToast } from '$lib/toasts';

	// UI Components
	import { Card, CardContent, CardHeader, CardFooter } from '$lib/components/ui/card';
	import { Badge } from '$lib/components/ui/badge';
	import { Button } from '$lib/components/ui/button';
	import * as DropdownMenu from '$lib/components/ui/dropdown-menu/index.js';
	import CardCarousel from './CardCarousel.svelte';
	import CollectionLink from './CollectionLink.svelte';
	import DeleteWarning from './DeleteWarning.svelte';

	// Icons from lucide-svelte
	import {
		Ellipsis,
		FilePenLine,
		Rocket,
		MapPinIcon,
		GlobeIcon,
		LockIcon,
		Link2Off,
		Plus,
		Trash,
		Link
	} from 'lucide-svelte';

	// Type Imports
	import type { Adventure, Collection, User } from '$lib/types';

	// Props
	export let readOnly: boolean = false;
	export let link: boolean = false;
	export let user: User | null;
	export let adventure: Adventure;
	export let collection: Collection | null = null;

	// Local State & Event Dispatcher
	const dispatch = createEventDispatcher();
	let isCollectionModalOpen: boolean = false;
	let isWarningModalOpen: boolean = false;
	let activityTypes: string[] = [];
	let unlinked: boolean = false;

	// Update activity types reactively
	$: if (adventure.activity_types) {
		activityTypes = [...adventure.activity_types];
		if (activityTypes.length > 3) {
			const remaining = activityTypes.length - 3;
			activityTypes = activityTypes.slice(0, 3).concat('+' + remaining);
		}
	}

	// Reactive update for unlinked status
	$: if (collection && collection.start_date && collection.end_date) {
		unlinked = adventure.visits.every((visit) => {
			if (!visit.start_date || !visit.end_date) return true;
			const isBeforeVisit = collection.end_date && collection.end_date < visit.start_date;
			const isAfterVisit = collection.start_date && collection.start_date > visit.end_date;
			return isBeforeVisit || isAfterVisit;
		});
	}

	// Helper Functions
	async function deleteAdventure() {
		const res = await fetch(`/api/adventures/${adventure.id}`, { method: 'DELETE' });
		if (res.ok) {
			addToast('info', $t('adventures.adventure_delete_success'));
			dispatch('delete', adventure.id);
		} else {
			console.error('Error deleting adventure');
		}
	}

	async function removeFromCollection() {
		const res = await fetch(`/api/adventures/${adventure.id}`, {
			method: 'PATCH',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({ collection: null })
		});
		if (res.ok) {
			addToast('info', $t('adventures.collection_remove_success'));
			dispatch('delete', adventure.id);
		} else {
			addToast('error', $t('adventures.collection_remove_error'));
		}
	}

	async function linkCollection(event: CustomEvent<number>) {
		const collectionId = event.detail;
		const res = await fetch(`/api/adventures/${adventure.id}`, {
			method: 'PATCH',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({ collection: collectionId })
		});
		if (res.ok) {
			console.log('Adventure linked to collection');
			addToast('info', $t('adventures.collection_link_success'));
			isCollectionModalOpen = false;
			dispatch('delete', adventure.id);
		} else {
			addToast('error', $t('adventures.collection_link_error'));
		}
	}

	function editAdventure() {
		dispatch('edit', adventure);
	}

	function dispatchLink() {
		dispatch('link', adventure);
	}

	// Determine the status badge color based on the adventure's visited status
	const getStatusColor = adventure.is_visited ? 'bg-green-500' : 'bg-blue-500';
</script>

<!-- Modals -->
{#if isCollectionModalOpen}
	<CollectionLink on:link={linkCollection} on:close={() => (isCollectionModalOpen = false)} />
{/if}
{#if isWarningModalOpen}
	<DeleteWarning
		title={$t('adventures.delete_adventure')}
		button_text="Delete"
		description={$t('adventures.adventure_delete_confirm')}
		is_warning={false}
		on:close={() => (isWarningModalOpen = false)}
		on:confirm={deleteAdventure}
	/>
{/if}

<!-- Main Card -->
<Card class="w-full max-w-sm bg-primary-foreground border-none outline-none">
	<!-- Adventure Image & Badge -->
	<div class="relative aspect-video w-full overflow-hidden">
		<CardCarousel adventures={[adventure]} />
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

	<!-- Card Header: Title & Location -->
	<CardHeader class="space-y-4">
		<a
			href={`/adventures/${adventure.id}`}
			class="scroll-m-20 text-xl font-semibold tracking-tight"
		>
			{adventure.name}
		</a>
		<div class="flex items-center text-muted-foreground">
			<MapPinIcon class="mr-1 h-4 w-4" />
			<span class="text-sm">{adventure.location}</span>
		</div>
	</CardHeader>

	<!-- Card Content: Badges -->
	<CardContent>
		<div class="flex flex-wrap gap-2">
			<Badge class={getStatusColor}>
				{adventure.is_visited ? '✓ Visited' : '⏳ Planned'}
			</Badge>
			<Badge variant="outline" class="capitalize">
				{(adventure.category?.icon ?? '') + (adventure.category?.display_name ?? '')}
			</Badge>
		</div>
	</CardContent>

	<!-- Card Footer: Actions -->
	<CardFooter class="flex justify-end overflow-visible">
		{#if !readOnly}
			{#if !link}
				{#if adventure.user_id == user?.uuid || (collection && user && collection.shared_with?.includes(user.uuid))}
					<DropdownMenu.Root>
						<DropdownMenu.Trigger>
							<Button variant="outline" class="p-1 rounded-lg bg-secondary">
								<Ellipsis class="w-8 h-8" />
							</Button>
						</DropdownMenu.Trigger>
						<!-- Added vertical spacing with "space-y-2" -->
						<DropdownMenu.Content class="w-56 space-y-2">
							<DropdownMenu.Item on:click={() => goto(`/adventures/${adventure.id}`)}>
								<Rocket class="w-6 h-6 mr-2" />
								<span>{$t('adventures.open_details')}</span>
							</DropdownMenu.Item>
							<DropdownMenu.Item on:click={editAdventure}>
								<FilePenLine class="w-6 h-6 mr-2" />
								<span>{$t('adventures.edit_adventure')}</span>
							</DropdownMenu.Item>
							{#if adventure.collection && user?.uuid == adventure.user_id}
								<DropdownMenu.Item on:click={removeFromCollection}>
									<Link2Off class="w-6 h-6 mr-2" />
									<span>{$t('adventures.remove_from_collection')}</span>
								</DropdownMenu.Item>
							{/if}
							{#if !adventure.collection}
								<DropdownMenu.Item on:click={() => (isCollectionModalOpen = true)}>
									<Plus class="w-6 h-6 mr-2" />
									<span>{$t('adventures.add_to_collection')}</span>
								</DropdownMenu.Item>
							{/if}
							<DropdownMenu.Item
								id="delete_adventure"
								data-umami-event="Delete Adventure"
								on:click={() => (isWarningModalOpen = true)}
							>
								<Trash class="w-6 h-6 mr-2" />
								<span>{$t('adventures.delete')}</span>
							</DropdownMenu.Item>
						</DropdownMenu.Content>
					</DropdownMenu.Root>
				{:else}
					<Button on:click={() => goto(`/adventures/${adventure.id}`)}>
						<Rocket class="w-6 h-6" />
					</Button>
				{/if}
			{/if}
			{#if link}
				<Button on:click={dispatchLink}>
					<Link class="w-6 h-6" />
				</Button>
			{/if}
		{/if}
	</CardFooter>
</Card>
