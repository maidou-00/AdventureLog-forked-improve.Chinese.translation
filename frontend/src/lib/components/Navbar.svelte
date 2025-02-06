<script lang="ts">
	// Import Svelte transitions (if you need them)
	import { fade, slide } from 'svelte/transition';
	// Import lucide icons – see the docs for available props (size, color, etc.)

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
	// Import your UI components
	import { Button } from '$lib/components/ui/button';
	import { Input } from '$lib/components/ui/input';
	import Toggle from './ui/theme/toggle.svelte';
	import { Select, SelectContent, SelectItem, SelectTrigger } from '$lib/components/ui/select';
	import Avatar from './Avatar.svelte';
	// import ThemeToggle from "$project/ThemeToggle.svelte";

	let isShowingAboutModal: boolean = false;

	// Navigation items – note that each icon is a component (see lucide‑svelte docs)
	const navItems = [
		{ href: '/dashboard', icon: HouseIcon, label: 'Home', auth: true },
		{ href: '/adventures', icon: MapPinIcon, label: 'Adventures', auth: true },
		{ href: '/collections', icon: CalendarIcon, label: 'Calendar', auth: true },
		{ href: '/calendar', icon: BookImageIcon, label: 'Collections', auth: true },
		{ href: '/users', icon: SearchIcon, label: 'Users', auth: true },
		{ href: '/settings', icon: SettingsIcon, label: 'Settings', auth: true }
	];

	// Instead of using stores (and the duplicate assignment), we use simple booleans/strings.
	let isMobileMenuOpen = false;
	let currentLang = 'en';

	// Languages available
	const languages = [
		{ value: 'en', label: 'English' },
		{ value: 'es', label: 'Español' },
		{ value: 'fr', label: 'Français' }
	];
</script>

<header
	class="sticky top-0 z-50 w-full bg-background/95 backdrop-blur supports-[backdrop-filter]:bg-background/60"
>
	<nav class="container flex h-16 items-center">
		<!-- Logo -->
		<a href="/" class="mr-6 flex items-center space-x-2">
			<img src="/favicon.png" alt="Logo" class="h-6 w-auto" />
			<span class="text-lg font-semibold">AdventureLog</span>
		</a>

		<!-- Desktop Navigation -->
		<div class="hidden md:flex md:flex-1 mr-4">
			<div class="flex items-center gap-6">
				{#each navItems as item}
					{#if item.auth && data.user}
						{@const Icon = item.icon}
						<a
							href={item.href}
							class="flex items-center gap-2 text-sm font-medium text-muted-foreground transition-colors hover:text-primary"
						>
							<Icon class="w-4 h-4" />
							{item.label}
						</a>
					{/if}
				{/each}
			</div>
		</div>

		<!-- Search Bar -->
		<div class="hidden md:flex md:flex-1 md:items-center md:justify-end md:gap-4">
			<div class="w-full max-w-xs">
				<Input type="search" placeholder="Search..." class="h-9" />
			</div>

			<!-- Language Selector -->
			<!-- <Select bind:value={currentLang}>
				<SelectTrigger class="w-[100px]">
					{currentLang.toUpperCase()}
				</SelectTrigger>
				<SelectContent>
					{#each languages as lang}
						<SelectItem value={lang.value}>{lang.label}</SelectItem>
					{/each}
				</SelectContent>
			</Select> -->

			<Button variant="outline">About</Button>
			<Toggle />
			<Button variant="ghost" class="ml-2 w-9 h-9">
				<SearchIcon class="w-4 h-4" />
			</Button>
			{#if data.user}
				<Avatar user={data.user} />
			{/if}
		</div>

		<!-- Mobile Menu Button -->
		<Button
			variant="ghost"
			class="ml-2 w-9 h-9 md:hidden"
			on:click={() => (isMobileMenuOpen = !isMobileMenuOpen)}
		>
			{#if isMobileMenuOpen}
				<XIcon class="w-4 h-4" />
			{:else}
				<MenuIcon class="w-4 h-4" />
			{/if}
			<span class="sr-only">Toggle menu</span>
		</Button>
	</nav>

	<!-- Mobile Menu -->
	{#if isMobileMenuOpen}
		<div class="container md:hidden" transition:slide>
			<div class="flex flex-col gap-4 pb-4">
				<div class="flex flex-col gap-2">
					{#each navItems as item}
						<a
							href={item.href}
							class="flex items-center gap-2 rounded-md p-2 text-sm font-medium text-muted-foreground hover:bg-accent hover:text-accent-foreground"
							on:click={() => (isMobileMenuOpen = false)}
						>
							<item.icon class="w-4 h-4" />
							{item.label}
						</a>
					{/each}
				</div>

				<div class="space-y-2">
					<Input type="search" placeholder="Search..." />

					<!-- <Select type="single" bind:value={currentLang}>
						<SelectTrigger>
							{currentLang.toUpperCase()}
						</SelectTrigger>
						<SelectContent>
							{#each languages as lang}
								<SelectItem value={lang.value}>{lang.label}</SelectItem>
							{/each}
						</SelectContent>
					</Select> -->

					<div class="flex items-center gap-2">
						<!-- Uncomment ThemeToggle if needed -->
						<!-- <ThemeToggle /> -->
						<Button on:click={() => (isShowingAboutModal = true)} variant="outline" class="flex-1"
							>About</Button
						>
					</div>
				</div>
			</div>
		</div>
	{/if}
</header>
