<script lang="ts">
	import { goto } from '$app/navigation';

	// Imports
	import SvelteToast from './svelteToast.svelte';
	import { showToast } from '$lib/utils/svelteToastsUtil';
	import { showModal } from '$lib/stores/showLoginForm';
	import { onMount } from 'svelte';

	// Variables
	let type = $props();
	let username = $state('');
	let email = $state('');
	let password = $state('');
	let sign = $state('');
	let cookieValue = $state('');

	// Functions
	onMount(async () => {
		const response = await fetch('/api/cookie', {
			method: 'GET',
			credentials: 'include'
		});
		const result = await response.json();
		if (result.message == 'success') {
			cookieValue = result.cookieValue;
		} else {
			console.error('Failed to fetch cookie.', result.message);
		}
	});
	async function login() {
		if (email && password) {
			const response = await fetch('/api/login', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					email: email,
					password: password,
					session_id: cookieValue
				})
			});
			const result = await response.json();
			console.log(result);
			if (result.message == 'user not found.') {
				showToast('Error', 'User not found. Please register.', 5000, 'error');
			} else if (result.message == 'invalid credentials.') {
				showToast('Error', 'Invalid credentials.', 5000, 'error');
			} else if (result.message == 'success') {
				sessionStorage.setItem('Email', email);
				showToast('Success', 'Successfully logged in.', 2000, 'success');
				goto('/home');
			} else if (result.status == 'error') {
				showToast('Error', 'There was an error', 5000, 'error');
			}
		}
	}
	async function register() {
		if (username && email && password) {
			try {
				const response = await fetch('/api/register', {
					method: 'POST',
					headers: {
						'Content-Type': 'application/json'
					},
					body: JSON.stringify({
						username: username,
						email: email,
						password: password
					})
				});
				const result = await response.json();
				if (result.message == 'User already exists.') {
					showToast('Error', 'User already exists. Please login.', 5000, 'error');
				} else if (result.message == 'New user created.') {
					sessionStorage.setItem('Email', email);
					showToast('Success', 'User created successfully.', 2000, 'success');
					goto('/home');
				} else if ((result.status = 'error')) {
					showToast('Error', 'There was an error', 5000, 'error');
				}
			} catch (error) {
				console.error(error);
			}
		} else {
			showToast('Error', 'Please fill all fields.', 5000, 'error');
		}
	}
	function removeFromView() {
		showModal.set(false);
	}
	if (type.type == 'register') {
		sign = 'Sign up';
	} else {
		sign = 'Login';
	}
</script>

<SvelteToast />

<form class="card-body">
	<div class="cart-title inline-flex">
		<h1 class="text-2xl font-bold">{sign}</h1>
		<div class="card-actions ml-auto">
			<a href="/" class="btn" id="close-button" onclick={removeFromView}>X</a>
		</div>
	</div>
	{#if type.type == 'register'}
		<div class="form-control">
			<label class="label input-bordered flex items-center gap-2">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 16 16"
					fill="currentColor"
					class="h-4 w-4 opacity-70"
				>
					<path
						d="M8 8a3 3 0 1 0 0-6 3 3 0 0 0 0 6ZM12.735 14c.618 0 1.093-.561.872-1.139a6.002 6.002 0 0 0-11.215 0c-.22.578.254 1.139.872 1.139h9.47Z"
					/>
				</svg>
				<input
					type="name"
					placeholder="Username"
					class="input input-bordered grow"
					required
					id="username"
					bind:value={username}
				/>
			</label>
		</div>
	{/if}
	<div class="form-control">
		<label class="label input-bordered flex items-center gap-2">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 16 16"
				fill="currentColor"
				class="h-4 w-4 opacity-70"
			>
				<path
					d="M2.5 3A1.5 1.5 0 0 0 1 4.5v.793c.026.009.051.02.076.032L7.674 8.51c.206.1.446.1.652 0l6.598-3.185A.755.755 0 0 1 15 5.293V4.5A1.5 1.5 0 0 0 13.5 3h-11Z"
				/>
				<path
					d="M15 6.954 8.978 9.86a2.25 2.25 0 0 1-1.956 0L1 6.954V11.5A1.5 1.5 0 0 0 2.5 13h11a1.5 1.5 0 0 0 1.5-1.5V6.954Z"
				/>
			</svg>

			<input
				type="email"
				placeholder="Email"
				class="input input-bordered grow"
				required
				id="email"
				bind:value={email}
			/>
		</label>
	</div>
	<div class="form-control">
		<label class="label input-bordered flex items-center gap-2">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 16 16"
				fill="currentColor"
				class="h-4 w-4 opacity-70"
			>
				<path
					fill-rule="evenodd"
					d="M14 6a4 4 0 0 1-4.899 3.899l-1.955 1.955a.5.5 0 0 1-.353.146H5v1.5a.5.5 0 0 1-.5.5h-2a.5.5 0 0 1-.5-.5v-2.293a.5.5 0 0 1 .146-.353l3.955-3.955A4 4 0 1 1 14 6Zm-4-2a.75.75 0 0 0 0 1.5.5.5 0 0 1 .5.5.75.75 0 0 0 1.5 0 2 2 0 0 0-2-2Z"
					clip-rule="evenodd"
				/>
			</svg>
			<input
				type="password"
				placeholder="Password"
				class="input input-bordered grow"
				required
				id="password"
				bind:value={password}
			/>
		</label>
		<span class="label">
			<a href="/forgot-password" class="link-hover link label-text-alt">Forgot password?</a>
		</span>
	</div>
	<div class="form-control mt-6">
		{#if type.type == 'login'}
			<button class="btn btn-primary" onclick={login}>Login</button>
		{:else}
			<button class="btn btn-primary" onclick={register}>Sign In</button>
		{/if}
	</div>
</form>
