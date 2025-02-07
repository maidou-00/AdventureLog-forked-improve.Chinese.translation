<script lang="ts">
	// Svelte transitions for mobile menu animations
	import { fade, slide } from 'svelte/transition';
	// Import lucide icons – adjust size and color as needed
	export let data: any;
	import {
		HouseIcon,
		MapPinIcon,
		CalendarIcon,
		BookImageIcon,
		MenuIcon,
		SearchIcon,
		XIcon,
		SettingsIcon
	} from 'lucide-svelte';
	// Import UI components from shadcn‑svelte
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import Toggle from './ui/theme/toggle.svelte';
	import { Select, SelectContent, SelectItem, SelectTrigger } from '$lib/components/ui/select';
	import Avatar from './Avatar.svelte';
	import AboutModal from './AboutModal.svelte';

	let isShowingAboutModal: boolean = false;
	let isMobileMenuOpen = false;
	let currentLang = 'en';

	// Navigation items – ensure that the labels and icons are correctly matched
	const navItems = [
		{ href: '/dashboard', icon: HouseIcon, label: 'Home', auth: true },
		{ href: '/adventures', icon: MapPinIcon, label: 'Adventures', auth: true },
		{ href: '/calendar', icon: CalendarIcon, label: 'Calendar', auth: true },
		{ href: '/collections', icon: BookImageIcon, label: 'Collections', auth: true },
		{ href: '/users', icon: SearchIcon, label: 'Users', auth: true },
		{ href: '/settings', icon: SettingsIcon, label: 'Settings', auth: true }
	];

	// Languages available for the language selector
	const languages = [
		{ value: 'en', label: 'English' },
		{ value: 'es', label: 'Español' },
		{ value: 'fr', label: 'Français' }
	];
</script>

{#if isShowingAboutModal}
	<AboutModal on:close={() => (isShowingAboutModal = false)} />
{/if}

<header
	class="sticky top-0 z-50 w-full bg-background/95 backdrop-blur-sm supports-[backdrop-filter]:bg-background/60 shadow-sm"
>
	<nav class="container mx-auto flex h-16 items-center px-4">
		<!-- Logo Section -->
		<div class="flex items-center space-x-2 mr-4">
			<a href="/" class="flex items-center">
				<img src="/favicon.png" alt="Logo" class="h-8 w-auto" />
				<span class="ml-2 text-xl font-semibold">AdventureLog</span>
			</a>
		</div>

		<!-- Desktop Navigation Links -->
		<div class="hidden md:flex flex-1 items-center space-x-6">
			{#each navItems as item}
				{#if item.auth && data.user}
					{@const Icon = item.icon}
					<a
						href={item.href}
						class="flex items-center gap-1 text-sm font-medium text-muted-foreground hover:text-primary transition-colors"
					>
						<Icon class="w-4 h-4" />
						<span>{item.label}</span>
					</a>
				{/if}
			{/each}
		</div>

		<!-- Desktop Actions: Search, Language Selector, About, Toggle & Avatar -->
		<div class="hidden md:flex items-center gap-4">
			<div class="relative">
				<Input type="search" placeholder="Search..." class="h-9 w-48 pr-8" />
				<SearchIcon class="absolute right-2 top-2 w-4 h-4 text-muted-foreground" />
			</div>
			<Select bind:value={currentLang}>
				<SelectTrigger class="w-20">
					{currentLang.toUpperCase()}
				</SelectTrigger>
				<SelectContent>
					{#each languages as lang}
						<SelectItem value={lang.value}>{lang.label}</SelectItem>
					{/each}
				</SelectContent>
			</Select>
			<Button on:click={() => (isShowingAboutModal = true)} variant="outline" class="px-3">
				About
			</Button>
			<Toggle />
			{#if data.user}
				<Avatar user={data.user} />
			{/if}
		</div>

		<!-- Mobile Menu Toggle Button -->
		<Button
			variant="ghost"
			class="ml-2 w-9 h-9 md:hidden"
			on:click={() => (isMobileMenuOpen = !isMobileMenuOpen)}
		>
			{#if isMobileMenuOpen}
				<XIcon class="w-5 h-5" />
			{:else}
				<MenuIcon class="w-5 h-5" />
			{/if}
			<span class="sr-only">Toggle menu</span>
		</Button>
	</nav>

	<!-- Mobile Menu -->
	{#if isMobileMenuOpen}
		<div class="container mx-auto px-4 md:hidden" transition:slide>
			<div class="flex flex-col gap-4 py-4">
				{#each navItems as item}
					<a
						href={item.href}
						class="flex items-center gap-2 rounded-md p-2 text-sm font-medium text-muted-foreground hover:bg-accent hover:text-accent-foreground"
						on:click={() => (isMobileMenuOpen = false)}
					>
						<item.icon class="w-4 h-4" />
						<span>{item.label}</span>
					</a>
				{/each}

				<div class="flex flex-col gap-2">
					<Input type="search" placeholder="Search..." class="w-full" />
					<Select bind:value={currentLang}>
						<SelectTrigger class="w-full">
							{currentLang.toUpperCase()}
						</SelectTrigger>
						<SelectContent>
							{#each languages as lang}
								<SelectItem value={lang.value}>{lang.label}</SelectItem>
							{/each}
						</SelectContent>
					</Select>
					<div class="flex items-center gap-2">
						<Button on:click={() => (isShowingAboutModal = true)} variant="outline" class="flex-1">
							About
						</Button>
						<Toggle />
					</div>
				</div>
			</div>
		</div>
	{/if}
</header>
