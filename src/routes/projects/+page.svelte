<script lang="ts">
	import { onMount } from 'svelte';
	import { supabase } from '$lib/supabase';

	let projects: any[] = [];
	let filteredProjects: any[] = [];
	let loading = true;
	let error = '';

	// Filter state
	let searchTerm = '';
	let selectedCategory = '';
	let minPrice = '';
	let maxPrice = '';
	let minBeds = '';

	// Categories from projects
	let categories: string[] = [];
	let locations = new Set<string>();

	onMount(async () => {
		await loadProjects();
	});

	async function loadProjects() {
		try {
			const { data, error: err } = await supabase
				.from('projects')
				.select('*')
				.eq('status', 'published')
				.order('featured', { ascending: false })
				.order('created_at', { ascending: false });

			if (err) throw err;

			projects = data || [];

			// Extract unique categories and locations
			const cats = new Set<string>();
			projects.forEach(p => {
				if (p.category) cats.add(p.category);
				if (p.location) locations.add(p.location);
			});
			categories = Array.from(cats).sort();

			applyFilters();
		} catch (err) {
			console.error('Error loading projects:', err);
			error = 'Failed to load properties';
		} finally {
			loading = false;
		}
	}

	function applyFilters() {
		filteredProjects = projects.filter(project => {
			const matchesSearch = !searchTerm || 
				project.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
				project.location.toLowerCase().includes(searchTerm.toLowerCase()) ||
				project.description.toLowerCase().includes(searchTerm.toLowerCase());

			const matchesCategory = !selectedCategory || project.category === selectedCategory;

			const price = parseInt(project.price) || 0;
			const matchesMinPrice = !minPrice || price >= parseInt(minPrice);
			const matchesMaxPrice = !maxPrice || price <= parseInt(maxPrice);

			const beds = parseInt(project.bedrooms) || 0;
			const matchesBeds = !minBeds || beds >= parseInt(minBeds);

			return matchesSearch && matchesCategory && matchesMinPrice && matchesMaxPrice && matchesBeds;
		});
	}

	function resetFilters() {
		searchTerm = '';
		selectedCategory = '';
		minPrice = '';
		maxPrice = '';
		minBeds = '';
		applyFilters();
	}

	function formatPrice(price: string) {
		return parseInt(price).toLocaleString();
	}
</script>

