<script>
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import { adminAuth, logoutAdmin } from '$lib/stores/adminAuth';
	import { supabase } from '$lib/supabase';

	let isLoggedIn = false;
	let projects: any[] = [];
	let loading = false;
	let showForm = false;

	// Form data
	let formData = {
		title: '',
		description: '',
		location: '',
		price: '',
		bedrooms: '',
		bathrooms: '',
		imageUrl: '',
		category: 'Residential'
	};

	let error = '';
	let success = '';

	onMount(() => {
		adminAuth.subscribe((auth) => {
			isLoggedIn = auth.isLoggedIn;
			if (!isLoggedIn) {
				goto('/admin/login');
			}
		});

		loadProjects();
	});

	async function loadProjects() {
		try {
			const { data, error: err } = await supabase.from('projects').select('*');
			if (err) throw err;
			projects = data || [];
		} catch (err) {
			console.error('Error loading projects:', err);
		}
	}

	async function handleSubmit() {
		error = '';
		success = '';
		loading = true;

		if (
			!formData.title ||
			!formData.description ||
			!formData.location ||
			!formData.price ||
			!formData.imageUrl
		) {
			error = 'Please fill in all required fields';
			loading = false;
			return;
		}

		try {
			const { data, error: err } = await supabase.from('projects').insert([formData]);

			if (err) throw err;

			success = 'Project added successfully!';
			formData = {
				title: '',
				description: '',
				location: '',
				price: '',
				bedrooms: '',
				bathrooms: '',
				imageUrl: '',
				category: 'Residential'
			};
			showForm = false;
			await loadProjects();
		} catch (err) {
			error = 'Error adding project: ' + (err?.message || 'Unknown error');
		} finally {
			loading = false;
		}
	}

	async function deleteProject(id: string) {
		if (!confirm('Are you sure you want to delete this project?')) return;

		try {
			const { error: err } = await supabase.from('projects').delete().eq('id', id);
			if (err) throw err;
			success = 'Project deleted successfully!';
			await loadProjects();
		} catch (err) {
			error = 'Error deleting project: ' + (err?.message || 'Unknown error');
		}
	}

	function handleLogout() {
		logoutAdmin();
		goto('/admin/login');
	}
</script>

