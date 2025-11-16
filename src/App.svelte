<script>
	import { EXERCISES } from './lib/exercises.js'
	import { EXERCISES_2_MOVEMENTS } from './lib/exercises-2-movements.js'
	import { EXERCISES_1_MOVEMENT } from './lib/exercises-1-movement.js'
	import { BREATHING_EXERCISES } from './lib/breathing.js'
	import { SMOKING_METHODS } from './lib/smoking.js'
	import { onMount } from 'svelte'

	let currentPage = 'menu' // 'menu' | 'exercises-select' | 'exercises' | 'breathing' | 'smoking' | 'favorites'
	let workoutType = null // null | 4 | 2 | 1
	let selectedExercises = []
	let selectedBreathing = null
	let selectedSmoking = null
	let selectedExerciseIds = new Set() // track clicked cards
	let lastExerciseId = null // track last generated exercise to avoid duplicates
	let favorites = JSON.parse(localStorage.getItem('favorites') || '[]') // load favorites from localStorage
	let workoutName = '' // name for saving a workout
	let favoritesHoverActive = false // for button hover state

	// Audio / music player state
	let audioUrl = null
	let audioPlaying = false
	/** @type {HTMLAudioElement|null} */
	let audioEl = null
	/** @type {HTMLInputElement|null} */
	let fileInputEl = null

	function toggleAudio() {
		if (!audioUrl) {
			// prompt user to choose a local file
			if (fileInputEl) fileInputEl.click()
			return
		}
		if (!audioEl) return
		if (audioPlaying) {
			if (audioEl) audioEl.pause()
			audioPlaying = false
		} else {
			if (audioEl) audioEl.play()
			audioPlaying = true
		}
	}

	function handleFileSelected(e) {
		const f = e.target.files && e.target.files[0]
		if (!f) return
		if (audioUrl) {
			URL.revokeObjectURL(audioUrl)
		}
		audioUrl = URL.createObjectURL(f)
		if (audioEl) {
			audioEl.src = audioUrl
			audioEl.play()
			audioPlaying = true
		}
	}

	function pickExerciseAvoidingLast(availableExercises) {
		// Filter out the last exercise to avoid consecutive duplicates
		let pool = availableExercises
		if (lastExerciseId) {
			pool = availableExercises.filter(e => e.id !== lastExerciseId)
		}
		
		if (pool.length === 0) {
			// If all are filtered out, use the full list
			pool = availableExercises
		}
		
		const i = Math.floor(Math.random() * pool.length)
		return pool[i]
	}

	function pickBalancedExercises(count = 4) {
		// Choose correct dataset based on workout type
		let dataset = EXERCISES
		if (count === 2) dataset = EXERCISES_2_MOVEMENTS
		if (count === 1) dataset = EXERCISES_1_MOVEMENT

		const categories = ['push', 'pull', 'squat', 'hinge'].slice(0, count)
		const pool = [...dataset]
		const out = []

		// Keep selected exercises in their positions
		for (let i = 0; i < selectedExercises.length; i++) {
			const ex = selectedExercises[i]
			if (selectedExerciseIds.has(ex.id)) {
				out[i] = ex
			}
		}

		// Fill remaining slots with balanced picks
		for (const cat of categories) {
			// Check if this category is already filled by a selected card
			let alreadyHas = out.some(e => e && e.category === cat)
			if (alreadyHas) continue

			const byCat = pool.filter(e => e.category === cat)
			if (byCat.length) {
				const picked = pickExerciseAvoidingLast(byCat)
				const slotIdx = out.findIndex(e => !e)
				if (slotIdx !== -1) {
					out[slotIdx] = picked
					lastExerciseId = picked.id
				} else {
					out.push(picked)
					lastExerciseId = picked.id
				}
				const idx = pool.findIndex(p => p.id === picked.id)
				if (idx !== -1) pool.splice(idx, 1)
			}
		}

		// Fill any remaining empty slots
		while (out.length < count && pool.length) {
			const picked = pickExerciseAvoidingLast(pool)
			out.push(picked)
			lastExerciseId = picked.id
			const idx = pool.findIndex(p => p.id === picked.id)
			if (idx !== -1) pool.splice(idx, 1)
		}

		selectedExercises = out.filter(e => e) // remove undefined entries
	}

	function saveWorkout() {
		if (workoutName.trim() && selectedExercises.length > 0) {
			const workout = {
				id: Date.now(),
				name: workoutName,
				exercises: selectedExercises,
				type: workoutType,
				date: new Date().toLocaleDateString('fr-FR')
			}
			favorites = [...favorites, workout]
			localStorage.setItem('favorites', JSON.stringify(favorites))
			workoutName = ''
			alert(`Entraînement "${workout.name}" sauvegardé!`)
		}
	}

	function deleteFavorite(id) {
		favorites = favorites.filter(f => f.id !== id)
		localStorage.setItem('favorites', JSON.stringify(favorites))
	}

	function pickRandomBreathing() {
		const i = Math.floor(Math.random() * BREATHING_EXERCISES.length)
		selectedBreathing = BREATHING_EXERCISES[i]
	}

	function pickRandomSmoking() {
		const i = Math.floor(Math.random() * SMOKING_METHODS.length)
		selectedSmoking = SMOKING_METHODS[i]
	}

	function capitalize(s) {
		return s ? s.charAt(0).toUpperCase() + s.slice(1) : ''
	}

	function goToMenu() {
		currentPage = 'menu'
		workoutType = null
	}

	function goToExerciseSelect() {
		currentPage = 'exercises-select'
		workoutType = null
	}

	function startWorkout(type) {
		workoutType = type
		currentPage = 'exercises'
		selectedExercises = []
		selectedExerciseIds.clear()
		lastExerciseId = null
	}

	function goToExercises() {
		currentPage = 'exercises'
		selectedExercises = []
		selectedExerciseIds.clear()
	}

	function toggleExerciseSelection(exId) {
		if (selectedExerciseIds.has(exId)) {
			selectedExerciseIds.delete(exId)
		} else {
			selectedExerciseIds.add(exId)
		}
		selectedExerciseIds = selectedExerciseIds // trigger reactivity
	}

	function goToBreathing() {
		currentPage = 'breathing'
		selectedBreathing = null
	}

	function goToSmoking() {
		currentPage = 'smoking'
		selectedSmoking = null
	}