<main class="projects-page">
	<!-- Page Header -->
	<div class="page-header">
		<div class="header-content">
			<h1>Explore Our Properties</h1>
			<p>Discover premium real estate opportunities across Nairobi</p>
		</div>
	</div>

	<div class="container">
		<div class="detail-grid">
			<!-- Sidebar Filters -->
			<aside class="filters-sidebar">
				<div class="filters">
					<h3>🔍 Search & Filter</h3>

					<!-- Search -->
					<div class="filter-group">
						<label for="search">Search by title or location</label>
						<input
							id="search"
							type="text"
							bind:value={searchTerm}
							on:change={applyFilters}
							placeholder="Enter keyword..."
						/>
					</div>

					<!-- Category -->
					{#if categories.length > 0}
						<div class="filter-group">
							<label for="category">Category</label>
							<select
								id="category"
								bind:value={selectedCategory}
								on:change={applyFilters}
							>
								<option value="">All Categories</option>
								{#each categories as cat}
									<option value={cat}>{cat}</option>
								{/each}
							</select>
						</div>
					{/if}

					<!-- Min Bedrooms -->
					<div class="filter-group">
						<label for="bedrooms">Minimum Bedrooms</label>
						<select
							id="bedrooms"
							bind:value={minBeds}
							on:change={applyFilters}
						>
							<option value="">All</option>
							<option value="1">1+</option>
							<option value="2">2+</option>
							<option value="3">3+</option>
							<option value="4">4+</option>
							<option value="5">5+</option>
						</select>
					</div>

					<!-- Min Price -->
					<div class="filter-group">
						<label for="minPrice">Min Price (KES)</label>
						<input
							id="minPrice"
							type="number"
							bind:value={minPrice}
							on:change={applyFilters}
							placeholder="0"
						/>
					</div>

					<!-- Max Price -->
					<div class="filter-group">
						<label for="maxPrice">Max Price (KES)</label>
						<input
							id="maxPrice"
							type="number"
							bind:value={maxPrice}
							on:change={applyFilters}
							placeholder="0"
						/>
					</div>

					<!-- Clear Filters -->
					{#if searchTerm || selectedCategory || minPrice || maxPrice || minBeds}
						<button class="btn-clear" on:click={resetFilters}>Clear Filters</button>
					{/if}
				</div>
			</aside>

			<!-- Main Content -->
			<section class="projects-content">
				{#if loading}
					<div class="loading-state">Loading properties...</div>
				{:else if error}
					<div class="error-message">{error}</div>
				{:else}
					<div class="results-header">
						<p class="results-info">
							Showing <span class="badge">{filteredProjects.length}</span> of
							<span class="badge">{projects.length}</span> properties
						</p>
					</div>

					{#if filteredProjects.length === 0}
						<div class="no-results">
							<p>📭 No properties match your search criteria</p>
							<button class="btn-reset" on:click={resetFilters}>Clear Filters</button>
						</div>
					{:else}
						<div class="projects-grid">
							{#each filteredProjects as project (project.id)}
								<div class="project-card">
									<a href="/projects/{project.id}" class="card-link">
										<div class="card-image">
											<img src={project.imageUrl} alt={project.title} />
											{#if project.featured}
												<div class="featured-badge">⭐ Featured</div>
											{/if}
										</div>

										<div class="card-content">
											<div class="card-header">
												<h3>{project.title}</h3>
												<span class="category">{project.category}</span>
											</div>

											<p class="location">📍 {project.location}</p>

											<div class="card-specs">
												{#if project.bedrooms}
													<span>🛏️ {project.bedrooms} Beds</span>
												{/if}
												{#if project.bathrooms}
													<span>🚿 {project.bathrooms} Baths</span>
												{/if}
											</div>

											{#if project.amenities}
												<p class="amenities">
													{project.amenities.split(',').slice(0, 2).join(', ')}...
												</p>
											{/if}

											<div class="card-footer">
												<span class="price">KES {formatPrice(project.price)}</span>
												<span class="view-btn">View Details →</span>
											</div>
										</div>
									</a>
								</div>
							{/each}
						</div>
					{/if}
				{/if}
			</section>
		</div>
	</div>
</main>

<style>
	:global(body) {
		background: #f9f9f9;
	}

	.projects-page {
		padding-top: 0;
	}

	.page-header {
		background: linear-gradient(135deg, #1f1810 0%, #3d3628 100%);
		color: white;
		padding: 60px 20px;
		text-align: center;
	}

	.header-content h1 {
		font-size: 48px;
		margin: 0;
		margin-bottom: 15px;
		font-weight: 700;
	}

	.header-content p {
		font-size: 18px;
		margin: 0;
		color: rgba(255, 255, 255, 0.8);
	}

	.container {
		max-width: 1400px;
		margin: 0 auto;
		padding: 40px 20px;
	}

	.detail-grid {
		display: grid;
		grid-template-columns: 280px 1fr;
		gap: 40px;
	}

	/* Sidebar Filters */
	.filters-sidebar {
		position: sticky;
		top: 20px;
	}

	.filters {
		background: white;
		padding: 25px;
		border-radius: 8px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
		border-top: 4px solid #d4af37;
	}

	.filters h3 {
		margin-top: 0;
		margin-bottom: 20px;
		color: #1f1810;
		font-size: 18px;
	}

	.filter-group {
		margin-bottom: 18px;
	}

	.filter-group label {
		display: block;
		color: #1f1810;
		font-size: 14px;
		font-weight: 600;
		margin-bottom: 6px;
	}

	.filter-group input,
	.filter-group select {
		width: 100%;
		padding: 8px 10px;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 14px;
		background: white;
	}

	.filter-group input:focus,
	.filter-group select:focus {
		outline: none;
		border-color: #0066cc;
		box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.1);
	}

	.btn-clear {
		width: 100%;
		background: #d4af37;
		color: #1f1810;
		padding: 10px;
		border: none;
		border-radius: 4px;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.2s;
		margin-top: 10px;
	}

	.btn-clear:hover {
		background: #c4991c;
		transform: translateY(-1px);
	}

	/* Main Content */
	.projects-content {
		padding: 0;
	}

	.results-header {
		margin-bottom: 30px;
		display: flex;
		align-items: center;
		justify-content: space-between;
	}

	.results-info {
		color: #666;
		font-size: 14px;
		margin: 0;
	}

	.badge {
		background: #d4af37;
		color: #1f1810;
		padding: 2px 8px;
		border-radius: 3px;
		font-weight: 600;
	}

	.loading-state {
		text-align: center;
		padding: 60px 20px;
		color: #999;
		font-size: 16px;
	}

	.error-message {
		background: #f8d7da;
		color: #721c24;
		padding: 15px 20px;
		border-radius: 4px;
		border: 1px solid #f5c6cb;
		margin-bottom: 20px;
	}

	.no-results {
		text-align: center;
		padding: 80px 20px;
		background: white;
		border-radius: 8px;
		color: #999;
	}

	.no-results p {
		font-size: 18px;
		margin-bottom: 20px;
	}

	.btn-reset {
		background: #0066cc;
		color: white;
		padding: 10px 20px;
		border: none;
		border-radius: 4px;
		font-weight: 600;
		cursor: pointer;
		transition: all 0.2s;
	}

	.btn-reset:hover {
		background: #0052a3;
		transform: translateY(-2px);
	}

	/* Projects Grid */
	.projects-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
		gap: 25px;
	}

	.project-card {
		background: white;
		border-radius: 8px;
		overflow: hidden;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
		transition: all 0.3s;
	}

	.project-card:hover {
		transform: translateY(-8px);
		box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
	}

	.card-link {
		text-decoration: none;
		color: inherit;
		display: block;
	}

	.card-image {
		position: relative;
		height: 220px;
		overflow: hidden;
		background: #ddd;
	}

	.card-image img {
		width: 100%;
		height: 100%;
		object-fit: cover;
		transition: transform 0.3s;
	}

	.project-card:hover .card-image img {
		transform: scale(1.08);
	}

	.featured-badge {
		position: absolute;
		top: 10px;
		left: 10px;
		background: rgba(212, 175, 55, 0.95);
		color: white;
		padding: 6px 12px;
		border-radius: 4px;
		font-size: 13px;
		font-weight: 600;
		z-index: 10;
	}

	.card-content {
		padding: 20px;
	}

	.card-header {
		display: flex;
		justify-content: space-between;
		align-items: flex-start;
		margin-bottom: 10px;
		gap: 10px;
	}

	.card-header h3 {
		margin: 0;
		color: #1f1810;
		font-size: 18px;
		flex: 1;
	}

	.category {
		background: #d4af37;
		color: #1f1810;
		padding: 4px 10px;
		border-radius: 3px;
		font-size: 12px;
		font-weight: 600;
		white-space: nowrap;
	}

	.location {
		color: #666;
		font-size: 13px;
		margin: 8px 0;
	}

	.card-specs {
		display: flex;
		gap: 15px;
		margin: 12px 0;
		font-size: 13px;
		color: #666;
	}

	.amenities {
		color: #999;
		font-size: 12px;
		margin: 10px 0;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}

	.card-footer {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-top: 15px;
		padding-top: 15px;
		border-top: 1px solid #eee;
	}

	.price {
		font-weight: 700;
		color: #0066cc;
		font-size: 16px;
	}

	.view-btn {
		color: #0066cc;
		font-weight: 600;
		font-size: 13px;
	}

	/* Mobile Responsive */
	@media (max-width: 768px) {
		.page-header {
			padding: 40px 20px;
		}

		.header-content h1 {
			font-size: 32px;
		}

		.header-content p {
			font-size: 16px;
		}

		.detail-grid {
			grid-template-columns: 1fr;
			gap: 20px;
		}

		.filters-sidebar {
			position: static;
			order: -1;
		}

		.filters {
			margin-bottom: 20px;
		}

		.projects-grid {
			grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
		}

		.card-header h3 {
			font-size: 16px;
		}
	}
</style>
