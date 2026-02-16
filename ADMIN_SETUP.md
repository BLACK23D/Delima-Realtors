# Admin Dashboard Setup Guide

## Overview
Your admin dashboard is now set up with:
- Password-protected login (`/admin/login`)
- Project management interface (`/admin/dashboard`)
- Supabase database integration

## Setup Steps

### 1. Create Supabase Account
1. Go to [supabase.com](https://supabase.com)
2. Sign up for free
3. Create a new project
4. Wait for the project to be created

### 2. Create Projects Table
1. In Supabase, go to **SQL Editor**
2. Click **New Query** and paste this SQL:

```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR NOT NULL,
  description TEXT NOT NULL,
  location VARCHAR NOT NULL,
  price VARCHAR NOT NULL,
  bedrooms VARCHAR,
  bathrooms VARCHAR,
  imageUrl VARCHAR NOT NULL,
  category VARCHAR DEFAULT 'Residential',
  created_at TIMESTAMP DEFAULT now(),
  created_by VARCHAR
);

-- Enable Row Level Security (RLP)
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Create policy to allow anyone to read
CREATE POLICY "Allow public read" ON projects
  FOR SELECT USING (true);

-- Create policy to allow inserts (we'll add auth later)
CREATE POLICY "Allow inserts" ON projects
  FOR INSERT WITH CHECK (true);

-- Create policy to allow deletes
CREATE POLICY "Allow deletes" ON projects
  FOR DELETE USING (true);
```

3. Click **Run** to execute the SQL

### 3. Get Your Supabase Credentials
1. In Supabase, click **Settings** (gear icon)
2. Go to **API**
3. Copy:
   - **Project URL** → `VITE_SUPABASE_URL`
   - **Project API Key (anon)** → `VITE_SUPABASE_ANON_KEY`

### 4. Update Environment Variables
Edit `.env.local` in your project root:

```
VITE_SUPABASE_URL=your_project_url_here
VITE_SUPABASE_ANON_KEY=your_anon_key_here
VITE_ADMIN_PASSWORD=delima123
```

Replace with your actual Supabase credentials.

### 5. Access the Admin Dashboard
1. Start your dev server: `npm run dev`
2. Go to `http://localhost:5173/admin/login`
3. Enter any email and password: **delima123**
4. Click Login

## Admin Dashboard Features

### Add New Project
1. Click **+ Add New Project** button
2. Fill in the required fields:
   - **Title** - Property name
   - **Description** - Property details
   - **Location** - Area in Nairobi
   - **Price** - Property cost in KES
   - **Image URL** - Link to property image
3. Optional fields:
   - **Bedrooms** - Number of bedrooms
   - **Bathrooms** - Number of bathrooms
   - **Category** - Residential, Commercial, Land, Luxury, Investment
4. Click **Add Project** to save

### View Projects
- All projects appear in a grid below the form
- Shows project details (image, price, location, specs)
- Click **Delete** to remove a project

## Updating the Projects Page
To display projects from the database on your main website:

Edit `src/routes/projects/+page.svelte` and replace the static project items with:

```svelte
<script>
	import { supabase } from '$lib/supabase';
	import { onMount } from 'svelte';

	let projects = [];

	onMount(async () => {
		const { data, error } = await supabase.from('projects').select('*');
		if (error) console.error('Error:', error);
		projects = data || [];
	});
</script>

{#each projects as project (project.id)}
	<div class="col-lg-4 col-md-6">
		<div class="project-item wow fadeInUp">
			<div class="project-img">
				<a href="#" data-cursor-text="View">
					<figure class="image-anime">
						<img src={project.imageUrl} alt={project.title} />
					</figure>
				</a>
			</div>
			<div class="project-title">
				<div style="display: flex; justify-content: space-between; align-items: center;">
					<h3><a href="#">{project.title}</a></h3>
					<span style="background: #d4af37; color: white; padding: 4px 10px; border-radius: 4px; font-size: 0.8rem; font-weight: bold;">
						{project.category}
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
			</div>
		</div>
	</div>
{/each}
```

## Changing Admin Password
To change the admin password, edit `.env.local`:
```
VITE_ADMIN_PASSWORD=your_new_password
```

Then restart your dev server.

## Troubleshooting

### "Missing Supabase environment variables"
- Make sure `.env.local` is in the project root
- Check that VITE_SUPABASE_URL and VITE_SUPABASE_ANON_KEY are filled
- Restart the dev server after adding env vars

### Can't login
- Check that the password matches `VITE_ADMIN_PASSWORD` in `.env.local`
- Any email works, only password matters

### Projects not loading
- Check that the `projects` table exists in Supabase
- Make sure RLS policies are set to allow public read/insert/delete
- Check browser console for error messages

### Images not displaying
- Make sure the URL is publicly accessible
- Try uploading images to Supabase Storage for better management

## Security Notes
This is a basic dashboard suitable for development/small teams. For production:
- Use Supabase user authentication instead of simple password
- Add email verification
- Implement proper role-based access control
- Use Supabase Storage for image uploads instead of external URLs
- Add activity logging

## Next Steps
1. Set up Supabase with the SQL from Step 2
2. Add your Supabase credentials to `.env.local`
3. Restart your dev server
4. Go to `/admin/login` and start adding projects!
