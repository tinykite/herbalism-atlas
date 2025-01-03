<script lang="ts">
	import { page } from '$app/state';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	interface Props {
		options: Array<string>;
	}

	let searchInput = $state('');
	let searchQuery = $state('');

	let { options }: Props = $props();
	let locations = $state([]);

	let menu = $state();
	let showMenu = $state(false);

	let filterValue = $state();

	onMount(() => {
		const { url } = page;

		if (url?.searchParams) {
			updateSearchInput(url);
		}
	});

	const updateSearchInput = (url) => {
		const city = url.searchParams.get('city');
		const state = url.searchParams.get('state');

		if (city && state) {
			searchQuery = `${city}, ${state}`;
		} else if (state) {
			searchQuery = state;
		}
	};

	const handleClick = (e) => {
		let value;

		if (e.target.matches('p')) {
			const parentTarget = e.target.parentElement;
			value = parentTarget.dataset.value;
		} else {
			value = e.target.dataset.value;
		}

		handleUpdate(value);
	};

	const handleUpdate = (value: string) => {
		searchQuery = value;
		showMenu = false;

		goto(`/?q=${value}`);
	};

	const handleMenuNavigation = (e) => {
		const dataAttributes = e.target.dataset;
		const currentIndex = parseInt(dataAttributes.index);

		switch (e.code) {
			case 'ArrowUp':
				if (currentIndex === 0) {
					const nextIndex = locations.length - 1;
					const nextTarget = menu.querySelector(`[data-index="${nextIndex}"]`);
					nextTarget.focus();
				} else {
					const nextIndex = currentIndex - 1;
					const nextTarget = menu.querySelector(`[data-index="${nextIndex}"]`);
					nextTarget.focus();
				}
				break;

			case 'ArrowDown':
				e.preventDefault();
				if (currentIndex <= locations.length - 2) {
					const nextIndex = currentIndex + 1;
					const nextTarget = menu.querySelector(`[data-index="${nextIndex}"]`);
					nextTarget.focus();
				} else {
					const nextTarget = menu.querySelector(`[data-index="0"]`);
					nextTarget.focus();
				}
				break;
			case 'Escape':
				searchQuery = '';
				locations = [];
				searchInput.focus();
				break;

			case 'Enter':
			case 'Space':
				filterValue = e.target.dataset.value;
				handleUpdate(filterValue);
			default:
				return;
		}
	};
	const handleSearchNavigation = (e) => {
		switch (e.code) {
			case 'Escape':
			case 'Tab':
				locations = [];
				break;
			case 'ArrowDown':
				if (showMenu && document.activeElement === searchInput) {
					const firstOption = menu.querySelectorAll('[data-index]')[0];
					return firstOption.focus();
				}
				break;
			case 'ArrowUp':
				console.log('arrow up hit');
				break;
			default:
				return;
		}
	};

	const handleLocationSearch = () => {
		if (searchQuery.length < 2) {
			return;
		}
		const autocompleteSuggestions = filterOptions(searchQuery);

		if (autocompleteSuggestions.length) {
			locations = autocompleteSuggestions;
			showMenu = true;
		}
	};

	const filterOptions = (value: string): string[] => {
		const targetValue = value.toLowerCase();
		return options.filter((option) => option.toLowerCase().includes(targetValue));
	};

	let timer;

	const debounceSearchInput = () => {
		clearTimeout(timer);
		if (searchQuery === '') {
			locations = [];
			goto('/');
		}

		timer = setTimeout(() => {
			handleLocationSearch();
		}, 200);
	};
</script>

