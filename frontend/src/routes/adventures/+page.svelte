<script lang="ts">
	// Svelte & App Utilities
	import { enhance, deserialize } from '$app/forms';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import { t } from 'svelte-i18n';
	import type { Adventure, Category } from '$lib/types';

	// Custom Components
	import AdventureCard from '$lib/components/AdventureCard.svelte';
	import AdventureModal from '$lib/components/AdventureModal.svelte';
	import CategoryFilterDropdown from '$lib/components/CategoryFilterDropdown.svelte';
	import CategoryModal from '$lib/components/CategoryModal.svelte';
	import NotFound from '$lib/components/NotFound.svelte';

	// Icons
	import Plus from '~icons/mdi/plus';

	// shadcn UI Components
	import { Card, CardContent, CardHeader, CardFooter } from '$lib/components/ui/card';
	import { Button } from '$lib/components/ui/button';
	import * as DropdownMenu from '$lib/components/ui/dropdown-menu';
	import { Sheet, SheetContent } from '$lib/components/ui/sheet';

	// Data & Pagination Setup
	export let data: any;
	console.log(data);
	let adventures: Adventure[] = data.props.adventures || [];
	let count = data.props.count || 0;
	let resultsPerPage: number = 25;
	let totalPages = Math.ceil(count / resultsPerPage);
	let currentPage: number = 1;

	// Sort and Filter State
	let currentSort = {
		order_by: '',
		order: '',
		visited: true,
		planned: true,
		includeCollections: true,
		is_visited: 'all'
	};
	let typeString: string = '';

	// Modal & Sidebar State
	let is_category_modal_open: boolean = false;
	let isAdventureModalOpen: boolean = false;
	let adventureToEdit: Adventure | null = null;
	let sidebarOpen: any = false;

	// Reactive: Update URL when typeString changes
	$: {
		if (typeof window !== 'undefined') {
			const url = new URL(window.location.href);
			if (typeString) {
				url.searchParams.set('types', typeString);
			} else {
				url.searchParams.delete('types');
			}
			goto(url.toString(), { invalidateAll: true, replaceState: true });
		}
	}

	import { Filter } from 'lucide-svelte';

	// Reactive: Read typeString from URL on client
	$: {
		if (typeof window !== 'undefined') {
			const url = new URL(window.location.href);
			const types = url.searchParams.get('types');
			if (types) typeString = types;
		}
	}

	// Handle page changes (pagination)
	function handleChangePage(pageNumber: number) {
		currentPage = pageNumber;
		const url = new URL(window.location.href);
		url.searchParams.set('page', pageNumber.toString());
		adventures = data.props.adventures;
		goto(url.toString(), { invalidateAll: true, replaceState: true });
	}

	// Update currentPage from URL if present
	$: {
		const url = new URL($page.url);
		const pageParam = url.searchParams.get('page');
		if (pageParam) currentPage = parseInt(pageParam);
	}

	// Update adventures and count when data changes
	$: {
		if (data.props.adventures) adventures = data.props.adventures;
		if (data.props.count) {
			count = data.props.count;
			totalPages = Math.ceil(count / resultsPerPage);
		}
	}

	// Update sorting based on URL parameters
	$: {
		const url = new URL($page.url);
		currentSort.order_by = url.searchParams.get('order_by') || 'updated_at';
		currentSort.order = url.searchParams.get('order_direction') || 'asc';
		currentSort.planned = url.searchParams.get('planned') === 'on';
		currentSort.visited = url.searchParams.get('visited') === 'on';
		currentSort.includeCollections = url.searchParams.get('include_collections') === 'on';
		if (!currentSort.visited && !currentSort.planned) {
			currentSort.visited = true;
			currentSort.planned = true;
		}
		currentSort.is_visited = url.searchParams.get('is_visited') || 'all';
	}

	// Adventure CRUD functions
	function deleteAdventure(event: CustomEvent<string>) {
		adventures = adventures.filter((adv) => adv.id !== event.detail);
	}
	function saveOrCreate(event: CustomEvent<Adventure>) {
		if (adventures.find((adv) => adv.id === event.detail.id)) {
			adventures = adventures.map((adv) => (adv.id === event.detail.id ? event.detail : adv));
		} else {
			adventures = [event.detail, ...adventures];
		}
		isAdventureModalOpen = false;
	}
	function editAdventure(event: CustomEvent<Adventure>) {
		adventureToEdit = event.detail;
		isAdventureModalOpen = true;
	}

	// Sidebar toggle
	function toggleSidebar() {
		sidebarOpen = !sidebarOpen;
	}
</script>