{#if isLoggedIn}
	<div class="admin-dashboard">
		<header class="admin-header">
			<div class="header-content">
				<h1>Admin Dashboard</h1>
				<button on:click={handleLogout} class="logout-btn">Logout</button>
			</div>
		</header>

		<main class="dashboard-main">
			<div class="dashboard-container">
				<div class="controls">
					<h2>Projects Management</h2>
					<button on:click={() => (showForm = !showForm)} class="btn-primary">
						{showForm ? 'Cancel' : '+ Add New Project'}
					</button>
				</div>

				{#if success}
					<div class="alert alert-success">{success}</div>
				{/if}

				{#if error}
					<div class="alert alert-error">{error}</div>
				{/if}

				{#if showForm}
					<div class="form-section">
						<h3>Add New Project</h3>
						<form on:submit|preventDefault={handleSubmit}>
							<div class="form-row">
								<div class="form-group">
									<label>Title *</label>
									<input type="text" bind:value={formData.title} placeholder="Project name" />
								</div>
								<div class="form-group">
									<label>Category</label>
									<select bind:value={formData.category}>
										<option value="Residential">Residential</option>
										<option value="Commercial">Commercial</option>
										<option value="Land">Land</option>
										<option value="Luxury">Luxury</option>
										<option value="Investment">Investment</option>
									</select>
								</div>
							</div>

							<div class="form-group">
								<label>Description *</label>
								<textarea
									bind:value={formData.description}
									placeholder="Detailed description of the property"
									rows="4"
								/>
							</div>

							<div class="form-row">
								<div class="form-group">
									<label>Location *</label>
									<input type="text" bind:value={formData.location} placeholder="e.g., Westlands, Nairobi" />
								</div>
								<div class="form-group">
									<label>Price (KES) *</label>
									<input type="number" bind:value={formData.price} placeholder="e.g., 5000000" />
								</div>
							</div>

							<div class="form-row">
								<div class="form-group">
									<label>Bedrooms</label>
									<input type="number" bind:value={formData.bedrooms} placeholder="e.g., 3" />
								</div>
								<div class="form-group">
									<label>Bathrooms</label>
									<input type="number" bind:value={formData.bathrooms} placeholder="e.g., 2" />
								</div>
							</div>

							<div class="form-group">
								<label>Image URL *</label>
								<input
									type="url"
									bind:value={formData.imageUrl}
									placeholder="https://example.com/image.jpg"
								/>
								{#if formData.imageUrl}
									<div class="image-preview">
										<img src={formData.imageUrl} alt="Preview" />
									</div>
								{/if}
							</div>

							<button type="submit" class="btn-submit" disabled={loading}>
								{loading ? 'Adding...' : 'Add Project'}
							</button>
						</form>
					</div>
				{/if}

				<div class="projects-list">
					<h3>All Projects ({projects.length})</h3>

					{#if projects.length === 0}
						<p class="no-projects">No projects added yet. Click "Add New Project" to get started.</p>
					{:else}
						<div class="projects-grid">
							{#each projects as project (project.id)}
								<div class="project-card">
									<img src={project.imageUrl} alt={project.title} class="project-image" />
									<div class="project-info">
										<h4>{project.title}</h4>
										<p class="category">{project.category}</p>
										<p class="location">{project.location}</p>
										<p class="price">KES {parseInt(project.price).toLocaleString()}</p>
										{#if project.bedrooms}
											<p class="specs">
												🛏️ {project.bedrooms} Beds | 🚿 {project.bathrooms} Baths
											</p>
										{/if}
										<button
											on:click={() => deleteProject(project.id)}
											class="btn-delete"
										>
											Delete
										</button>
									</div>
								</div>
							{/each}
						</div>
					{/if}
				</div>
			</div>
		</main>
	</div>
{/if}

<style>
	.admin-dashboard {
		background: #f5f5f5;
		min-height: 100vh;
	}

	.admin-header {
		background: #1f1810;
		color: white;
		padding: 20px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	}

	.header-content {
		max-width: 1300px;
		margin: 0 auto;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.header-content h1 {
		margin: 0;
		font-size: 24px;
	}

	.logout-btn {
		background: #d4af37;
		color: #1f1810;
		padding: 8px 16px;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-weight: 600;
		transition: background 0.3s;
	}

	.logout-btn:hover {
		background: #efbe5c;
	}

	.dashboard-main {
		padding: 30px 20px;
	}

	.dashboard-container {
		max-width: 1300px;
		margin: 0 auto;
	}

	.controls {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 30px;
	}

	.controls h2 {
		margin: 0;
		color: #1f1810;
	}

	.btn-primary {
		background: #0066cc;
		color: white;
		padding: 10px 20px;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-weight: 600;
		transition: background 0.3s;
	}

	.btn-primary:hover {
		background: #0052a3;
	}

	.alert {
		padding: 15px;
		border-radius: 4px;
		margin-bottom: 20px;
	}

	.alert-success {
		background: #efe;
		color: #060;
		border: 1px solid #080;
	}

	.alert-error {
		background: #fee;
		color: #c33;
		border: 1px solid #f00;
	}

	.form-section {
		background: white;
		padding: 30px;
		border-radius: 8px;
		margin-bottom: 30px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	}

	.form-section h3 {
		color: #1f1810;
		margin-top: 0;
	}

	.form-row {
		display: grid;
		grid-template-columns: 1fr 1fr;
		gap: 20px;
	}

	.form-group {
		margin-bottom: 20px;
	}

	.form-group label {
		display: block;
		margin-bottom: 8px;
		color: #1f1810;
		font-weight: 600;
		font-size: 14px;
	}

	.form-group input,
	.form-group select,
	.form-group textarea {
		width: 100%;
		padding: 10px;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 14px;
		font-family: inherit;
	}

	.form-group input:focus,
	.form-group select:focus,
	.form-group textarea:focus {
		outline: none;
		border-color: #0066cc;
		box-shadow: 0 0 0 3px rgba(0, 102, 204, 0.1);
	}

	.image-preview {
		margin-top: 10px;
		max-width: 200px;
	}

	.image-preview img {
		width: 100%;
		border-radius: 4px;
	}

	.btn-submit {
		background: #0066cc;
		color: white;
		padding: 12px 24px;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-weight: 600;
		transition: background 0.3s;
	}

	.btn-submit:hover:not(:disabled) {
		background: #0052a3;
	}

	.btn-submit:disabled {
		opacity: 0.6;
		cursor: not-allowed;
	}

	.projects-list {
		background: white;
		padding: 30px;
		border-radius: 8px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	}

	.projects-list h3 {
		color: #1f1810;
		margin-top: 0;
	}

	.no-projects {
		text-align: center;
		color: #999;
		padding: 40px 20px;
	}

	.projects-grid {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
		gap: 20px;
	}

	.project-card {
		border: 1px solid #ddd;
		border-radius: 8px;
		overflow: hidden;
		transition: box-shadow 0.3s;
	}

	.project-card:hover {
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
	}

	.project-image {
		width: 100%;
		height: 200px;
		object-fit: cover;
	}

	.project-info {
		padding: 15px;
	}

	.project-info h4 {
		margin: 0 0 5px 0;
		color: #1f1810;
	}

	.category {
		display: inline-block;
		background: #d4af37;
		color: #1f1810;
		padding: 4px 8px;
		border-radius: 4px;
		font-size: 12px;
		font-weight: 600;
		margin-bottom: 8px;
	}

	.location,
	.price,
	.specs {
		margin: 5px 0;
		font-size: 14px;
		color: #666;
	}

	.price {
		font-weight: 600;
		color: #0066cc;
	}

	.btn-delete {
		width: 100%;
		background: #dc3545;
		color: white;
		padding: 8px;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-weight: 600;
		margin-top: 10px;
		transition: background 0.3s;
	}

	.btn-delete:hover {
		background: #c82333;
	}

	@media (max-width: 768px) {
		.form-row {
			grid-template-columns: 1fr;
		}

		.projects-grid {
			grid-template-columns: 1fr;
		}

		.controls {
			flex-direction: column;
			gap: 15px;
			align-items: flex-start;
		}
	}
</style>