onMount(() => {
	// If a bundled audio file exists in the assets folder, load it automatically
	try {
		const bundled = new URL('../assets/audio/Like A G6.mp3', import.meta.url).href
		audioUrl = bundled
		if (audioEl) {
			audioEl.src = audioUrl
			audioEl.preload = 'auto'
			audioEl.volume = 0.9
		}
	} catch (err) {
		// file might not exist; fallback keeps file input for user selection
		console.warn('Bundled audio not found or failed to load:', err)
	}
})
</script>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		overflow: hidden;
		background-color: #1a1a1a;
		color: #e0e0e0;
	}

	.app {
		display: flex;
		flex-direction: column;
		height: 100vh;
		width: 100vw;
		overflow: hidden;
		font-family: system-ui, sans-serif;
		background-color: #1a1a1a;
	}

	header {
		background-color: #0f1419;
		color: #b294cc;
		padding: 0.75rem;
		text-align: center;
		border-bottom: 1px solid rgba(178, 148, 204, 0.2);
		position: relative;
		flex-shrink: 0;
	}

	header h1 {
		margin: 0;
		font-size: 1.2rem;
	}

	header h3 {
		margin: 0.25rem 0 0 0;
		font-size: 0.85rem;
		font-weight: 400;
	}

	.favorites-btn {
		position: absolute;
		right: 1rem;
		top: 50%;
		transform: translateY(-50%);
		background-color: #b294cc;
		color: #1a1a1a;
		border: none;
		border-radius: 50%;
		width: 50px;
		height: 50px;
		font-size: 1.5rem;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: background-color 0.2s;
		font-weight: bold;
	}

	.music-btn {
		position: absolute;
		left: 1rem;
		top: 50%;
		transform: translateY(-50%);
		background-color: transparent;
		color: #b294cc;
		border: 1px solid rgba(178,148,204,0.08);
		border-radius: 50%;
		width: 44px;
		height: 44px;
		font-size: 1.1rem;
		cursor: pointer;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.music-btn:hover {
		background-color: rgba(178,148,204,0.06);
	}

	.title-btn {
		background: transparent;
		border: none;
		color: inherit;
		cursor: pointer;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 0;
	}


	.favorites-btn:hover {
		background-color: #c4a0d6;
	}

	.favorites-badge {
		position: absolute;
		top: -5px;
		right: -5px;
		background-color: #1a1a1a;
		color: #b294cc;
		border-radius: 50%;
		width: 24px;
		height: 24px;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 0.75rem;
		border: 1px solid #b294cc;
	}

	.content {
		flex: 1;
		padding: 0.75rem;
		padding-bottom: calc(84px + env(safe-area-inset-bottom));
		display: flex;
		flex-direction: column;
		background-color: #1a1a1a;
	}

	/* Menu styles */
	.menu-container {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		max-width: 600px;
		margin: 1.5rem auto;
		width: 100%;
		padding: 0 0.5rem;
	}

	.menu-button {
		padding: 0.75rem 1rem;
		font-size: 0.95rem;
		background-color: #b294cc;
		color: #1a1a1a;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		transition: background-color 0.2s;
		font-weight: 600;
	}

	.menu-button:hover {
		background-color: #c4a0d6;
	}

	/* Page header with back button */
	.page-header {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		margin-bottom: 1rem;
		flex-shrink: 0;
	}

	.back-button {
		padding: 0.4rem 0.8rem;
		background-color: #333333;
		color: #b294cc;
		border: 1px solid #b294cc;
		border-radius: 5px;
		cursor: pointer;
		font-size: 0.8rem;
		transition: background-color 0.2s;
		flex-shrink: 0;
	}

	.back-button:hover {
		background-color: #444444;
	}

	.page-title {
		margin: 0;
		font-size: 1.2rem;
		color: #b294cc;
	}

	/* Cards container */
	.cards {
		display: flex;
		flex-direction: column;
		gap: 8px;
		width: 100%;
		margin-bottom: 0.75rem;
		flex: 1;
		overflow-y: auto;
	}

	.card {
		background: #2a2a2a;
		border: 2px solid #333333;
		border-radius: 8px;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
		display: flex;
		flex-direction: row;
		align-items: center;
		padding: 0.6rem;
		overflow: hidden;
		cursor: pointer;
		transition: all 0.2s;
		flex-shrink: 0;
	}

	.card:hover {
		border-color: #b294cc;
	}

	.card.selected {
		border-color: #b294cc;
		background: #333333;
		box-shadow: 0 0 10px rgba(178, 148, 204, 0.3);
	}

	.card-image {
		width: 80px;
		height: 80px;
		object-fit: cover;
		border-radius: 6px;
		margin-left: 0.5rem;
		flex-shrink: 0;
	}

	.card-body {
		display: flex;
		flex-direction: column;
		justify-content: center;
		flex: 1 1 auto;
		min-width: 0;
	}

	.card-category {
		font-size: 0.85rem;
		color: #b294cc;
		background: rgba(178, 148, 204, 0.1);
		padding: 0.25rem 0.5rem;
		border-radius: 6px;
		align-self: flex-start;
	}

	/* Single item display */
	.single-item {
		width: 100%;
		max-width: 400px;
		background: #2a2a2a;
		border: 1px solid #333333;
		border-radius: 10px;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.3);
		padding: 1rem;
		margin-bottom: 1.5rem;
	}

	.single-item img {
		width: 100%;
		height: 200px;
		object-fit: cover;
		border-radius: 6px;
		margin-bottom: 1rem;
	}

	.single-item-title {
		font-size: 1.2rem;
		margin: 0;
		color: #b294cc;
	}

	/* Button styles */
	.action-button {
		width: 100%;
		padding: 0.6rem 0.8rem;
		background-color: #b294cc;
		color: #1a1a1a;
		border: none;
		border-radius: 6px;
		font-size: 0.9rem;
		cursor: pointer;
		transition: background-color 0.2s;
		font-weight: 600;
		flex-shrink: 0;
	}

	.action-button:hover {
		background-color: #c4a0d6;
	}

	/* Bottom bar fixed for mobile so controls are always visible */
	.bottom-bar {
		position: fixed;
		left: 0;
		right: 0;
		bottom: env(safe-area-inset-bottom, 0);
		display: flex;
		display: none;
		gap: 0.5rem;
		padding: 0.5rem;
		padding-bottom: calc(8px + env(safe-area-inset-bottom));
		background: linear-gradient(180deg, rgba(15,20,25,0.95), rgba(15,20,25,0.98));
		border-top: 1px solid rgba(178,148,204,0.08);
		z-index: 60;
		box-shadow: 0 -4px 12px rgba(0,0,0,0.6);
	}

	.bottom-input {
		padding: 0.5rem;
		background-color: #2a2a2a;
		color: #e0e0e0;
		border: 1px solid #b294cc;
		border-radius: 6px;
		font-size: 0.9rem;
		flex: 1 1 auto;
	}

	.no-selection {
		color: #888888;
	}

	/* Responsive */
	@media (max-width: 1024px) {
		.card {
			flex: 1 1 calc(50% - 6px);
		}
	}

	@media (max-width: 680px) {
		.card {
			flex: 1 1 100%;
		}
		.cards {
			flex-direction: column;
		}
		.bottom-bar {
			display: flex;
		}
	}
