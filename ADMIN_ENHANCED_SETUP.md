# Enhanced Admin Dashboard Setup Guide

## New Features Overview
This enhanced setup includes:
- **Edit Projects** - Update existing properties
- **Image Upload** - Upload images to Supabase Storage
- **Status Management** - Publish/Draft status for properties
- **Featured Properties** - Mark properties as featured
- **Search & Filter** - Find properties by location, price, bedrooms
- **Project Details Page** - View full property information
- **Contact Tracking** - Track inquiries from contact form
- **Admin Contacts Panel** - Manage inquiries

## Database Schema Updates

### Projects Table (Enhanced)
```sql
CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR NOT NULL,
  description TEXT NOT NULL,
  location VARCHAR NOT NULL,
  price VARCHAR NOT NULL,
  bedrooms VARCHAR,
  bathrooms VARCHAR,
  imageUrl VARCHAR,
  category VARCHAR DEFAULT 'Residential',
  status VARCHAR DEFAULT 'published', -- published, draft, sold
  featured BOOLEAN DEFAULT false,
  amenities TEXT, -- JSON array stored as text
  created_at TIMESTAMP DEFAULT now(),
  updated_at TIMESTAMP DEFAULT now(),
  created_by VARCHAR
);

-- Enable Row Level Security
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;

-- Allow public read
CREATE POLICY "Allow public read" ON projects
  FOR SELECT USING (true);

-- Allow inserts
CREATE POLICY "Allow inserts" ON projects
  FOR INSERT WITH CHECK (true);

-- Allow updates
CREATE POLICY "Allow updates" ON projects
  FOR UPDATE USING (true);

-- Allow deletes
CREATE POLICY "Allow deletes" ON projects
  FOR DELETE USING (true);
```

### Contacts Table (New)
```sql
CREATE TABLE contacts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR NOT NULL,
  email VARCHAR NOT NULL,
  phone VARCHAR,
  property_id UUID REFERENCES projects(id),
  message TEXT NOT NULL,
  interested_in VARCHAR,
  status VARCHAR DEFAULT 'new', -- new, contacted, closed
  created_at TIMESTAMP DEFAULT now(),
  replied_at TIMESTAMP
);

-- Enable Row Level Security
ALTER TABLE contacts ENABLE ROW LEVEL SECURITY;

-- Allow public read and insert
CREATE POLICY "Allow public insert" ON contacts
  FOR INSERT WITH CHECK (true);

-- Allow admin read/update
CREATE POLICY "Allow admin access" ON contacts
  FOR SELECT USING (true);

CREATE POLICY "Allow admin update" ON contacts
  FOR UPDATE USING (true);
```

### Storage Bucket (New)
Create a storage bucket for project images:
```
Bucket name: project-images
Public: true (make it public for image URLs)
```

## Setup Steps

### 1. Update Projects Table
1. Go to Supabase SQL Editor
2. Run the Projects table SQL above (it will modify existing table)
3. Add the new columns: status, featured, amenities, updated_at

**If you already have a projects table**, run these ALTER commands instead:
```sql
ALTER TABLE projects ADD COLUMN status VARCHAR DEFAULT 'published';
ALTER TABLE projects ADD COLUMN featured BOOLEAN DEFAULT false;
ALTER TABLE projects ADD COLUMN amenities TEXT;
ALTER TABLE projects ADD COLUMN updated_at TIMESTAMP DEFAULT now();
```

### 2. Create Contacts Table
1. In Supabase SQL Editor, run the Contacts table SQL above
2. Verify the table appears in the Tables list

### 3. Create Storage Bucket
1. Go to **Storage** in Supabase
2. Click **New Bucket**
3. Name: `project-images`
4. Uncheck "Make bucket private" (make it public)
5. Create bucket
6. Click on the bucket, go to **Policies**
7. Click **New policy** → **For full customization**
8. Name: `Allow public read`
9. Allowed operations: `SELECT`
10. Policy definition: `true`
11. Save policy

### 4. Environment Variables
No new environment variables needed. The SDK uses your existing Supabase credentials.

### 5. Restart Dev Server
```bash
npm run dev
```

## Admin Dashboard Routes

- `/admin/login` - Login page
- `/admin` - Admin home with feature cards
- `/admin/dashboard` - Project management (CRUD, upload, status)
- `/admin/contacts` - Contact inquiries management
- `/projects/{id}` - Public project details page
- `/projects` - Projects listing with search/filter

## Form Fields (Enhanced)

### Projects Form
- **Title** ✓ (required)
- **Description** ✓ (required)
- **Location** ✓ (required)
- **Price** ✓ (required)
- **Category** (Residential, Commercial, Land, Luxury, Investment)
- **Bedrooms** (optional number)
- **Bathrooms** (optional number)
- **Image** (file upload to Supabase Storage)
- **Status** (Published, Draft, Sold)
- **Featured** (toggle checkbox)
- **Amenities** (comma-separated list)

## Database Policies Explained

**Projects Table:**
- `Allow public read` - Everyone can view published properties
- `Allow inserts/updates` - Admin can add and modify properties
- `Allow deletes` - Admin can remove properties

**Contacts Table:**
- `Allow public insert` - Website visitors can submit inquiries
- `Allow admin access` - Admin can view and manage inquiries

## Testing the Features

### Add Project with Upload
1. Go to `/admin/dashboard`
2. Click "Add New Project"
3. Fill in form
4. Click image input → upload file
5. Set Status to "Published"
6. Toggle "Featured" if needed
7. Click "Add Project"

### Edit Project
1. Click edit icon on any project card
2. Update fields
3. Upload new image if needed
4. Click "Save Changes"

### View Published Projects
1. Go to `/projects`
2. Use search/filter to find properties
3. Click property card to view details

### Submit Contact Inquiry
1. Go to `/contact`
2. Fill inquiry form
3. Submit - saved to contacts table
4. Admin sees it in `/admin/contacts`

### Manage Contacts
1. Go to `/admin/contacts`
2. View all inquiries
3. Mark as "Contacted" or "Closed"
4. View inquiry details

## Next Steps
After setup is complete:
1. Add 3-5 test properties in admin dashboard
2. Upload images for each
3. Mark one as "Featured"
4. Test search/filter on projects page
5. Submit test contact form
6. View in admin panel

## Troubleshooting

**Images not uploading:**
- Check Storage bucket exists and is public
- Verify Supabase credentials in `.env.local`
- Check browser console for errors

**Can't see published projects:**
- Verify projects have `status: 'published'`
- Check projects page filtering logic

**Contacts not appearing:**
- Verify contacts table exists with correct schema
- Check browser console for form submission errors
- Verify contact form is properly wired to database

## Notes
- All times use UTC (created_at, updated_at, replied_at)
- Contact inquiries are stored indefinitely - archive old ones manually
- Image uploads limited to ~10MB per file (Supabase default)
- Status field helps with property lifecycle (published → sold)
