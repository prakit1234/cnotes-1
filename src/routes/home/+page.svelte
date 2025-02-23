<script lang="ts">
	import { showToast } from '$lib/utils/svelteToastsUtil';
	import { onMount } from 'svelte';
	import SvelteToast from '../components/svelteToast.svelte';
	import type { note, searchResult } from '../types';
	import Note from '../components/note.svelte';
	import { notesStore } from '$lib/stores/noteStore'; // Import the store

	// Variables
	let error: string = '';
	let notes: note[] = [];
	let searchQuery: string = '';
	let searchResults: searchResult[];
	let shouldShowSearchResults: boolean = false;
	let conf = {
		height: 700,
		menubar: false,
		shortcuts: false,
		skin: 'oxide-dark',
		content_css: 'dark',
		plugins: [
			'advlist',
			'autolink',
			'lists',
			'link',
			'image',
			'charmap',
			'anchor',
			'searchreplace',
			'visualblocks',
			'code',
			'fullscreen',
			'insertdatetime',
			'media',
			'table',
			'preview',
			'help',
			'wordcount'
		],
		toolbar:
			'undo redo | blocks | ' +
			'bold italic forecolor underline | alignleft aligncenter alignright alignjustify | bullist numlist | ' +
			'table tabledelete | tableinsertrowbefore tableinsertrowafter tabledeleterow | tableinsertcolbefore tableinsertcolafter tabledeletecol' +
			'bullist numlist outdent indent | ' +
			' help',
		a11y_advanced_options: true,
		file_picker_types: 'image'
	};

	// Subscribe to the store
	notesStore.subscribe((value) => {
		notes = value; // Update local notes variable when store changes
	});

	// Functions
	async function getNotes(userEmail: string) {
		if (userEmail) {
			try {
				const request = await fetch('/api/notes', {
					method: 'POST',
					headers: {
						'Content-Type': 'application/json'
					},
					body: JSON.stringify({
						Email: userEmail
					})
				});
				const result = await request.json();
				if (result.status === 'success') {
					notesStore.set(result.response); // Set the store with fetched notes
					localStorage.setItem('notes', JSON.stringify(result.response));
				} else {
					error = result.message;
				}
			} catch (error) {
				console.error('Error:', error);
			}
		} else {
			showToast('error', 'Please login to view your notes.', 2000, 'error');
		}
	}
	onMount(() => {
		const userEmail = sessionStorage.getItem('Email');
		getNotes(userEmail);

		// Add click event listener to document to hide search results when clicking outside
		document.addEventListener('click', (event) => {
			const searchBar = document.querySelector('.search-bar');
			if (searchBar && !searchBar.contains(event.target as Node)) {
				shouldShowSearchResults = false;
			}
		});
	});
	function search() {
		if (searchQuery.length > 0) {
			shouldShowSearchResults = true;
			// Use filter and map for better performance
			const matches = notes
				.filter((note) => note.title.toLowerCase().includes(searchQuery.toLowerCase()))
				.map((note) => ({
					title: note.title,
					slug: note.slug
				}));

			searchResults = matches.map((match) => ({
				title: match.title,
				slug: match.slug
			}));
		} else {
			shouldShowSearchResults = false;
			searchResults = null;
		}
	}
	function handleKeyDown(event: any) {
		search();
	}
</script>

<SvelteToast />

<div class="main">
	<div class="header">
		<div class="search-bar mb-5 mt-5 grow pl-5 pr-5">
			<label class="input input-bordered flex items-center gap-2">
				<input
					type="text"
					class="grow search-input"
					placeholder="Search for a note"
					on:keydown={handleKeyDown}
					bind:value={searchQuery}
				/>

				<button on:click={search} class="btn btn-circle btn-ghost search-button" aria-label="Search">
					<svg
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 16 16"
						fill="currentColor"
						class="h-4 w-4 opacity-70"
					>
						<path
							fill-rule="evenodd"
							d="M9.965 11.026a5 5 0 1 1 1.06-1.06l2.755 2.754a.75.75 0 1 1-1.06 1.06l-2.755-2.754ZM10.5 7a3.5 3.5 0 1 1-7 0 3.5 3.5 0 0 1 7 0Z"
							clip-rule="evenodd"
						/>
					</svg>
				</button>
			</label>
			{#if shouldShowSearchResults}
				<div class="search-results mt-2 p-2">
					{#if searchResults.length > 0}
						{#each searchResults as searchResult}
							<a href={`#${searchResult.slug}`}>
								<p class="text-lg font-bold">
									{searchResult.title}
								</p>
							</a>
							<hr />
						{/each}
					{:else}
						<p>No note found.</p>
					{/if}
				</div>
			{/if}
		</div>
		<div class="add-note">
			<a class="addNoteButton btn bg-accent" href="/home/add-new-note">New Note</a>
		</div>
	</div>
	<div class="notes p-5">
		{#if error}
			<p class="error">{error}</p>
		{:else if notes.length > 0}
			<div class="notes-grid">
				{#each notes as note}
					<Note {note} />
				{/each}
			</div>
		{:else}
			<p class="loadingNotes">Loading your Notes...</p>
		{/if}
	</div>
</div>

<style>
	.header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		gap: 15px;
	}

	/* Search bar styles with gradient and smooth animations */
	.search-bar {
		background: linear-gradient(90deg, #4a90e2, #007bff);
		border-radius: 12px;
		padding: 12px;
		box-shadow: 0 4px 10px rgba(0, 123, 255, 0.3);
		transition: box-shadow 0.4s ease, transform 0.4s ease;
	}

	.search-bar:hover {
		box-shadow: 0 6px 15px rgba(0, 123, 255, 0.5);
		transform: translateY(-3px);
	}

	/* Input styles */
	.search-input {
		width: 100%;
		padding: 10px;
		border: none;
		border-radius: 8px;
		font-size: 16px;
		color: #fff;
		background: rgba(255, 255, 255, 0.2);
		outline: none;
		transition: background 0.3s ease;
	}

	.search-input::placeholder {
		color: rgba(255, 255, 255, 0.7);
	}

	.search-input:focus {
		background: rgba(255, 255, 255, 0.4);
	}

	/* Button styles with simpler gradient and animations */
	.search-button {
		background: linear-gradient(90deg, #007bff, #0056b3);
		color: white;
		border: none;
		border-radius: 8px;
		padding: 10px 20px;
		font-size: 16px;
		cursor: pointer;
		transition: transform 0.4s ease, background-color 0.4s ease;
	}

	.search-button:hover {
		transform: scale(1.05);
		background: linear-gradient(90deg, #0056b3, #003d80);
	}

	/* Add Note button styles moved to the left for cleaner layout */
	.addNoteButton {
		background: linear-gradient(90deg, #4a90e2, #007bff);
		color: white;
		border: none;
		border-radius: 8px;
		padding: 12px 24px;
		font-size: 16px;
		cursor: pointer;
		transition: transform 0.4s ease, background 0.4s ease;
		margin-left: 20px;
	}

	.addNoteButton:hover {
		transform: translateY(-3px);
		background: linear-gradient(90deg, #007bff, #0056b3);
	}

	/* Search results container with smoother animations */
	.search-results {
		background: #f0f4f8;
		border-radius: 8px;
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
		padding: 10px;
		margin-top: 10px;
		transition: opacity 0.4s ease, transform 0.4s ease;
		transform: translateY(-5px);
		opacity: 0;
	}

	.search-results.fade-in {
		transform: translateY(0);
		opacity: 1;
	}

	.search-results.fade-out {
		transform: translateY(-5px);
		opacity: 0;
	}
</style>