</style>

<div class="app">
	<header style="display: flex; align-items: center; justify-content: space-between;">
		<div style="width: 56px; display:flex; justify-content:flex-start;">
			<button class="music-btn" on:click={toggleAudio} title="Lecture de la musique">{audioPlaying ? '⏸' : '🎵'}</button>
		</div>

		<button class="title-btn" on:click={goToMenu} aria-label="Aller au menu principal">
			<h1 style="margin: 0;">Simple Bien-être</h1>
			<h3 style="margin: 0.25rem 0 0 0;">Le générateur</h3>
		</button>

		<div style="width: 56px; display:flex; justify-content:flex-end;">
			<button 
				class="favorites-btn"
				on:click={() => currentPage = 'favorites'}
				title="Voir les séances favorites"
			>
				⭐{#if favorites.length > 0}<span class="favorites-badge">{favorites.length}</span>{/if}
			</button>
		</div>
	</header>
    
	<input bind:this={fileInputEl} type="file" accept="audio/*" on:change={handleFileSelected} style="display:none" />
	<audio bind:this={audioEl} on:ended={() => { audioPlaying = false }} preload="auto"></audio>

	<div class="content">
	{#if currentPage === 'menu'}
		<div class="menu-container">
			<h2 style="text-align: center; margin-top: 0;">Sélectionner une option</h2>
			<button class="menu-button" on:click={goToExerciseSelect}>🏋️‍♂️ Générateur d'Exercices</button>
			<button class="menu-button" on:click={goToBreathing}>🧘 Respiration et Conscience</button>
			<button class="menu-button" on:click={goToSmoking}>🚬 Méthodes de Fumage</button>
		</div>
	{/if}	{#if currentPage === 'exercises-select'}
		<div class="page-header">
			<button class="back-button" on:click={goToMenu}>← Retour</button>
			<h2 class="page-title">Choisir le Type d'Entraînement</h2>
		</div>
		<div class="menu-container" style="margin-top: 2rem;">
			<button class="menu-button" on:click={() => startWorkout(4)}>Entraînement Complet (4 mouvements)</button>
			<button class="menu-button" on:click={() => startWorkout(2)}>Entraînement Rapide (2 mouvements)</button>
			<button class="menu-button" on:click={() => startWorkout(1)}>Mouvement Unique</button>
		</div>
	{/if}	{#if currentPage === 'exercises'}
		<div class="page-header">
			<button class="back-button" on:click={() => { currentPage = 'exercises-select'; selectedExercises = []; selectedExerciseIds.clear(); }}>← Retour</button>
			<h2 class="page-title">Générateur - {workoutType} mouvements</h2>
		</div>
		<div class="cards">
			{#if selectedExercises.length}
				{#each selectedExercises as ex}
					<div 
						class="card"
						class:selected={selectedExerciseIds.has(ex.id)}
						on:click={() => toggleExerciseSelection(ex.id)}
						on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); toggleExerciseSelection(ex.id); } }}
						role="button"
						tabindex="0"
					>
						<div class="card-body">
							<h4 style="margin: 0; color: #e0e0e0; font-size: 0.95rem; text-align: left; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;">{ex.name}</h4>
							<div class="card-category" style="margin-top: 0.35rem;">{capitalize(ex.category)}</div>
						</div>
						<img src={ex.image} alt={ex.name} class="card-image" />
					</div>
				{/each}
			{:else}
				<div class="no-selection">Aucun exercice sélectionné. Cliquez sur le bouton pour générer des exercices.</div>
			{/if}
		</div>

	{/if}	{#if currentPage === 'breathing'}
		<div class="page-header">
			<button class="back-button" on:click={goToMenu}>← Retour</button>
			<h2 class="page-title">Respiration et Conscience</h2>
		</div>
		{#if selectedBreathing}
			<div class="single-item">
				<img src={selectedBreathing.image} alt={selectedBreathing.name} />
				<div class="single-item-title"><strong>{selectedBreathing.name}</strong></div>
			</div>
		{:else}
			<div class="no-selection">Aucun exercice de respiration sélectionné. Cliquez sur le bouton pour en générer un.</div>
		{/if}
		<button class="action-button" on:click={pickRandomBreathing}>Générer un exercice de respiration</button>
	{/if}		{#if currentPage === 'smoking'}
			<div class="page-header">
				<button class="back-button" on:click={goToMenu}>← Retour</button>
				<h2 class="page-title">Méthodes de Fumage</h2>
			</div>
			{#if selectedSmoking}
				<div class="single-item">
					<img src={selectedSmoking.image} alt={selectedSmoking.name} />
					<div class="single-item-title"><strong>{selectedSmoking.name}</strong></div>
				</div>
			{:else}
				<div class="no-selection">Aucune méthode de fumage sélectionnée. Cliquez sur le bouton pour en générer une.</div>
			{/if}
			<button class="action-button" on:click={pickRandomSmoking}>Générer une méthode de fumage</button>
		{/if}

		{#if currentPage === 'favorites'}
			<div class="page-header">
				<button class="back-button" on:click={goToMenu}>← Retour</button>
				<h2 class="page-title">Mes Séances Favorites</h2>
			</div>
			
			{#if favorites.length > 0}
				<div class="cards">
					{#each favorites as fav}
						<div class="card" style="cursor: default;">
							<div style="padding: 1rem;">
								<h3 style="margin: 0 0 0.5rem 0; color: #b294cc;">{fav.name}</h3>
								<p style="margin: 0.25rem 0; color: #888888; font-size: 0.9rem;">
									{fav.type} mouvements • {new Date(fav.date).toLocaleDateString('fr-FR')}
								</p>
								<div style="margin-top: 0.75rem; display: flex; flex-wrap: wrap; gap: 0.5rem;">
									{#each fav.exercises as ex}
										<div class="card-category">{capitalize(ex.category)}</div>
									{/each}
								</div>
								<div style="margin-top: 1rem; display: flex; gap: 0.5rem;">
									<button class="action-button" style="flex: 1; padding: 0.5rem; font-size: 0.9rem;" on:click={() => { selectedExercises = fav.exercises; selectedExerciseIds = new Set(fav.exercises.map(e => e.id)); workoutType = fav.type; currentPage = 'exercises'; }}>Charger</button>
									<button class="action-button" style="flex: 1; padding: 0.5rem; font-size: 0.9rem; background-color: #9a6b9a;" on:click={() => { deleteFavorite(fav.id); favorites = favorites; }}>Supprimer</button>
								</div>


								</div>
						</div>
					{/each}
				</div>
			{:else}
				<div class="no-selection" style="text-align: center; margin-top: 2rem;">
					Aucune séance favorite pour l'instant. Créez-en une et sauvegardez-la!
				</div>
			{/if}
		{/if}
	{#if currentPage === 'exercises'}
		<div class="bottom-bar">
			<input class="bottom-input" type="text" bind:value={workoutName} placeholder="Nom de la séance..." />
			<button class="action-button" on:click={() => pickBalancedExercises(workoutType)} style="flex: 0 0 110px;">Générer</button>
			<button class="action-button" on:click={saveWorkout} style="flex: 0 0 110px;">💾 Sauvegarder</button>
		</div>
	{/if}
	</div>
</div>