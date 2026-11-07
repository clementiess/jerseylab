# JerseyLab

JerseyLab is a React storefront built with Vite. Supabase provides the online
product catalog, customer and admin authentication, and image storage.

## Run locally

1. Install dependencies with `npm install`.
2. Copy `.env.example` to `.env.local`.
3. Configure the Supabase project URL and publishable key in `.env.local`.
4. Run `npm run dev`.

The Supabase publishable key is intended for browser use. Never put a
`service_role` or secret key in a `VITE_` variable.

## Set up Supabase

1. Create a Supabase project.
2. In the Supabase SQL Editor, run these migrations in timestamp order:
   [`supabase/migrations/20261103130000_create_store.sql`](./supabase/migrations/20261103130000_create_store.sql)
   and
   [`supabase/migrations/20261103170000_add_design_category_price_year.sql`](./supabase/migrations/20261103170000_add_design_category_price_year.sql)
   Then run
   [`supabase/migrations/20261103190000_add_premium_link.sql`](./supabase/migrations/20261103190000_add_premium_link.sql)
   and
   [`supabase/migrations/20261103195000_secure_download_links.sql`](./supabase/migrations/20261103195000_secure_download_links.sql)
   to enable protected design links. These migrations create or update the
   products table, public preview image bucket, row-level security policies,
   realtime updates, and sample designs. Existing product categories are
   converted to Jersey or Mockup. The archives migration is no longer needed
   for new setups.
3. In Supabase **Project Settings → API**, copy the Project URL and the
   publishable key. Add them to `.env.local`:

   ```dotenv
   VITE_SUPABASE_URL=https://YOUR_PROJECT_ID.supabase.co
   VITE_SUPABASE_PUBLISHABLE_KEY=YOUR_SUPABASE_PUBLISHABLE_KEY
   ```

   The older anon key can also be used as `VITE_SUPABASE_ANON_KEY`.
4. Restart the Vite server after changing `.env.local`.
5. Enable email/password sign-up in Supabase **Authentication → Providers**.
   Customers can then create accounts in the storefront. Configure email confirmation and
   allowed redirect URLs under **Authentication → URL Configuration** as
   appropriate for your store.
6. In Supabase **Authentication → Users**, create your admin user and set a
   strong password.
7. Copy that user's UUID and run this in the SQL Editor:

   ```sql
   insert into public.store_admins (user_id)
   values ('PASTE_ADMIN_USER_UUID_HERE');
   ```

8. Open the account dialog in the storefront and sign in with that account.
   Only users listed in `store_admins` can upload designs and their links.

The SQL migrations enable row-level security. Visitors can read products and
images; only authorized admins can publish products and upload images. Images
and product details are saved in Supabase and are available after refresh and
to other visitors. Designs have Jersey or Mockup categories, Free or Premium
price tiers, and an upload year. Admins provide an HTTPS link for every design. Download links are stored
separately from public product data and can only be read by authenticated users.
The link opens in a new tab from the design detail page after the signed-in user
requests it.

For an existing Supabase project, run the new
[`20261103195000_secure_download_links.sql`](./supabase/migrations/20261103195000_secure_download_links.sql)
migration before deploying this version. It moves existing links into the
protected table and removes the publicly readable link column.

## Deploy

Deploy the Vite app to a static host such as Vercel or Netlify. Add
`VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` as build environment
variables in the hosting dashboard, then redeploy. Add the deployed site URL
to the allowed redirect URLs in Supabase **Authentication → URL Configuration**.

The publishable key is safe to expose in the browser because access is
controlled by the database and storage policies. Do not expose a Supabase
secret or `service_role` key.

## Scripts

- `npm run dev` — start the development server
- `npm run build` — create a production build
- `npm run preview` — preview the production build
- `npm run lint` — run ESLint
