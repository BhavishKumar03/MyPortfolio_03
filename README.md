# Portfolio Website

A modern, full-stack portfolio website built with React, Vite, and Supabase. Features a responsive design, contact form, and admin dashboard for managing messages.

## Features

✨ **Modern UI**
- Clean, responsive design with Tailwind CSS
- Hero section with call-to-action
- About, Projects, and Contact sections
- Professional footer

🔐 **Authentication & Security**
- Supabase Auth integration
- Row-Level Security (RLS) policies
- Protected admin routes
- Admin verification via admins table

📊 **Admin Dashboard**
- View all contact messages
- Delete messages
- Real-time loading states
- Empty state UI

📝 **Contact Form**
- Email validation
- Success/error notifications
- Stores messages in Supabase
- Public accessibility

🚀 **Deployment Ready**
- Optimized for Vercel
- Environment variable configuration
- Production-grade code structure

## Tech Stack

### Frontend
- **React 18** - UI library
- **Vite** - Build tool (5x faster than Create React App)
- **React Router** - Client-side routing
- **Tailwind CSS** - Utility-first CSS framework

### Backend
- **Supabase** - Backend as a Service
  - PostgreSQL database
  - Authentication
  - Row-Level Security (RLS)
  - Real-time capabilities

### Deployment
- **Vercel** - Frontend hosting

## Prerequisites

- Node.js 16+ and npm or yarn
- Supabase project with tables already created
- Git (for version control)

## Project Setup

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Supabase

