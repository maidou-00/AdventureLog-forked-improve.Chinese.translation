<script lang="ts">
	import { goto } from '$app/navigation';
	import { t } from 'svelte-i18n';
	import { fade, fly } from 'svelte/transition';
	import { Button } from '$lib/components/ui/button';
	import { Badge } from '$lib/components/ui/badge';
	import { CompassIcon, MapPinIcon, CalendarIcon, UsersIcon } from 'lucide-svelte';
	import AdventureOverlook from '$lib/assets/AdventureOverlook.webp';
	export let data;
	import {
		Card,
		CardContent,
		CardDescription,
		CardHeader,
		CardTitle
	} from '$lib/components/ui/card';
	import { MapIcon, StarIcon } from 'lucide-svelte';

	const features = [
		{
			icon: CompassIcon,
			title: 'Smart Route Planning',
			description: 'AI-powered itinerary creation based on your preferences, time, and budget.'
		},
		{
			icon: MapIcon,
			title: 'Hidden Gems',
			description:
				'Discover local secrets and off-the-beaten-path destinations for unique experiences.'
		},
		{
			icon: UsersIcon,
			title: 'Travel Community',
			description: 'Connect with fellow adventurers, share tips, and join group expeditions.'
		},
		{
			icon: StarIcon,
			title: 'Personalized Experience',
			description: 'Get tailored recommendations based on your travel style and interests.'
		}
	];
</script>

<section id="hero" class="container relative">
	<div class="flex flex-col items-center gap-4 py-20 text-center lg:py-32">
		<div in:fly={{ y: 20, duration: 500 }} class="space-y-4">
			<Badge class="mb-4">✨ Your Next Adventure Awaits</Badge>
			<h1 class="text-4xl font-bold tracking-tighter sm:text-5xl md:text-6xl lg:text-7xl">
				Plan Your Dream Adventure <br />
				With Confidence
			</h1>
			<p class="mx-auto max-w-[700px] text-muted-foreground md:text-xl">
				Discover and plan unforgettable trips with our AI-powered travel planner. Create custom
				itineraries, find hidden gems, and connect with fellow adventurers.
			</p>
		</div>

		<div in:fade={{ delay: 200, duration: 500 }} class="flex flex-col gap-4 min-[400px]:flex-row">
			<!-- <Button size="lg">Start Planning Free</Button>
			<Button size="lg" variant="outline">View Adventures</Button> -->
			<Button size="lg" on:click={() => goto('/login')}>Login</Button>
			<Button size="lg" variant="outline" on:click={() => goto('/signup')}>Signup</Button>
		</div>

		<div in:fade={{ delay: 400, duration: 500 }} class="mt-8">
			<Card class="border-muted/50">
				<CardContent class="flex gap-4 p-4 sm:gap-8">
					<div class="flex items-center gap-2">
						<UsersIcon class="h-4 w-4 text-muted-foreground" />
						<span class="text-sm">50K+ Travelers</span>
					</div>
					<div class="flex items-center gap-2">
						<MapPinIcon class="h-4 w-4 text-muted-foreground" />
						<span class="text-sm">1000+ Destinations</span>
					</div>
					<div class="hidden items-center gap-2 sm:flex">
						<CalendarIcon class="h-4 w-4 text-muted-foreground" />
						<span class="text-sm">24/7 Support</span>
					</div>
				</CardContent>
			</Card>
		</div>
	</div>

	<div
		in:fade={{ delay: 600, duration: 500 }}
		class="grid gap-4 py-8 md:grid-cols-2 lg:grid-cols-4"
	>
		{#each features as feature}
			<Card>
				<CardHeader>
					<div
						class="mb-2 inline-flex h-10 w-10 items-center justify-center rounded-lg bg-primary/10"
					>
						<svelte:component this={feature.icon} class="h-5 w-5 text-primary" />
					</div>
					<CardTitle class="text-xl">{feature.title}</CardTitle>
				</CardHeader>
				<CardContent>
					<CardDescription>{feature.description}</CardDescription>
				</CardContent>
			</Card>
		{/each}
	</div>
</section>

<svelte:head>
	<title>Home | AdventureLog</title>
	<meta
		name="description"
		content="AdventureLog is a platform to log your adventures and plan your travel."
	/>
</svelte:head>
