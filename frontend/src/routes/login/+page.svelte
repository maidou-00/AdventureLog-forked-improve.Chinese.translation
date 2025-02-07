<script lang="ts">
	import { enhance } from '$app/forms';
	import { t } from 'svelte-i18n';
	import FileImageBox from '~icons/mdi/file-image-box';
	export let data;
	let isImageInfoModalOpen: boolean = false;
	let socialProviders = data.props?.socialProviders ?? [];
	import GitHub from '~icons/mdi/github';
	import OpenIdConnect from '~icons/mdi/openid';
	import { page } from '$app/stores';

	import ImageInfoModal from '$lib/components/ImageInfoModal.svelte';
	import type { Background } from '$lib/types.js';
	let quote: { quote: string; author: string } = data.props?.quote ?? { quote: '', author: '' };
	let background: Background = data.props?.background ?? { url: '/images/placeholder.svg' };

	// Shadcn UI component imports
	import { Button } from '$lib/components/ui/button/index.js';
	import { Input } from '$lib/components/ui/input/index.js';
	import { Label } from '$lib/components/ui/label/index.js';
</script>

{#if isImageInfoModalOpen}
	<ImageInfoModal {background} on:close={() => (isImageInfoModalOpen = false)} />
{/if}

<!-- Side-by-side layout with full viewport height -->
<div class="w-full lg:grid lg:h-screen lg:grid-cols-[1fr_2fr]">
	<!-- Left: Login Form -->
	<div class="flex items-center justify-center py-12">
		<div class="mx-auto grid w-[300px] gap-6">
			<div class="grid gap-2 text-center">
				<!-- AdventureLog Logo -->
				<img src="/favicon.png" alt="AdventureLog Logo" width="64" height="64" class="mx-auto" />
				<h1 class="text-3xl font-bold title-animate">{$t('auth.login')}</h1>
				<p class="text-muted-foreground text-sm">Enter your credentials to access your account.</p>
			</div>
			<form method="post" use:enhance class="grid gap-4 input-animate">
				<div class="grid gap-2">
					<Label for="username">{$t('auth.username')}</Label>
					<Input
						id="username"
						name="username"
						type="text"
						placeholder="Enter your username"
						required
						class="w-full"
					/>
				</div>
				<div class="grid gap-2">
					<div class="flex items-center justify-between">
						<Label for="password">{$t('auth.password')}</Label>
					</div>
					<Input
						id="password"
						name="password"
						type="password"
						placeholder="Enter your password"
						required
						class="w-full"
					/>
				</div>
				{#if $page.form?.mfa_required}
					<div class="grid gap-2">
						<Label for="totp">TOTP</Label>
						<Input
							id="totp"
							name="totp"
							type="text"
							placeholder="Enter TOTP code"
							required
							class="w-full"
							autocomplete="one-time-code"
							pattern="[0-9]*"
							inputmode="numeric"
						/>
					</div>
				{/if}
				<Button type="submit" class="w-full py-3">
					{$t('auth.login')}
				</Button>
				{#if socialProviders.length > 0}
					<div class="divider my-4 text-center text-sm">
						{$t('auth.or_3rd_party')}
					</div>
					<div class="flex justify-center gap-4">
						{#each socialProviders as provider (provider.provider)}
							<Button href={provider.url} class="social-animate flex items-center space-x-2">
								{#if provider.provider === 'github'}
									<GitHub class="w-5 h-5" />
								{:else if provider.provider === 'openid_connect'}
									<OpenIdConnect class="w-5 h-5" />
								{/if}
								<span>{provider.name}</span>
							</Button>
						{/each}
					</div>
				{/if}
				<div class="mt-4 text-center text-sm">
					Don't have an account?
					<a href="/signup" class="underline hover:text-primary transition-colors">Sign up</a>
					<br />
					<a href="/user/reset-password" class="underline hover:text-primary transition-colors">
						Forgot password?
					</a>
				</div>
				{#if ($page.form?.message && $page.form?.message.length > 1) || $page.form?.type === 'error'}
					<div class="text-center text-destructive mt-4">
						{$t($page.form.message) || $t('auth.login_error')}
					</div>
				{/if}
			</form>
		</div>
	</div>
	<!-- Right: Image (now forced to full viewport height) -->
	<div class="hidden lg:block relative h-screen">
		<img
			src={background.url}
			alt="Login illustration"
			width="1920"
			height="1080"
			class="h-full w-full object-cover"
		/>
		<div class="absolute inset-0 flex items-center justify-center p-8">
			<div class="text-center">
				<p class="text-2xl font-semibold">"{quote.quote}"</p>
				<p class="text-lg mt-2">- {quote.author}</p>
			</div>
		</div>
	</div>
</div>

<!-- Floating button for Image Info Modal -->
<div class="fixed bottom-4 right-4 z-50">
	<Button variant="ghost" class="btn-circle" on:click={() => (isImageInfoModalOpen = true)}>
		<FileImageBox class="w-4 h-4" />
	</Button>
</div>

<svelte:head>
	<title>Login | AdventureLog</title>
	<meta
		name="description"
		content="Login to your AdventureLog account to start logging your adventures!"
	/>
</svelte:head>