1. Create a Supabase project at [supabase.com](https://supabase.com)
2. Create the required tables:

#### users table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  password VARCHAR(255) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### admins table
```sql
CREATE TABLE admins (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

#### contact_messages table
```sql
CREATE TABLE contact_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

### 3. Set Up Row-Level Security (RLS)

Enable RLS on all tables and create policies:

```sql
-- Enable RLS
ALTER TABLE contact_messages ENABLE ROW LEVEL SECURITY;
ALTER TABLE admins ENABLE ROW LEVEL SECURITY;

-- Allow public INSERT on contact_messages
CREATE POLICY "Allow public insert on contact_messages" ON contact_messages
  FOR INSERT WITH CHECK (true);

-- Allow admin SELECT on contact_messages
CREATE POLICY "Allow admin select on contact_messages" ON contact_messages
  FOR SELECT
  USING (
    auth.uid() IN (
      SELECT id FROM admins 
      WHERE email = (SELECT email FROM auth.users WHERE id = auth.uid())
    )
  );

-- Allow admin DELETE on contact_messages
CREATE POLICY "Allow admin delete on contact_messages" ON contact_messages
  FOR DELETE
  USING (
    auth.uid() IN (
      SELECT id FROM admins 
      WHERE email = (SELECT email FROM auth.users WHERE id = auth.uid())
    )
  );
```

### 4. Configure Environment Variables

1. Copy `.env.example` to `.env.local`:
```bash
cp .env.example .env.local
```

2. Fill in your Supabase credentials in `.env.local`:
```
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_KEY=your_supabase_anon_key
```

Get these values from your Supabase project:
- Go to Settings → API → Project URL and Project API keys
- Copy the URL and anon/public key

### 5. Create Admin User

1. Sign up in your Supabase Auth dashboard
2. Insert the admin email into the admins table:

```sql
INSERT INTO admins (email) VALUES ('your-admin@example.com');
```

## Development

### Start Development Server

```bash
npm run dev
```

The app will be available at `http://localhost:5173`

### Build for Production

```bash
npm run build
```

This creates an optimized build in the `dist/` directory.

### Preview Production Build

```bash
npm run preview
```

## Project Structure

```
src/
├── components/          # Reusable React components
│   ├── Navigation.jsx
│   ├── Hero.jsx
│   ├── About.jsx
│   ├── Projects.jsx
│   ├── ContactForm.jsx
│   ├── Footer.jsx
│   ├── ProtectedRoute.jsx
│   └── index.js
├── pages/              # Page components
│   ├── HomePage.jsx
│   ├── LoginPage.jsx
│   ├── AdminPage.jsx
│   ├── NotFoundPage.jsx
│   └── index.js
├── contexts/           # React Context for state management
│   └── AuthContext.jsx
├── services/           # Business logic and API calls
│   └── contactService.js
├── App.jsx             # Main app component with routing
├── main.jsx            # Application entry point
├── index.css           # Global styles
└── supabaseClient.js   # Supabase configuration
```

## Routes

| Route | Public | Description |
|-------|--------|-------------|
| `/` | Yes | Home page with all sections |
| `/login` | Yes | Admin login page |
| `/admin` | No | Protected admin dashboard |
| `/*` | Yes | 404 Not Found page |

## Features Explained

### Authentication Flow
1. Admin enters credentials on `/login`
2. Supabase Auth validates credentials
3. App checks if user email exists in `admins` table
4. If admin, redirect to `/admin`; otherwise show error

### Contact Form Flow
1. Visitor fills out form on home page
2. Form validates inputs (name, email, message)
3. Message inserted into `contact_messages` table
4. Success message displayed
5. Admin can view/delete via dashboard

### Protected Routes
- Only logged-in admin users can access `/admin`
- Non-admins are redirected to `/login`
- Uses `ProtectedRoute` component wrapper

## Deployment to Vercel

### 1. Push to GitHub

```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/your-username/portfolio.git
git push -u origin main
```

### 2. Deploy to Vercel

1. Go to [vercel.com](https://vercel.com/)
2. Click "New Project"
3. Import your GitHub repository
4. Add environment variables:
   - `VITE_SUPABASE_URL`
   - `VITE_SUPABASE_KEY`
5. Click "Deploy"

### 3. Update GitHub Repository with Vercel Secret

Vercel will provide your deployment URL. Save it for reference.

## Security Notes

⚠️ **Important Security Considerations:**

1. **Never commit `.env.local`** - Keep private keys secret
2. **Use Supabase RLS** - Database policies ensure only admins can access/delete messages
3. **Validate inputs** - Server-side validation happens via RLS policies
4. **Use HTTPS** - Always use HTTPS in production
5. **Update dependencies** - Regularly update packages for security patches

## Troubleshooting

### Issue: "Missing Supabase credentials"
- Verify `.env.local` exists and has correct values
- Restart development server after updating `.env.local`

### Issue: Admin login fails
- Ensure user exists in Supabase Auth
- Check that user email is in `admins` table
- Verify RLS policies are correctly configured

### Issue: Contact form not submitting
- Check browser console for errors
- Verify RLS policy allows public INSERT
- Ensure table name is spelled correctly

### Issue: Admin dashboard shows no messages
- Refresh the page (F5)
- Verify users are admin (check `admins` table)
- Check RLS SELECT policy is working

## Performance Optimization

✅ Already implemented:
- Tailwind CSS for minimal CSS bundle
- Vite for fast build and HMR
- Code splitting with React Router
- Lazy loading images
- Optimized Supabase queries

## Future Enhancements

- [ ] Add project images/galleries
- [ ] Implement message pagination
- [ ] Add search/filter for admin dashboard
- [ ] Email notifications for new messages
- [ ] Dark mode toggle
- [ ] Blog section with Markdown support
- [ ] Analytics integration
- [ ] CDN for static assets

## Contributing

Contributions are welcome! Please follow these steps:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see LICENSE file for details.

## Support

For issues or questions:
1. Check the Troubleshooting section above
2. Review Supabase documentation: [supabase.com/docs](https://supabase.com/docs)
3. Check React Router docs: [reactrouter.com](https://reactrouter.com)
4. Open an issue on GitHub

## Author

Created with ❤️ for portfolio showcase

---

**Last Updated:** 2026
**Version:** 1.0.0
# MyPortfolio
