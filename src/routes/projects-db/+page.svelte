<script>
	import { supabase } from '$lib/supabase';
	import { onMount } from 'svelte';

	let projects = [];
	let loading = true;

	onMount(async () => {
		try {
			const { data, error } = await supabase.from('projects').select('*');
			if (error) throw error;
			projects = data || [];
		} catch (err) {
			console.error('Error loading projects:', err);
		} finally {
			loading = false;
		}
	});
</script>

<!-- Page Projects Start -->
<div class="page-project">
	<div class="container">
		<div class="row section-row" style="margin-bottom: 50px;">
			<div class="col-lg-12">
				<!-- Section Title Start -->
				<div class="section-title" style="text-align: center;">
					<h3 class="wow fadeInUp">featured listings</h3>
					<h2 class="text-anime-style-2" data-cursor="-opaque">
						Discover Our Premium <span>Property Portfolio</span>
					</h2>
					<p style="color: #666; margin-top: 15px; font-size: 1.05rem;">
						Explore our carefully curated selection of residential, commercial, and investment
						properties across Nairobi's most desirable neighborhoods.
					</p>
				</div>
				<!-- Section Title End -->
			</div>
		</div>

		{#if loading}
			<div class="row" style="text-align: center; padding: 60px 20px;">
				<p>Loading properties...</p>
			</div>
		{:else if projects.length === 0}
			<div class="row" style="text-align: center; padding: 60px 20px;">
				<p>No properties available at the moment. Please check back soon!</p>
			</div>
		{:else}
			<div class="row">
				{#each projects as project (project.id)}
					<div class="col-lg-4 col-md-6">
						<!-- Project Item Start -->
						<div class="project-item wow fadeInUp">
							<div class="project-img">
								<a href="#" data-cursor-text="View">
									<figure class="image-anime">
										<img src={project.imageUrl} alt={project.title} />
									</figure>
								</a>
							</div>
							<div class="project-title">
								<div
									style="display: flex; justify-content: space-between; align-items: center;"
								>
									<h3><a href="#">{project.title}</a></h3>
									<span
										style="background: #d4af37; color: white; padding: 4px 10px; border-radius: 4px; font-size: 0.8rem; font-weight: bold;"
									>
										{project.category || 'Residential'}
									</span>
								</div>
								<p style="color: #999; font-size: 0.9rem; margin-top: 5px;">
									{project.location}
								</p>
								<p style="color: #0066cc; font-weight: 600; margin-top: 5px;">
									KES {parseInt(project.price).toLocaleString()}
								</p>
								{#if project.bedrooms}
									<p style="color: #999; font-size: 0.85rem;">
										🛏️ {project.bedrooms} Beds | 🚿 {project.bathrooms} Baths
									</p>
								{/if}
								<p style="color: #666; font-size: 0.85rem; margin-top: 10px; line-height: 1.5;">
									{project.description.substring(0, 100)}...
								</p>
							</div>
							<div class="project-btn">
								<a href="#"><img src="/lib/assets/arrow-white.svg" alt="" /></a>
							</div>
						</div>
						<!-- Project Item End -->
					</div>
				{/each}
			</div>
		{/if}
	</div>
</div>
<!-- Page Projects End -->