<search class="searchWrapper">
	<div class="search">
		<svg
			class="search__icon"
			aria-hidden="true"
			xmlns="http://www.w3.org/2000/svg"
			viewBox="0 0 24 24"
			fill="none"
			stroke="currentColor"
			stroke-width="2"
			stroke-linecap="round"
			stroke-linejoin="round"><circle cx="11" cy="11" r="8" /><path d="m21 21-4.3-4.3" /></svg
		>

		<form method="POST" action="/">
			<label id="search" class="u-visuallyHidden" for="location">Search</label>
			<input
				bind:this={searchInput}
				class="search__input"
				type="search"
				role="combobox"
				aria-controls="location-results"
				aria-autocomplete="list"
				aria-expanded="false"
				autocomplete="off"
				name="location"
				id="location"
				onkeyup={(e) => handleSearchNavigation(e)}
				oninput={() => debounceSearchInput()}
				bind:value={searchQuery}
			/>
		</form>
	</div>
	{#if showMenu}
		<ul
			class="autocomplete"
			id="location-results"
			aria-labelledby="location-search"
			role="listbox"
			bind:this={menu}
			onclick={(e) => handleClick(e)}
			onkeydown={(e) => handleMenuNavigation(e)}
		>
			{#each locations as location, i}
				<li
					class="autocomplete__option"
					role="option"
					aria-selected="false"
					tabindex="-1"
					data-index={i}
					data-value={location}
				>
					<svg
						class="autocomplete__icon"
						xmlns="http://www.w3.org/2000/svg"
						viewBox="0 0 24 24"
						fill="none"
						stroke="currentColor"
						aria-hidden="true"
						stroke-width="2"
						stroke-linecap="round"
						stroke-linejoin="round"
						><path
							d="M20 10c0 4.993-5.539 10.193-7.399 11.799a1 1 0 0 1-1.202 0C9.539 20.193 4 14.993 4 10a8 8 0 0 1 16 0"
						/><circle cx="12" cy="10" r="3" /></svg
					>
					<p class="location__commonName">{location}</p>
				</li>
			{/each}

			{#if searchQuery.length >= 3 && locations.length === 0}
				<li
					class="autocomplete__option autocomplete__option--empty"
					role="option"
					aria-selected="false"
				>
					No results
				</li>
			{/if}
		</ul>
	{/if}
</search>

<style>
	.searchWrapper {
		position: relative;
	}
	.search {
		display: flex;
		align-items: center;
		column-gap: 0.5rem;
		position: relative;
		color: #397740;
	}

	.search__icon {
		position: absolute;
		height: 0.9rem;
		width: auto;
		margin-inline-start: 0.75rem;
		color: currentColor;
		pointer-events: none;
	}

	.search__input {
		border-radius: 9999px;
		padding-inline-start: 1.85rem;
		padding-inline-end: 1rem;
		padding-block: 0.25rem;
		border: 1px solid #6f976e;
		outline-offset: 4px;
		font-family: 'tiempos', 'serif';
		font-weight: 500;
		font-style: normal;
	}

	.search__input:hover {
		border-color: #184b35;
	}
	.search__input:focus-visible,
	.search__input:focus {
		outline-style: solid;
		outline-width: 2px;
		outline-color: #4900fc;
	}

	.search__input::placeholder {
		font-size: 1rem;
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		color: rgba(57, 119, 64, 0.7);
	}

	.autocomplete {
		display: grid;
		position: absolute;
		border-radius: 0.5rem;
		background: white;
		list-style-type: none;
		padding: 0;
		margin-block-start: 1.5rem;
		overflow: auto;
		max-height: 50vh;
		width: 100%;
		box-shadow:
			0 4px 80px 4px rgba(0, 0, 0, 0.02),
			0 1px 22px 0 rgba(0, 0, 0, 0.08);
	}

	.autocomplete__option {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin: 0;
		padding-inline: 1rem;
		padding-block: 0.5rem;
		cursor: pointer;
		border-bottom: 1px solid rgba(56, 80, 50, 0.1);
	}

	.autocomplete__icon {
		height: 1rem;
		width: auto;
	}

	.autocomplete__option--empty {
		text-align: center;
	}
	.autocomplete__option:hover,
	.autocomplete__option:focus {
		background-color: #4c3a78;
		color: #fff;
		outline: none;
	}
</style>
