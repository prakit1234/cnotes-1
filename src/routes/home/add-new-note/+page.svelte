<script lang="ts">
	// Imports
	import { showToast } from '$lib/utils/svelteToastsUtil';
	import type { note } from '../../types';
	import SvelteToast from '../../components/svelteToast.svelte';
	import Editor from '../../components/editor.svelte';
	import { goto } from '$app/navigation';

	// Variables
	// @ts-expect-error
	let newNote: note[] = [{}];
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

	// Functions
	const quickValidate = (note) => {
		return (
			(note?.title &&
				note?.note_content &&
				note?.subject &&
				note?.grade &&
				note?.board &&
				note?.date_created &&
				note?.school) ??
			false
		);
	};
	async function addNewNote() {
		if (quickValidate(newNote[0])) {
			showToast('Saving...', 'Saving your note', 2500, 'info');
			const response = await fetch('/api/notes/note/add-note', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					note: newNote,
					email: sessionStorage.getItem('Email')
				})
			});
			const result = await response.json();
			if (result.status === 'success') {
				showToast('Success', 'Note added successfully', 2500, 'success');
				goto('/home');
			} else {
				showToast('Error', result.message, 2500, 'error');
			}
		} else {
			showToast('Error', 'Please fill all the fields', 2500, 'error');
		}
	}
</script>

<SvelteToast />

<div class="main">
	<form>
		<div class="header-box">
			<h2 class="mb-2 text-3xl">Add a New Note</h2>
			<div class="new-note-data">
				<label class="form-control w-full max-w-xs">
					<div class="label">
						<span class="label-text">Title:</span>
					</div>
					<input
						type="text"
						class="input input-bordered w-full max-w-xs"
						bind:value={newNote[0].title}
						required
						placeholder="Title"
					/>
				</label>
				<label class="form-control w-full max-w-xs">
					<div class="label">
						<span class="label-text">Board:</span>
					</div>
					<input
						type="text"
						class="input input-bordered w-full max-w-xs"
						bind:value={newNote[0].board}
						required
						placeholder="Board"
					/>
				</label>
				<label class="form-control w-full max-w-xs">
					<div class="label">
						<span class="label-text">Date Created:</span>
					</div>
					<input
						type="date"
						class="input input-bordered w-full max-w-xs"
						bind:value={newNote[0].date_created}
						required
						placeholder="Date Created"
					/>
				</label>
				<label class="form-control w-full max-w-xs">
					<div class="label">
						<span class="label-text">Grade:</span>
					</div>
					<input
						type="text"
						class="input input-bordered w-full max-w-xs"
						bind:value={newNote[0].grade}
						required
						placeholder="Grade"
					/>
				</label>
				<label class="form-control w-full max-w-xs">
					<div class="label">
						<span class="label-text">School:</span>
					</div>
					<input
						type="text"
						class="input input-bordered w-full max-w-xs"
						bind:value={newNote[0].school}
						required
						placeholder="School"
					/>
				</label>
				<label class="form-control w-full max-w-xs">
					<div class="label">
						<span class="label-text">Subject:</span>
					</div>
					<input
						type="text"
						class="input input-bordered w-full max-w-xs"
						bind:value={newNote[0].subject}
						required
						placeholder="Subject"
					/>
				</label>
				<label class="form-control">
					<div class="label">
						<span class="label-text">Note Content</span>
					</div>
					<Editor data={newNote} {conf} />
				</label>
			</div>
			<br /><br />
		</div>
		<button class="btn btn-outline btn-primary" on:click={addNewNote}>Add Note</button>
	</form>
</div>

<style>
  /* General layout styles */
  .main {
    padding: 20px;
    background-color: #000000; /* Black theme */
    color: #ffffff; /* White text */
    animation: fadeInDown 0.8s ease-in-out;
  }

  .header-box {
    text-align: center;
    margin-bottom: 20px;
    animation: fadeInDown 0.6s ease-in-out;
  }

  .header-box h2 {
    font-size: 2.5rem;
    font-weight: bold;
    color: #4caf50; /* Green for header text */
    animation: fadeInDown 0.8s ease-in-out;
  }

  /* Form container */
  .new-note-data {
    display: flex;
    flex-direction: column; /* Align fields vertically */
    gap: 20px; /* Space between fields */
    animation: fadeInDown 1s ease-in-out;
  }

  /* Label styling */
  .form-control {
    width: 100%; /* Full width for inputs */
    display: flex;
    flex-direction: column;
  }

  .form-control .label-text {
    font-weight: 600;
    color: #cccccc; /* Light gray for labels */
    margin-bottom: 5px;
    transition: color 0.3s ease-in-out;
  }

  .form-control .label-text:hover {
    color: #4caf50; /* Green on hover */
  }

  /* Input styling */
  .form-control input {
    padding: 10px;
    border: 2px solid #333333;
    border-radius: 8px;
    font-size: 1rem;
    background-color: #111111; /* Darker background for inputs */
    color: #ffffff;
    transition: all 0.3s ease-in-out;
  }

  .form-control input:focus {
    outline: none;
    border-color: #4caf50;
    background-color: #222222; /* Slightly lighter on focus */
    box-shadow: 0 0 8px rgba(76, 175, 80, 0.4);
  }

  /* Button styling */
  .btn {
    padding: 10px 20px;
    font-size: 1rem;
    font-weight: 600;
    border: 2px solid #4caf50;
    border-radius: 8px;
    background-color: #4caf50;
    color: #000000; /* Black text for contrast */
    cursor: pointer;
    transition: all 0.3s ease-in-out;
    align-self: center; /* Center align the button */
  }

  .btn:hover {
    background-color: #333333; /* Dark hover effect */
    color: #4caf50;
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(76, 175, 80, 0.5);
  }

  .btn:active {
    transform: translateY(0);
    box-shadow: none;
  }

  /* Animations */
  @keyframes fadeInDown {
    from {
      opacity: 0;
      transform: translateY(-50px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  /* Ensure responsiveness for both desktop and mobile */
  @media (max-width: 768px) {
    .main {
      padding: 10px;
    }
    .new-note-data {
      gap: 15px;
    }
    .header-box h2 {
      font-size: 2rem;
    }
    .btn {
      width: 100%; /* Full-width button on smaller screens */
    }
  }
</style>