{#if isAdventureModalOpen}
	<AdventureModal
		{adventureToEdit}
		on:close={() => (isAdventureModalOpen = false)}
		on:save={saveOrCreate}
	/>
{/if}

{#if is_category_modal_open}
	<CategoryModal on:close={() => (is_category_modal_open = false)} />
{/if}

<!-- Sidebar as a Sheet Component -->
<Sheet open={sidebarOpen} on:openChange={(open) => (sidebarOpen = open)}>
	<SheetContent side="left" class="p-6 w-80">
		<form method="get" class="space-y-6">
			<CategoryFilterDropdown bind:types={typeString} />
			<Button variant="outline" on:click={() => (is_category_modal_open = true)} class="w-full">
				{$t('adventures.manage_categories')}
			</Button>
			<hr class="border-t my-4" />
			<div class="space-y-4">
				<h3 class="text-xl font-bold text-center">{$t('adventures.sort')}</h3>
				<!-- Order Direction -->
				<div>
					<p class="text-lg font-semibold mb-2">{$t('adventures.order_direction')}</p>
					<div class="flex space-x-4">
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="order_direction"
								value="asc"
								checked={currentSort.order === 'asc'}
								class="form-radio"
							/>
							<span>{$t('adventures.ascending')}</span>
						</label>
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="order_direction"
								value="desc"
								checked={currentSort.order === 'desc'}
								class="form-radio"
							/>
							<span>{$t('adventures.descending')}</span>
						</label>
					</div>
				</div>
				<!-- Order By -->
				<div>
					<p class="text-lg font-semibold mb-2">{$t('adventures.order_by')}</p>
					<div class="flex flex-wrap gap-4">
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="order_by"
								value="updated_at"
								checked={currentSort.order_by === 'updated_at'}
								class="form-radio"
							/>
							<span>{$t('adventures.updated')}</span>
						</label>
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="order_by"
								value="name"
								checked={currentSort.order_by === 'name'}
								class="form-radio"
							/>
							<span>{$t('adventures.name')}</span>
						</label>
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="order_by"
								value="date"
								checked={currentSort.order_by === 'date'}
								class="form-radio"
							/>
							<span>{$t('adventures.date')}</span>
						</label>
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="order_by"
								value="rating"
								checked={currentSort.order_by === 'rating'}
								class="form-radio"
							/>
							<span>{$t('adventures.rating')}</span>
						</label>
					</div>
				</div>
				<!-- Visited Filter -->
				<div>
					<p class="text-lg font-semibold mb-2">{$t('adventures.visited')}</p>
					<div class="flex space-x-4">
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="is_visited"
								value="all"
								checked={currentSort.is_visited === 'all'}
								class="form-radio"
							/>
							<span>{$t('adventures.all')}</span>
						</label>
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="is_visited"
								value="true"
								checked={currentSort.is_visited === 'true'}
								class="form-radio"
							/>
							<span>{$t('adventures.visited')}</span>
						</label>
						<label class="flex items-center space-x-1">
							<input
								type="radio"
								name="is_visited"
								value="false"
								checked={currentSort.is_visited === 'false'}
								class="form-radio"
							/>
							<span>{$t('adventures.not_visited')}</span>
						</label>
					</div>
				</div>
				<!-- Include Collections -->
				<div class="space-y-2">
					<p class="text-lg font-semibold">{$t('adventures.sources')}</p>
					<label class="flex items-center space-x-2">
						<input
							type="checkbox"
							name="include_collections"
							id="include_collections"
							checked={currentSort.includeCollections}
							class="form-checkbox"
						/>
						<span>{$t('adventures.collection_adventures')}</span>
					</label>
				</div>
			</div>
			<Button type="submit" class="w-full">
				{$t('adventures.filter')}
			</Button>
		</form>
	</SheetContent>
</Sheet>

<!-- Main Content Area -->
<div class="flex-1 p-6 pb-24">
	<h1 class="text-center text-4xl font-extrabold tracking-tight">
		{$t('navbar.my_adventures')}
	</h1>
	<p class="text-center text-xl font-semibold mt-2">
		{count}
		{$t('adventures.count_txt')}
	</p>
	{#if adventures.length === 0}
		<NotFound error={undefined} />
	{/if}
	<div class="mt-6 grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6 auto-rows-fr">
		{#each adventures as adventure}
			<AdventureCard
				user={data.user}
				{adventure}
				on:delete={deleteAdventure}
				on:edit={editAdventure}
			/>
		{/each}
	</div>
	<!-- Pagination -->
	{#if totalPages > 1}
		<div class="mt-6 flex items-center justify-center space-x-2">
			{#each Array.from({ length: totalPages }, (_, i) => i + 1) as page}
				{#if currentPage !== page}
					<Button variant="outline" on:click={() => handleChangePage(page)}>
						{page}
					</Button>
				{:else}
					<Button variant="default">
						{page}
					</Button>
				{/if}
			{/each}
		</div>
	{/if}
</div>

<!-- Bottom Actions Bar -->
<div class="fixed inset-x-0 bottom-0 flex justify-end items-center p-4 space-x-4">
	<Button on:click={toggleSidebar}>
		<Filter class="w-6 h-6" />
	</Button>
	<Button
		on:click={() => {
			isAdventureModalOpen = true;
			adventureToEdit = null;
		}}
	>
		<Plus class="w-6 h-6" />
	</Button>
</div>

<svelte:head>
	<title>{$t('navbar.adventures')}</title>
	<meta name="description" content="View your completed and planned adventures." />
</svelte:head>
